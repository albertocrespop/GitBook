---
description: '🧠Dificultad: Fácil | 🔓28/08/2025'
---

# 🟩 AguaDeMayo

## 🕵️ Reconocimiento

Comenzamos haciendo un escaneo de puertos con `nmap` :

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Tenemos un servicio web corriendo en la máquina, además del SSH. Enumerando nos encontramos con un directorio llamado `images` donde hay habilitado un directory indexing. Observamos una imagen, `agua_ssh.jpg`.

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<div align="left"><figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

## 🚪 Ganando acceso

Con el comando `strings` no hay nada que resalte, igualmente probando fuerza bruta con el usuario `agua` no conseguimos nada, tampoco buscando exploits con la versión de Apache conocida.

Mirando el código fuente de la página inicial de Apache, encuentro un comentario al final:

```html
<!--
++++++++++[>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>++++++++++++>++++++++++>++++++++++++>++++++++++>+++++++++++>+++++++++++>+>+<<<<<<<<<<<<<<<<<-]>--.>+.>--.>+.>---.>+++.>---.>---.>+++.>---.>+..>-----..>---.>.>+.>+++.>.
-->
```

Preguntándole a ChatGPT, descubro que es código Brainfuck. Con un intérprete online, decodifico la cadena:

<div align="left"><figure><img src="../../.gitbook/assets/image (15) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Probando esta cadena como contraseña para el usuario `agua`, accedo a la máquina por SSH.

<figure><img src="../../.gitbook/assets/image (16) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalando privilegios

<figure><img src="../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

Vemos que tenemos permisos para ejecutar el binario `bettercap` como `root` sin necesidad de introducir contraseña.&#x20;

Al ejecutar `bettercap`, vemos con el comando `help` que podemos ejecutar comandos de shell añadiendo al principio un `!`. Con esto, conseguimos ejecutar comandos como `root`.

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

Adicionalmente, si ejecutamos el siguiente comando como `root`

```bash
chmod +s /bin/bash
```

podemos abrir una terminal como `root` con `/bin/bash -p`.
