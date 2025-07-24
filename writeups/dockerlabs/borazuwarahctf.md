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

# BorazuwarahCTF

La máquina está corriendo un servicio web. Esto lo he comprobado introduciendo la IP en el navegador. Nos encontramos con una foto de un huevo kinder.

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250622010559.png" alt="" width="251"><figcaption></figcaption></figure></div>

Antes de comprobar directorios ocultos, creo que la imagen nos puede decir algo, así que procedo a descargarme la imagen para analizarla. Si usamos el comando `exiftool` podemos ver los metadatos de la imagen:

<figure><img src="../../.gitbook/assets/Pasted image 20250622010836.png" alt=""><figcaption></figcaption></figure>

En el campo `description` encontramos un usuario: **`borazuwarah`**. Sobre la contraseña no nos dice nada. Puede que haya un archivo oculto en su interior, ya que en los metadatos de la imagen aparece el propio programa `exiftool` utilizado para esconder imágenes y metadatos ocultos. Vamos a comprobar si la imagen tiene información oculta, para ello utilizo el comando `steghide extract -sf imagen.jpeg` y vemos si hay suerte.

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250622011152.png" alt=""><figcaption></figcaption></figure></div>

Efectivamente había un fichero oculto, pero no nos ha valido para nada. Nos indica que la pista está dentro de la imagen, por lo que creo que se refiere a lo que ya hemos encontrado anteriormente. He supuesto que este usuario se puede utilizar para acceder por SSH a la máquina, por lo que si no tenemos más información, podemos intentar hacerle un ataque de fuerza bruta al SSH para el usuario encontrado.

```bash
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://$IPTARGET
```

```bash
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: borazuwarah   password: 123456
1 of 1 target successfully completed, 1 valid password found
```

Una vez entramos por ssh a la máquina, ejecutamos el comando `sudo -l` para ver los permisos que tenemos como sudoer.

```bash
sudo -l
```

```
Matching Defaults entries for borazuwarah on 7eec632a86c9:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User borazuwarah may run the following commands on 7eec632a86c9:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash
```

Podemos correr el ejecutable `/bin/bash` con sudo sin necesidad de contraseña.

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250722015209.png" alt=""><figcaption></figcaption></figure></div>
