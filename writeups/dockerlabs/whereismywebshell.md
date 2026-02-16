---
description: '🧠Dificultad: Fácil | 🔓16/02/2026'
---

# 🟩 WhereIsMyWebShell

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Solo se encuentra el puerto del servicio web activo. Procedemos a realizar un escaneo de directorios:

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Si entramos a la página `warning.html`, encontramos el siguiente mensaje:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Tendremos que realizar un escaneo de parámetros en `shell.php` para ver qué parametro es el utilizado para la webshell.

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u "http://$IPTARGET/shell.php?FUZZ=whoami" -mc 200
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Vemos que el parámetro es `parameter`. Con esto podemos ejecutar comandos en la máquina víctima.

## 🚪 Ganando acceso

Con burpsuite capturamos la petición e inyectamos el comando que nos ejecuta la reverse shell. En este caso he usado PHP con la herramienta [reverse shell generator](https://www.revshells.com/).

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Nos ponemos a la escucha con `netcat` y mandamos la petición.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Buscamos los ficheros .txt en el sistema. Encontramos un fichero en `/tmp/.secret.txt`:

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Este fichero contiene la contraseña de `root`, por lo que nos da acceso directo.

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
