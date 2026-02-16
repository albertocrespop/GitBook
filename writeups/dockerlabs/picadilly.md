---
description: '🧠Dificultad: Fácil | 🔓30/01/2026'
---

# 🟩 Picadilly

## 🕵️ Reconocimiento

Comenzamos realizando un escaneo de puertos con nmap:

{% code overflow="wrap" %}
```bash
nmap -sC -sV -Pn $IPTARGET
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

Vemos puertos web abiertos (HTTP y HTTPS). Vamos a ver qué directorios encontramos en estos servicios:

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Para la página que usa SSL/TLS, vamos a utilizar el parámetro -k para ignorar los errores de los certificados.

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "https://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js -k
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (7) (1) (1).png" alt=""><figcaption></figcaption></figure>

La página web que no está securizada contiene un directory listing. En este se encuentra el archivo `backup.txt`, que contiene lo siguiente:

<figure><img src="../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

Si probamos a poner esta palabra en la herramienta Cyberchef y utilizamos el cifrado césar, encontramos que si desplazamos 3 veces hacia atrás las letras, vemos una posible contraseña coherente.&#x20;

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Veamos qué hay en el servicio HTTPS. Hallamos una sección donde podemos subir archivos a la web.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Probamos a subir un archivo llamado `prueba`, pero este no se muestra en la página. Si cambiamos la extensión desde burpsuite a `.txt`, entonces si podemos visualizarlo.

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Directamente podemos probar si podemos subir un archivo `.php`:

<figure><img src="../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ha funcionado correctamente, aunque visualizar el archivo php no nos ayuda de mucho de momento. Nos interesa ejecutarlo. Por ello, accedemos al directorio `/uploads` descubierto anteriormente. Aquí vemos que todos los archivos han sido subidos correctamente:

<figure><img src="../../.gitbook/assets/image (6) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Podemos subir una reverse shell en php a la web y ejecutarla. Para ello utilizaremos la revershe shell de pentestmonkey:

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

Nos ponemos a la escucha en nuestra máquina y accedemos al archivo php que hemos subido desde el directorio `/uploads` para que se ejecute:

<figure><img src="../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Una vez dentro, podemos pivotar al usuario mateo con la contraseña que hemos encontrado antes, aunque hay un pequeño error: la contraseña real es `easycrazy`, y no `easycrxazy` como habíamos resuelto.

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

Una vez accedido como mateo, vemos los permisos que tenemos con sudo:

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar una shell usando php como root. En este caso, utilizo el comando que se encuentra en [gtfobins](https://gtfobins.github.io/).

<figure><img src="../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>
