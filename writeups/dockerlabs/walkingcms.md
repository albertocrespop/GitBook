---
description: '🧠Dificultad: Fácil | 🔓15/02/2026'
---

# 🟩 WalkingCMS

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

Vemos que solo está el puerto 80 activo (servicio web). Si realizamos un escaneo de directorios encontramos lo siguiente:

{% code overflow="wrap" expandable="true" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

Estamos ante un WordPress. Vamos a enumerarlo con la herramienta `wpscan`:

{% code overflow="wrap" %}
```bash
wpscan --url http://$IPTARGET/wordpress --enumerate --api-token <token>
```
{% endcode %}

Esta herramienta nos encuentra un usuario que podemos utilizar en un ataque de diccionario.

<figure><img src="../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

Para realizar este ataque se ha utilizado la wordlist `rockyou.txt`.

{% code overflow="wrap" %}
```bash
wpscan --password-attack xmlrpc -t 20 -U mario -P /usr/share/wordlists/rockyou.txt --url http://$IPTARGET/wordpress
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (129).png" alt=""><figcaption></figcaption></figure>

Encontramos una combinación para entrar al panel de WordPress: `mario:love`.

<figure><img src="../../.gitbook/assets/image (130).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Una vez dentro, podemos modificar un fichero php de una plantilla de WordPress para inyectar una reverse shell. Cada vez que solicitemos ese recurso dentro del directorio `wp-content`, se ejecutará el código que nos da acceso a la máquina.

En este caso, voy a editar el fichero `index.php` de la plantilla `twentytwentytwo` para añadir la reverse shell de pentestmonkey.

<figure><img src="../../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

Una vez actualizado el recurso, lo solicitamos a través del siguiente enlace. Previamente, nos ponemos a la escucha con `netcat`.

```
http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php
```

<figure><img src="../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Si buscamos los binarios con el bit SUID activado, encontramos uno potencialmente peligroso: `/usr/bin/env`. Con este binario podemos ejecutar cualquier comando o binario, y al tener el bit SUID activo, lo podemos hacer con los privilegios del propietario del archivo (`root`).

<figure><img src="../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>
