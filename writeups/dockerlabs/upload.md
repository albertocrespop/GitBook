---
description: '🧠Dificultad: Fácil | 🔓04/02/2026'
---

# Upload

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos usando `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET
```

<figure><img src="../../.gitbook/assets/image (105).png" alt=""><figcaption></figcaption></figure>

Solo está abierto el puerto del servicio HTTP. Realizamos un escaneo de directorios en el servicio web.

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

Si entramos a la página, encontramos un menú muy simple donde podemos subir nuestros archivos. Probamos a subir un archivo `.php` de prueba para ver si lo filtra.

<figure><img src="../../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

Se ha subido correctamente, por lo que este puede ser nuestro vector de ataque.

## 🚪 Ganando acceso

Subimos una reverse shell en PHP al servidor. En mi caso utilizaré la reverse shell de pentestmonkey.

<figure><img src="../../.gitbook/assets/image (108).png" alt=""><figcaption></figcaption></figure>

Nos ponemos a la escucha con netcat en nuestra máquina y ejecutamos el fichero PHP que hemos subido al servidor.

<figure><img src="../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Una vez metidos en la máquina, verificamos los privilegios que tenemos con `sudo`:

<figure><img src="../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar el binario `/usr/bin/env` como `root` sin necesidad de contraseña. Esto nos otorga una shell directa como `root`.

<figure><img src="../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>
