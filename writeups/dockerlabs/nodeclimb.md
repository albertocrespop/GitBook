---
description: '🧠Dificultad: Fácil | 🔓26/01/2026'
---

# NodeClimb

## 🕵️ Reconocimiento

Realizamos un escaneo de puertos con nmap:

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

Vemos que se encuentra abierto el puerto FTP y SSH. En el servicio FTP nos permiten loguearnos anónimamente. Vamos a ver si encontramos algún archivo.

<figure><img src="../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

Hay un zip llamado `secretitopicaron.zip`. Nos lo llevamos a nuestra máquina para manipularlo.

## 🚪 Ganando acceso

El archivo zip tiene una contraseña, por lo que no podemos extraer su contenido. Usando `fcrackzip` podemos hacerle fuerza bruta. En mi caso, haré un ataque de diccionario con el `rockyou.txt`:

```bash
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt secretitopicaron.zip
```

<figure><img src="../../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

Encontramos la contraseña: `password1`. Ahora sí podemos ver el contenido del zip:

<figure><img src="../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

Accedemos vía SSH con las credenciales encontradas:

<figure><img src="../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Verificamos los permisos que tenemos con sudo en el sistema:

<figure><img src="../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar `/usr/bin/node /home/mario/script.js` como cualquier usuario (nos interesa el root) sin necesidad de contraseña. El fichero script.js está vacío, y como tenemos permisos de escritura podemos modificarlo. En [gtfobins](https://gtfobins.github.io/) encontramos una manera de obtener una shell en node. Si ejecutamos el script con sudo, obtendríamos una shell como root:

<figure><img src="../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

