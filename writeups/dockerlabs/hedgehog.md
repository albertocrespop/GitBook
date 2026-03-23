---
description: '🧠Dificultad: Muy fácil | 🔓28/07/2025'
---

# 🟦 Hedgehog

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos usando `nmap` :

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Si accedemos a la página que aloja el servidor desde el navegador, nos muestra un simple mensaje en texto plano:

```
tails
```

Aparentemente no nos dice nada, pero puede ser un nombre de usuario de otro servicio de la máquina, como el `ssh`.

El escaneo de directorios no nos muestra nada relevante (solo el directorio `server-status` que da un `403` ).

## 🚪 Ganando acceso

Al probar a hacer fuerza bruta al servicio `ssh` con `rockyou.txt` con el usuario `tails` , no obtenemos nada, algo que parece raro. El nombre de usuario me recordó al comando en linux `tail` que muestra la últimas líneas de un fichero. Intentando hacer fuerza bruta con las últimas 100 líneas del `rockyou.txt` , no he conseguido nada. Por ello, para no estar probando con `n` líneas cada vez, podemos generar una wordlist revertida de la que hayamos escogido. Con el comando `tac` (`cat` al revés) podemos leer las líneas de un fichero al revés:

```bash
tac /usr/share/wordlists/rockyou.txt > reversed_rockyou.txt
```

También nos damos cuenta que, especialmente, en las últimas líneas del `rockyou` hay muchas líneas con espacios seguidos de la contraseña, por lo que eliminaremos dichos espacios:

<div align="left"><figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

```bash
tr -d ' ' < reversed_rockyou.txt > reversed_filtered_rockyou.txt
```

<div align="left"><figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Ahora sí, probamos la wordlist con `hydra`:

```bash
hydra -l tails -P reversed_filtered_rockyou.txt ssh://$IPTARGET
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalando privilegios

Vemos con `sudo -l` que tenemos permisos para ejecutar cualquier binario como el usuario `sonic` sin necesidad de contraseña:

<div align="left"><figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Así que podemos abrir un `bash`:

<div align="left"><figure><img src="../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Con el usuario `sonic`, tenemos permisos para ejecutar todos los binarios como `root` sin necesidad de contraseña, así que iniciamos otra sesión en `bash`:

<div align="left"><figure><img src="../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>
