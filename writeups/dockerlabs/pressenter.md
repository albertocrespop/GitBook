---
description: '🧠Dificultad: Fácil | 🔓16/10/2025'
---

# Pressenter

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/Pasted image 20251016205727.png" alt=""><figcaption></figcaption></figure>

En la página principal no encontramos ninguna funcionalidad relevante. En el pie de página vemos una pista:

```plaintext
Find us at pressenter.hl
```

<figure><img src="../../.gitbook/assets/Pasted image 20251016210926.png" alt=""><figcaption></figcaption></figure>

Si accedemos a `pressenter.hl` después de meterlo al fichero `/etc/hosts`, nos encontramos con una nueva página:

<figure><img src="../../.gitbook/assets/Pasted image 20251016211113.png" alt=""><figcaption></figcaption></figure>

Investigando un poco podemos averiguar que estamos ante un wordpress (por ejemplo, si inspeccionamos el código de la página vemos redirecciones a directorios empezando por `wp`).

Usando `wpscan` podemos encontrar los usuarios que hay registrados en el wordpress:

```bash
wpscan --url http://pressenter.hl --enumerate --api-token <API>
```

<figure><img src="../../.gitbook/assets/Pasted image 20251016224816.png" alt=""><figcaption></figcaption></figure>

Ejecutamos un ataque de fuerza bruta con los usuarios que hemos encontrado junto al `rockyou.txt`:

```bash
wpscan --password-attack xmlrpc -t 20 -U pressi, hacker -P /usr/share/wordlists/rockyou.txt --url http://pressenter.hl
```

<figure><img src="../../.gitbook/assets/Pasted image 20251016224347.png" alt=""><figcaption></figcaption></figure>

Podemos editar algún fichero `php` de un tema que no esté en uso, como `Twenty Twenty Three`. Editamos el fichero `patterns/hidden-404.php` con una webshell:

<figure><img src="../../.gitbook/assets/Pasted image 20251016231757.png" alt=""><figcaption></figcaption></figure>

Ahora si lanzamos una petición a dicho fichero y le pasamos un comando al parámetro `cmd`, nos muestra el resultado del comando.

<figure><img src="../../.gitbook/assets/Pasted image 20251016231727.png" alt=""><figcaption></figcaption></figure>

## 🚪 Ganando acceso

Si subimos una reverse shell (en mi caso, la de pentestmonkey) y lanzamos una petición al archivo `php`, obtenemos acceso a la máquina:

<figure><img src="../../.gitbook/assets/Pasted image 20251016231945.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Pasted image 20251016231927.png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Mirando el fichero `/etc/passwd`, encontramos un usuario llamado `enter`:

<figure><img src="../../.gitbook/assets/Pasted image 20251016232650.png" alt=""><figcaption></figcaption></figure>

Buscamos el fichero de configuración de wordpress para ver si encontramos información sensible:

```bash
find / -type f -name "wp-config.php" 2>/dev/null
```

<figure><img src="../../.gitbook/assets/Pasted image 20251016232410.png" alt=""><figcaption></figcaption></figure>

Encontramos las credenciales de una base de datos con el usuario `admin` y la contraseña `rooteable`. Accedemos al a base de datos con:

```bash
mysql -u admin -p
```

Una vez dentro, enumeramos las `DATABASES` y `TABLES` de la base de datos. Encontramos una tabla dentro de la base de datos `wordpress` en la tabla `wp_usernames`:

<figure><img src="../../.gitbook/assets/Pasted image 20251016234945.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Pasted image 20251016235009.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Pasted image 20251016232636.png" alt=""><figcaption></figcaption></figure>

Probamos a entrar con las credenciales que hemos encontrado:

<figure><img src="../../.gitbook/assets/Pasted image 20251016232712.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Pasted image 20251016232759.png" alt=""><figcaption></figcaption></figure>

La escalada a `root` es contraintuitiva, puesto que se reutilizan credenciales del usuario `enter`:

<figure><img src="../../.gitbook/assets/Pasted image 20251016234220.png" alt=""><figcaption></figcaption></figure>
