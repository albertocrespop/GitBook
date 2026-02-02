---
description: '🧠Dificultad: Fácil | 🔓02/09/2025'
---

# ChocolateLovers

## 🕵️ Reconocimiento

<figure><img src="../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

La máquina solo tiene el puerto 80 abierto con un servicio web. Si accedemos a él a través de un navegador, vemos la página por defecto de Apache. En los comentarios de la página se hace referencia a `nibbleblog`. Si accedemos al directorio `/nibbleblog`, vemos lo siguiente:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Estamos en la página principal de Nibbleblog, una aplicación que permite crear y administrar blogs. En el post inicial nos dan un enlace a `/nibbleblog/admin.php`. Si accedemos a este sitio encontraremos un inicio de sesión al panel de administrador. Probando con las credenciales `admin:admin`, ganamos acceso al panel:

<div align="center" data-full-width="false"><figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

<figure><img src="../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

Si buscamos exploits para nibbleblog con `searchsploit`, vemos que hay uno para subir ficheros arbitrarios a través de `metasploit`:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Aunque el exploit es sencillo de llevar a cabo manualmente, uso metasploit para ganar una shell.

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

El exploit falló al principio, y es que para que funcione correctamente, debemos tener instalado el plugin `My image`. Para ello, nos vamos al panel de plugins y lo instalamos:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ahora si, ganamos una sesión con `meterpreter`:

<div align="left"><figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

<div align="left"><figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

## 💥 Escalada de privilegios

<mark style="color:$warning;">Mejorar la shell desde meterpreter es bastante tedioso, por ello, recomiendo que ganéis acceso manualmente y no uséis metasploit.</mark>

Comprobamos los permisos que tenemos con `sudo`:

<figure><img src="../../.gitbook/assets/image (11) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Teniendo permisos para ejecutar php con `chocolate`, podemos ejecutar a través de la función `system()` el comando que queramos, por ejemplo, `/bin/bash`. De esta manera obtendremos una shell como el usuario `chocolate`:

<figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Una vez obtenida la shell como usuario `chocolate`, listamos los procesos del sistema (`sudo -l` nos pide contraseña):

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Existe un proceso de `root` que ejecuta el fichero `script.php` cada 0.5 segundos con `php`.

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

El script no hace prácticamente nada. Vemos que tenemos permisos suficientes sobre el fichero, por lo que podemos cambiar el contenido de este para ponerle el bit suid a `/bin/bash`. Después, bastará con ejecutar `bash -p` para obtener una shell como `root`.

```bash
echo "<?php system('chmod +s /bin/bash'); ?>" > script.php
```

<div align="left"><figure><img src="../../.gitbook/assets/image (14) (1) (1).png" alt=""><figcaption></figcaption></figure></div>
