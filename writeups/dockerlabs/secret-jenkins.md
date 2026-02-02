---
description: '🧠Dificultad: Fácil | 🔓30/01/2026'
---

# Secret Jenkins

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con nmap:

```bash
nmap -sC -sV -Pn $IPTARGET
```

<figure><img src="../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

Si entramos a la página principal en el puerto 8080, vemos que es un Jenkins (se intuye por el nombre de la máquina también). Con whatweb podemos averiguar la versión de Jenkins ante la que estamos.

<figure><img src="../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

Si buscamos la versión de Jenkins vemos que existe un exploit que nos habilita un LFI. Con esta vulnerabilidad podemos listar ficheros en el sistema.

## 🚪 Ganando acceso

Utilizamos el siguiente [exploit](https://www.exploit-db.com/exploits/51993) para listar el fichero `/etc/passwd`:

<figure><img src="../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

Probamos a hacer fuerza bruta al usuario `bobby`, y encontramos una contraseña para poder acceder por SSH a la máquina.

<figure><img src="../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Vemos los permisos que tenemos con sudo para el usuario bobby:

<figure><img src="../../.gitbook/assets/image (103).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar python3 como el usuario `pinguinito` sin necesidad de introducir su contraseña. Ejecutamos una shell con python3 usando sudo:

<figure><img src="../../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

Ahora vemos los permisos que tenemos con el usuario `pinguinito`:

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Vemos que se puede ejecutar la orden `/usr/bin/python3 /opt/script.py` como cualquier usuario sin necesidad de contraseña. El problema viene cuando nos damos cuenta que ese script no tiene permisos de escritura, por lo que no podemos modificarlo para obtener una shell como root. Lo que sí podemos hacer es un library hijacking de la librería `shutil` para la función `shutil.copy`, que es la que usa el script.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

La función copy en nuestra librería falsa nos ejecuta una shell. Ahora ejecutamos el script y obtenemos una shell como root (por usar sudo).

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>
