---
description: '🧠Dificultad: Fácil | 🔓15/03/2026'
---

# 🟩 Psycho

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Tenemos abiertos únicamente los servicios HTTP y SSH. Realizamos un escaneo de directorios con `ffuf` al servicio web:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

No hay nada que llame la atención, por lo que nos metemos desde el navegador para ver el contenido de la página.

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Al final de la página aparece un mensaje de error. Es posible que esté intentando procesar algún parámetro, ya que estamos ante un fichero `.php`. Con un escaneo usando ffuf encontramos el parámetro oculto. Usamos de referencia el tamaño del contenido devuelto por el servicio.

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/burp-parameter-names.txt -u "http://$IPTARGET/index.php?FUZZ=test" -fs 2596
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

El parámetro oculto era secret. Comprobamos si este parámetro es usado para mostrar el contenido de ficheros. Probamos con un fichero que se encuentra dentro del directorio `assets`, que conocemos porque hay directory listing habilitado.

```
http://172.17.0.2/index.php?secret=assets/background.jpg
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

Efectivamente podemos leer ficheros a través de este parámetro.

## 🚪 Ganando acceso

Leemos el fichero `/etc/passwd` para obtener los usuarios de la máquina.

```
http://172.17.0.2/index.php?secret=../../../../../../../../etc/passwd
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

Probando fuerza bruta con los usuarios `vaxei` y `luisillo`, no encontramos nada. Si intentamos coger la clave RSA para conectarnos por SSH directamente a la máquina, vemos que podemos leer la clave del directorio personal de `vaxei`.

```
http://172.17.0.2/index.php?secret=../../../../../../../../home/vaxei/.ssh/id_rsa
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

Copiamos el contenido en nuestra máquina y nos conectamos por SSH.

<figure><img src="../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Comprobamos los permisos con `sudo` del usuario `vaxei`.

<figure><img src="../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar el binario `/usr/bin/perl` como el usuario `luisillo` sin necesidad de introducir contraseña. En [gtfobins](https://gtfobins.org/gtfobins/) encontramos una manera de aprovechar esta configuración:

```bash
sudo -u luisillo perl -e 'exec "/bin/sh"'
```

<figure><img src="../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

Si vemos ahora los permisos con sudo de este usuario, encontramos lo siguiente:

<figure><img src="../../.gitbook/assets/image (10) (1) (1).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar `/usr/bin/python3 /opt/paw.py` como el usuario `root`. Solo tenemos permisos de lectura sobre este fichero, así que no podemos modificarlo para obtener una shell.

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

Secuestramos la función `run` de la librería `subprocess`.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Y ejecutamos el script con sudo.

<figure><img src="../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>
