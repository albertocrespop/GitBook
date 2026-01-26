---
description: '🧠Dificultad: Fácil | 🔓24/01/2026'
---

# Los 40 Ladrones

## 🕵️ Reconocimiento

Realizamos un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET -oX salida.xml
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Vemos que solo está el puerto 80 abierto, correspondiente al de un servicio web con Apache. Ejecutando un escaneo de directorios encontramos los siguientes:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Si accedemos al `index.html`, encontramos esta información:

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Los número 7000, 8000 y 9000 pueden referirse a puertos para un port knocking (por eso se refiere a un tal `toctoc`, que podría ser el usuario para el SSH).

## 🚪 Ganando acceso

Con la herramienta `knock`, introducimos los puertos que forman parte de la "contraseña" para abrir el puerto cerrado, que hasta ahora esperamos que sea el 22 pero podría ser otro.

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Efectivamente nos ha abierto el puerto del servicio SSH. Ahora podemos proceder a realizar un ataque de fuerza bruta con el usuario `toctoc`:

<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Verificamos los permisos que tenemos con sudo:

<figure><img src="../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar un bash localizado en `/opt/bash/` como root sin necesidad de contraseña.

<figure><img src="../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>
