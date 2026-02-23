---
description: '🧠Dificultad: Fácil | 🔓16/02/2026'
---

# 🟩 Whoiam

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/image (148).png" alt=""><figcaption></figcaption></figure>

Vemos que solo está abierto el puerto 80 con un WordPress. Hacemos un escaneo de directorios con `ffuf`:

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (143).png" alt=""><figcaption></figcaption></figure>

Vemos un directorio que nos llama la atención: `/backups`.&#x20;

<figure><img src="../../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

Descargamos el zip y lo extraemos en nuestra máquina. Vemos que contiene un fichero con credenciales.

<figure><img src="../../.gitbook/assets/image (142).png" alt=""><figcaption></figcaption></figure>

Probamos estas credenciales en el panel de WordPress y obtenemos acceso al panel de administrador.

<figure><img src="../../.gitbook/assets/image (144).png" alt=""><figcaption></figcaption></figure>

Si echamos un vistazo a la versión de los plugins, encontramos un plugin que es vulnerable (`Modern Events Calendar 5.16.2`):

<figure><img src="../../.gitbook/assets/image (145).png" alt=""><figcaption></figcaption></figure>

Vemos un RCE que necesita autenticación (la tenemos).

## 🚪 Ganando acceso

Configuramos metasploit para usar el exploit.

<figure><img src="../../.gitbook/assets/image (146).png" alt=""><figcaption></figcaption></figure>

Una vez tenemos todo configurado, conseguimos una sesión de meterpreter y abrimos una shell.

<figure><img src="../../.gitbook/assets/image (147).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Una vez dentro de la máquina, vemos los permisos que tenemos con sudo con el usuario `www-data`:

<figure><img src="../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

Podemos usar el binario `/usr/bin/find` como `rafa` sin necesidad de contraseña para conseguir una shell con el parámetro `-exec`:

```bash
find . -exec /bin/sh \; -quit
```

<figure><img src="../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

Como `rafa` volvemos a ver los permisos con `sudo`:

<figure><img src="../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

Tenemos permisos para ejecutar `/usr/sbin/debugfs` como `ruben` sin contraseña. En [gtfobins](https://gtfobins.org/gtfobins) vemos un exploit que podemos usar para conseguir una shell.

<figure><img src="../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

De nuevo, vemos los permisos que tenemos con `ruben`. Podemos ejecutar `/bin/bash /opt/penguin.sh` como `root` sin necesidad de contraseña. El script no lo podemos modificar porque no tenemos permisos de escritura en él.

<figure><img src="../../.gitbook/assets/image (139).png" alt=""><figcaption></figcaption></figure>

Si introducimos como variable `a[$(/bin/bash >&2 1>&2)]`, lo que va a hacer el intérprete es intentar calcular la dirección de un array (el array `a[]`). Para ello, va a evaluar lo que hay dentro de la expresión `$()`, que corresponde a un bash.

<figure><img src="../../.gitbook/assets/image (140).png" alt=""><figcaption></figcaption></figure>
