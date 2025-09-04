---
description: '🧠Dificultad: Fácil | 🔓04/09/2025'
---

# DockerLabs

## 🕵️ Reconocimiento

Realizamos un escaneo de puertos con `nmap`:

<figure><img src="../../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

Solo está abierto el puerto 80. Vemos cómo es la página que se está alojando desde un navegador:

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

Es una página muy parecida a la de DockerLabs. Realizamos un escaneo de directorios para ver rutas ocultas con `ffuf`:

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://$IPTARGET/FUZZ -e .html,.php,.txt,.xml,.js
```

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

La ruta `/uploads` puede guardar los ficheros que se suban desde otra ruta. Si observamos el directorio `/machine.php`, vemos un formulario para subir nuestras máquinas en `.zip`:

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

Los ficheros que se suban correctamente, se almacenan en la ruta `/uploads`.

Comprobamos qué extensiones son válidas en este formulario con la herramienta `fuzz`. Para ello, he capturado una petición con `burpsuite` y la he guardado en un fichero en mi máquina. En la parte donde se indica la extensión del fichero, he añadido la palabra `FUZZ` para que `ffuf` la reconozca:

<figure><img src="../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>

Nos quedaremos con las peticiones cuya respuesta contenga la cadena `ha sido subido correctamente`.

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Fuzzing/extensions-most-common.fuzz.txt -request request -request-proto http -mr "ha sido subido correctamente"
```

<figure><img src="../../.gitbook/assets/image (55).png" alt="" width="563"><figcaption></figcaption></figure>

Este PHP acepta las extensiones `zip` y `phar`. Nos interesa la extensión `phar`, ya que puede ser interpretada y ejecutada como código PHP en el servidor, permitiéndonos subir una reverse shell.

## 🚪 Ganando acceso

Subimos nuestra reverse shell al servidor con la extension `.phar`. La reverse shell que usaré es la de [pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

<figure><img src="../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Comprobamos los permisos que tenemos con `sudo`:

<figure><img src="../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar `cut` y grep como `root`. Esto nos puede servir para leer el fichero que queramos. Indagando por los directorios, encontramos un fichero en `/opt/nota.txt`:

<figure><img src="../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

Probamos la contraseña para `root`:

<figure><img src="../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>
