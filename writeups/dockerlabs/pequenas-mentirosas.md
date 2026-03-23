---
description: '🧠Dificultad: Fácil | 🔓23/03/2026'
---

# 🟩 Pequeñas Mentirosas

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

{% code overflow="wrap" %}
```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Solo están abiertos el puerto SSH y HTTP. Realizamos un escaneo de directorios sobre el servicio web:

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Si entramos a `index.html` encontramos la siguiente pista:

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Hacemos un ataque de diccionario para el usuario `a` con la wordlist `rockyou.txt`:

{% code overflow="wrap" %}
```bash
hydra -l a -P /usr/share/wordlists/rockyou.txt ssh://$IPTARGET
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Miramos los usuarios del sistema. Encontramos un usuario llamado `spencer`.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Buscamos todos los ficheros con la extensión `.txt`. Encontramos algunos interesantes.&#x20;

{% code overflow="wrap" %}
```bash
find / -type f -name "*.txt" 2>/dev/null
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

La forma correcta de continuar sería intentar romper el hash de spencer (`hash_spencer.txt`) en nuestra máquina atacante, pero como sabemos el nombre del usuario también podemos hacer ataque de diccionario, aunque en entornos reales esto sería menos recomendable. Aún así he escogido la segunda opción.

{% code overflow="wrap" %}
```bash
hydra -l spencer -P /usr/share/wordlists/rockyou.txt ssh://$IPTARGET
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Las credenciales son `spencer:password1`. Entramos como spencer a la máquina y comprobamos los permisos que tenemos con `sudo`:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Podemos ejecutar `/usr/bin/python3` como `root` sin necesidad de contraseña. Con esto podemos conseguir una shell.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>
