---
description: '🧠Dificultad: Fácil | 🔓13/10/2025'
---

# Pntopntobarra

## 🕵️ Reconocimiento

Hacemos un escaneo de puertos con `nmap`:

```bash
nmap -sC -sV -Pn $IPTARGET | tee nmap
```

<figure><img src="../../.gitbook/assets/Pasted image 20251013183000 (1).png" alt=""><figcaption></figcaption></figure>

Al entrar a la página web, encontramos lo siguiente:

<figure><img src="../../.gitbook/assets/Pasted image 20251013183654.png" alt=""><figcaption></figcaption></figure>

Si le damos clic al primer botón, se borra todo el sistema y tendrás que reiniciar la máquina (no preguntéis por qué lo sé). El segundo botón nos redirecciona al siguiente directorio:

```url
http://172.17.0.2/ejemplos.php?images=./ejemplo1.png
```

<figure><img src="../../.gitbook/assets/Pasted image 20251013183902.png" alt=""><figcaption></figcaption></figure>

Probamos a meter una entrada para ver si el parámetro es vulnerable a `LFI` y ver el fichero `/etc/passwd`:

```url
http://172.17.0.2/ejemplos.php?images=../../../../etc/passwd
```

<figure><img src="../../.gitbook/assets/Pasted image 20251013184121.png" alt=""><figcaption></figcaption></figure>

El parámetro `images` es vulnerable a un `LFI`.

## 🚪 Ganando acceso

Como vemos en el fichero, existe un usuario llamado `nico` con una terminal interactiva. Podemos probar a obtener su clave privada RSA y conectarnos a la máquina por `ssh`:

<figure><img src="../../.gitbook/assets/Pasted image 20251013185640.png" alt=""><figcaption></figcaption></figure>

Copiamos este fichero a nuestra máquina, lo movemos a nuestra carpeta `/home/usuario/.ssh` y le damos permisos con `sudo chmod 600 /home/usuario/.ssh/id_rsa`. Una vez hecho esto, podemos acceder a la máquina:

<figure><img src="../../.gitbook/assets/Pasted image 20251013185525.png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Una vez dentro, vemos que tenemos permiso para ejecutar el binario `env` con `sudo` como cualquier usuario sin necesidad de contraseña. Con `env` podemos montar un entorno ejecutando `sh` como `root`, dándonos acceso así a una terminal con privilegios de administrador:

```bash
sudo env /bin/sh
```

<figure><img src="../../.gitbook/assets/Pasted image 20251013190021.png" alt=""><figcaption></figcaption></figure>
