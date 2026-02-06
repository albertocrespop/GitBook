---
description: '🧠Dificultad: Fácil | 🔓13/10/2025'
---

# 🟩 Library

## 🕵️ Reconocimiento

Realizamos un escaneo de puertos con `nmap`:

<figure><img src="../../.gitbook/assets/Pasted image 20251009160514.png" alt=""><figcaption></figcaption></figure>

Vemos que están abiertos el puerto 22 (`ssh`) y 80 (`http`). Si accedemos al `/index.html`, vemos la página default de `apache`. Hacemos un fuzzing de directorios con `ffuf` para encontrar otras rutas interesantes:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://$IPTARGET/FUZZ -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/Pasted image 20251013152033.png" alt=""><figcaption></figcaption></figure>

Vemos la existencia de un `index.php`. En el navegador nos encontramos lo siguiente al acceder a dicha página:

<figure><img src="../../.gitbook/assets/Pasted image 20251013152220.png" alt=""><figcaption></figcaption></figure>

Nos encontramos con una cadena extraña. No es `base64`, ni ninguna otra codificación que se me haya ocurrido, por lo que puede ser una contraseña o un usuario para el `ssh`.&#x20;

## 🚪 Ganando acceso

Probamos con `hydra` y hacemos un ataque de fuerza bruta:

```bash
hydra -L /usr/share/wordlists/rockyou.txt -p "JIFGHDS87GYDFIGD" ssh://172.17.0.2
```

<figure><img src="../../.gitbook/assets/Pasted image 20251013153836.png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Una vez dentro de la máquina, nos encontramos con un script curioso. Además, vemos que tenemos permisos para ejecutar ese script con `python` como cualquier usuario (`root` entre ellos).

<figure><img src="../../.gitbook/assets/Pasted image 20251013154603.png" alt=""><figcaption></figcaption></figure>

A partir de aquí, se me ocurrieron varias maneras de escalar privilegios en esta máquina.

## Ruta 1:

El script existente en `/opt/` lo que hace es copiar el fichero `/opt/script.py` a `/tmp/script_backup.py`. Si yo muevo este script a otra carpeta (o incluso lo elimino, pero lo necesito para explicar la siguiente ruta xd) y creo mi script malicioso en `/opt/script.py`, puedo ejecutarlo como `root`.

<figure><img src="../../.gitbook/assets/Pasted image 20251013160122.png" alt=""><figcaption></figcaption></figure>

El script original que había, lo he movido a la carpeta `/opt/hola`, y el script que he dejado en la carpeta `/opt/` nos abre una `shell`. Claro, si este script lo ejecutamos como `root`, nos abrirá una `shell` como `root`. Como tenemos permisos para ejecutar ese script en concreto como `root`, podemos escalar privilegios fácilmente.

<figure><img src="../../.gitbook/assets/Pasted image 20251013160246.png" alt=""><figcaption></figcaption></figure>

## Ruta 2:

La manera que creo que está diseñada para esta máquina (por el nombre te puedes hacer una idea) es hacer un `library hijacking`. El script utiliza la librería `shutil`, por lo que si creamos el archivo shutil.py detro de `/opt`, `python` va a usar este archivo antes de buscar en otras rutas.

Modificamos la función `copy` para que ejecute una `shell`. Después, ejecutamos con `sudo` el script.

<figure><img src="../../.gitbook/assets/Pasted image 20251013161534.png" alt=""><figcaption></figcaption></figure>
