---
description: '🧠Dificultad: Fácil | 🔓16/10/2025'
---

# Vulnvault

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/Pasted image 20251015190452.png" alt=""><figcaption></figcaption></figure>

Como hay un servicio `HTTP` activo, hacemos un escaneo de directorios:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://$IPTARGET/FUZZ -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/Pasted image 20251015192905.png" alt=""><figcaption></figcaption></figure>

En el directorio `/index.php` hay un apartado llamado `Acerca de` donde nos da a entender que la función de generar reportes puede ser vulnerable a `Command Injection`:&#x20;

<figure><img src="../../.gitbook/assets/Pasted image 20251015190557.png" alt=""><figcaption></figcaption></figure>

El apartado para generar nuestros reportes es el siguiente:

<figure><img src="../../.gitbook/assets/Pasted image 20251015190858.png" alt=""><figcaption></figcaption></figure>

Si probamos a introducir en algún campo la siguiente cadena:

```plaintext
; echo "Prueba"
```

Vemos que se ha ejecutado el comando.

<figure><img src="../../.gitbook/assets/Pasted image 20251015191300.png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Vamos a intentar inyectar una reverse shell. Para ello, vamos a ver algunos binarios que hay disponibles en el sistema con el comando `which`:

<figure><img src="../../.gitbook/assets/Pasted image 20251015191637.png" alt=""><figcaption></figcaption></figure>

Tenemos `python3`, `bash` y `sh`. Aunque no aparezca en la captura, también tenemos `php` y otros binarios que nos pueden servir.

Probamos a mandar la reverse shell:

<figure><img src="../../.gitbook/assets/Pasted image 20251015191713.png" alt=""><figcaption></figcaption></figure>

Pero a nuestra máquina no nos llega ninguna petición. Intentando con distintas reverse shells, vemos que no funciona correctamente. Es posible que el código `php` esté escapando algunos caracteres.

Vamos a intentar ver qué usuarios hay en el sistema:

<figure><img src="../../.gitbook/assets/Pasted image 20251016164232.png" alt=""><figcaption></figcaption></figure>

Existe un usuario llamado `samara`. Curiosamente tenemos permisos para ver su `/home`:

<figure><img src="../../.gitbook/assets/Pasted image 20251016164328.png" alt=""><figcaption></figcaption></figure>

Podemos obtener el `id_rsa` de `samara` para acceder por `ssh`:&#x20;

<figure><img src="../../.gitbook/assets/Pasted image 20251016164421.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Pasted image 20251016164617.png" alt=""><figcaption></figcaption></figure>

Lo copiamos a nuestra máquina y le damos permisos de lectura/escritura:

```bash
chmod 600 id_rsa
```

Ahora si podemos acceder por `ssh` con el `id_rsa`:

<figure><img src="../../.gitbook/assets/Pasted image 20251016164554.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Pasted image 20251016164759.png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Una vez dentro de la máquina, listamos los servicios que están corriendo.

<figure><img src="../../.gitbook/assets/Pasted image 20251016164941.png" alt=""><figcaption></figcaption></figure>

Nos fijamos que se está ejecutando el script localizado en `/usr/local/bin/echo.sh` como `root`. Para este script, tenemos permisos de escritura, por lo que podemos modificarlo.

<figure><img src="../../.gitbook/assets/Pasted image 20251016164927.png" alt=""><figcaption></figcaption></figure>

En mi caso, voy a hacer que el binario `/bin/bash` tenga el bit `SUID` activado para obtener una shell con `root` fácil.

<figure><img src="../../.gitbook/assets/Pasted image 20251016165200.png" alt=""><figcaption></figcaption></figure>

Si esperamos, vemos que ahora el binario efectivamente tiene el bit `SUID` activado. Ejecutamos `/bin/bash -p` para lanzar una shell con permisos del propietario (`root`):

<figure><img src="../../.gitbook/assets/Pasted image 20251016165256.png" alt=""><figcaption></figcaption></figure>
