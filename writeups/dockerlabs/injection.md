---
description: '🧠Dificultad: Muy fácil | 🔓14/07/2025'
---

# 🟦 Injection

Tras desplegar la máquina, pruebo a abrir el navegador y ver si tiene alojada una página web. En efecto, aloja una página con un Login.

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250620000152 (1).png" alt="" width="430"><figcaption></figcaption></figure></div>

Como la máquina se llama Injection, me lleva a pensar que es vulnerable a SQL Injection. Pruebo poniendo una `'` en el campo user y en el campo de contraseña cualquier cosa.

<figure><img src="../../.gitbook/assets/Pasted image 20250620000411.png" alt=""><figcaption></figcaption></figure>

Muestra un mensaje de error de SQL. El servicio web no está sanitizando la entrada antes de hacer la consulta a la base de datos. Pruebo un query sencillo para hacer bypass al Login, `' or '1'='1' --` .

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250620000951.png" alt="" width="536"><figcaption></figcaption></figure></div>

El bypass ha sido un éxito. Se muestra en pantalla un nombre de usuario y una contraseña. Pruebo a loguearme por SSH con estas credenciales.

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250620001358.png" alt="" width="562"><figcaption></figcaption></figure></div>

Hemos conseguido el acceso a la máquina. Para escalar privilegios, vamos a ver primero qué binarios hay en el sistema cuyo propietario sea `root` con el bit SUID activado.

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250620002142.png" alt="" width="410"><figcaption></figcaption></figure></div>

Buscando cada binario en la página de [gtfobins](https://gtfobins.github.io/), encuentro una explotación para el binario `env`, cuyo propietario en nuestro caso es `root`. Este exploit se logra ejecutando `/bin/sh -p` a través de `env`, que preserva el UID efectivo del proceso (`root`) gracias al uso del flag `-p`, por lo que se abrirá una shell como `root`.

<div align="left"><figure><img src="../../.gitbook/assets/Pasted image 20250620002446.png" alt=""><figcaption></figcaption></figure></div>
