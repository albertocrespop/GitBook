---
description: '🧠Dificultad: Fácil | 🔓28/08/2025'
---

# 🟩 Amor

## 🕵️ Reconocimiento

El escaneo de puertos con nmap nos muestra lo siguiente:

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

La máquina solo cuenta con dos servicios: SSH y HTTP.  Si revisamos la página web nos encontramos con lo siguiente:

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Probando fuerza bruta en el SSH con el usuario `juan` no obtuve nada, pero sí para el usuario `carlota`:

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalando privilegios

Mirando el `/home` del usuario actual, me encuentro con una serie de carpetas que me llevan hasta una imagen `jpg`.

<div align="left"><figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure></div>

Además, en las variables de entorno del sistema existe una variable llamada `SECRET` que nos sugiere que hay algo escondido en la imagen.

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Pasamos la imagen a nuestra máquina con un servidor de `python`, y ejecutamos `steghide` para ver si hay algo escondido en la imagen.

<div align="left"><figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure></div>

Esto pueden ser las credenciales de algún usuario. Si vemos el `/etc/passwd`, hay un usuario `oscar` que tiene un shell de inicio de sesión asignado, por lo que podemos probar con este usuario.

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

Viendo los permisos de sudo, vemos que tenemos permiso para ejecutar el binario `/usr/bin/ruby`  como `root` sin necesidad de contraseña. En [gtfobins](https://gtfobins.github.io/gtfobins/ruby/) encontramos un comando para explotar este fallo de seguridad:

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>
