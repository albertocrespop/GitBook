---
description: '🧠Dificultad: Fácil | 🔓02/09/2025'
---

# ConsoleLog

## 🕵️ Reconocimiento

Escaneamos los puertos de la máquina con `nmap`:

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Vemos que hay dos servicios HTTP abiertos: uno en el puerto 80 y otro en el 3000. Además, la máquina tiene abierto un servicio SSH en el puerto 5000.

Si nos vamos al navegador, nos encontramos con una página con un botón. Cada vez que le damos al botón, se llama a la función `console.log` que imprime lo siguiente en la consola:

<figure><img src="../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

Realizamos un escaneo de directorios sobre este servicio:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://$IPTARGET/FUZZ -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

En el directorio `/backend` se encuentran algunos ficheros utilizados para el backend del servicio web. Entre ellos, nos fijamos en el fichero llamado `server.js`:

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Este script en `js` gestiona peticiones POST de un servicio alojado en el puerto 3000 (visto previamente). Por cada petición POST que se hace al directorio `/recurso/` de dicho servidor, se comprueba si la petición incluye el token `tokentraviesito`. En dicho caso, se devuelve la cadena `lapassworddebackupmaschingonadetodas`. De lo contrario, se devuelve un 401.

Hacemos la prueba:

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

La cadena de antes puede ser una contraseña, por lo que podemos hacer password spraying al SSH para ver qué usuario tiene esta contraseña:

```bash
hydra -L /usr/share/wordlists/rockyou.txt -p lapassworddebackupmaschingonadetodas ssh://$IPTARGET:5000
```

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

Encontramos unas credenciales:

```
lovely:lapassworddebackupmaschingonadetodas
```

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Primero de todo, comprobamos los permisos que tenemos con `sudo`:

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

Contamos con permisos para el binario `nano` como `root` sin necesidad de contraseña.

Una forma de escalar privilegios en esta máquina es editar el fichero `/etc/shadow` con nano quitándole la contraseña al usuario `root`. De esta manera, haciendo un `su` tendremos acceso a `root` directamente cuando queramos.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Otro camino es ejecutar código dentro de nano para obtener una shell como `root` (visto en [gtfobins](https://gtfobins.github.io/)):

```bash
sudo nano
# Presionar CTRL+R y CTRL+X
reset; sh 1>&0 2>&0
```

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
