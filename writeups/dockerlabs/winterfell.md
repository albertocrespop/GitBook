---
description: '🧠Dificultad: Fácil | 🔓25/02/2026'
---

# 🟩 Winterfell

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/image (154).png" alt=""><figcaption></figcaption></figure>

Vemos abiertos dos sambas, un servicio web y el puerto SSH. Realizamos un escaneo de directorios sobre el servicio web con `ffuf`:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/image (156).png" alt=""><figcaption></figcaption></figure>

Si nos vamos al directorio `dragon`, encontramos un directory listing. En él vemos un fichero que contiene una posible lista de contraseñas.

<figure><img src="../../.gitbook/assets/image (155).png" alt=""><figcaption></figcaption></figure>

Con `crackmapexec` podemos enumerar los usuarios del samba.

```bash
crackmapexec smb 172.17.0.2 --users
```

<figure><img src="../../.gitbook/assets/image (153).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Una vez tenemos el usuario del samba (`jon`), podemos hacer una ataque de diccionario para conseguir las credenciales.

```bash
crackmapexec smb 172.17.0.2 -u users.txt -p passwords.txt
```

<figure><img src="../../.gitbook/assets/image (157).png" alt=""><figcaption></figcaption></figure>

La combinación correcta es `jon:seacercaelinvierno`. Usamos esta información para ganar acceso al samba. Dentro podemos ver un fichero llamado `proteccion_del_reino`. Nos lo llevamos a nuestra máquina y lo leemos.

<figure><img src="../../.gitbook/assets/image (158).png" alt=""><figcaption></figcaption></figure>

Como se ve a continuación, el fichero menciona una contraseña cifrada. Como termina con un símbolo `=`, se puede deducir que está cifrada en base64, así que podemos descifrarla fácilmente.

<figure><img src="../../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

Con esta contraseña podemos acceder por SSH con el usuario `jon` y obtener acceso directo a la máquina.

<figure><img src="../../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Una vez dentro del sistema como `jon`, vemos un mensaje en su carpeta personal.

<figure><img src="../../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>

Si revisamos los permisos que tenemos con `sudo`, vemos el script del que se habla en el mensaje. Podemos ejecutarlo como el usuario `aria` sin necesidad de introducir contraseña. El programa es el siguiente.

<figure><img src="../../.gitbook/assets/image (163).png" alt=""><figcaption></figcaption></figure>

Sobre este programa no tenemos permisos de escritura (no podemos modificarlo), así que tenemos que recurrir a otro método: library hijacking. Podemos secuestrar una librería usada en el programa para que una función determinada (en mi caso, la función `getuser()`) haga lo que queramos.

Vamos a hacer que la función ejecute `/bin/bash` para poder introducir comandos.

<figure><img src="../../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

Una vez hecho esto, conseguiríamos una shell como `aria`.

<figure><img src="../../.gitbook/assets/image (162).png" alt=""><figcaption></figcaption></figure>

De nuevo, vemos los permisos que tenemos con `sudo`, pero esta vez para el usuario `aria`.

<figure><img src="../../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>

Podemos listar cualquier directorio y leer cualquier fichero como el usuario `daenerys`. Aprovechamos esto para leer directamente lo que haya en el directorio personal de este usuario.

<figure><img src="../../.gitbook/assets/image (166).png" alt=""><figcaption></figcaption></figure>

La contraseña de `daeneris` es `drakaris` (xd).

<figure><img src="../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

Vamos a revisar esta vez los permisos con `sudo` del usuario `daenerys`.

<figure><img src="../../.gitbook/assets/image (168).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar `/usr/bin/bash /home/daenerys/.secret/.shell.sh` directamente como `root` sin contraseña. Además, podemos modificar el script, aunque el trabajo nos lo han dado casi hecho porque es una reverse shell. Simplemente cambiamos la IP y el puerto al de nuestra máquina, ejecutamos el script con `sudo`, y obtenemos una shell como `root`.

<figure><img src="../../.gitbook/assets/image (151).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (152).png" alt=""><figcaption></figcaption></figure>
