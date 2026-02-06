---
description: '🧠Dificultad: Fácil | 🔓08/09/2025'
---

# 🟩 Escolares

## 🕵️ Reconocimiento

Empezamos con un escaneo de puertos con `nmap`:

<figure><img src="../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

Vemos que hay abierto un servicio HTTP y otro SSH. En el código fuente del servicio web encontramos este comentario:

```html
<!-- INFORMACION DEL PERSONAL -->
<!-- ./profesores.html -->
```

Si nos vamos al directorio `/profesores.html`, vemos lo siguiente:

<figure><img src="../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

Encontramos una lista con información de posibles usuarios. Si le echamos un vistazo al `HTML` de la página web, vemos que el profesor `Luis` es el administrador del wordpress:

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

Esto nos indica que seguramente se esté hosteando un wordpress en este servidor. Realizamos un escaneo de directorios con `ffuf`:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://$IPTARGET/FUZZ -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

Accediendo al directorio `/wordpress`, me he encontrado con el fallo de que muchos recursos que está intentando pedir no los resuelve. Esto se debe a que los está intentando pedir del dominio `escolares.dl`, así que lo añadimos al `/etc/hosts`. Después de esto, los recursos cargan correctamente.

<figure><img src="../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

La única publicación que hay subida al wordpress es del usuario `luisillo`, el admin del wordpress.

## 🚪 Ganando acceso

Como tenemos información del administrador, podemos crear una lista de posibles contraseñas para hacer fuerza bruta. Con `cupp -i` generamos una wordlist con la información que encontramos en `/profesores.html`:

<figure><img src="../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

Con `burpsuite` podemos hacer un ataque al formulario de login de wordpress con la wordlist generada. Capturando una petición `POST` al intentar iniciar sesión, podemos personalizarla para hacer fuerza bruta con el usuario `luisillo` y las contraseñas de la wordlist de `cupp`. Cuando fallas un inicio de sesión, aparece una advertencia indicando que la contraseña no es correcta, por lo que usaremos esa frase como flag para saber si el inicio de sesión fue exitoso o no.

<mark style="color:orange;">Sé que no es el mejor método para hacerlo, pero me queda pendiente aprender a atacar páginas wordpress y a usar las herramientas que hay dedicadas para esto. En próximas máquinas de wordpress intentaré haber aprendido mejores métodos.</mark>

<figure><img src="../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

Tras unos minutos, obtenemos la respuesta que queríamos:

<figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

La contraseña de `luisillo` es `Luis1981`. Iniciamos sesión en el panel.

<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

Como somos administradores, podemos subir una reverse shell en `php` al gestor de archivos.

<figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

En el directorio `/home` vemos un fichero con la contraseña de `luisillo`:

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

Si comprobamos los permisos de `sudo`, vemos que tenemos permisos para ejecutar el binario `awk` como `root`:

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

En [gtfobins](https://gtfobins.github.io/) encontramos un comando para explotar esta configuración y escalamos como `root`:

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>
