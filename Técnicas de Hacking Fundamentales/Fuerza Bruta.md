# Fuerza Bruta y Cracking de Credenciales

Esta guía recopila las herramientas más utilizadas para realizar auditorías de autenticación en servicios de red y para recuperar contraseñas a partir de hashes o archivos protegidos.

---

# Hydra

Hydra permite realizar ataques de diccionario contra numerosos protocolos.

## Contraseña desconocida

```bash
hydra -l <usuario> -P /usr/share/wordlists/rockyou.txt <objetivo>
```

## Usuario desconocido

```bash
hydra -L /usr/share/wordlists/metasploit/unix_users.txt -p <contraseña> <objetivo>
```

---

# Objetivos compatibles

## SSH

```text
ssh://<IP>
```

Acceso tras obtener credenciales:

```bash
ssh <usuario>@<IP>
```

---

## FTP

```text
ftp://<IP>
```

Acceso:

```bash
ftp <IP>
```

---

## MySQL

```text
mysql://<IP>
```

---

## Formularios Web (POST) — el caso habitual

El formato de `http-post-form` tiene tres partes separadas por `:` →
`"<ruta>:<cuerpo con ^USER^/^PASS^>:<condición de fallo>"`.

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP> \
  http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Login Failed"
```

- `F=<texto>` → marca el intento como **fallido** si la respuesta contiene ese texto (ej. `Login Failed`, `incorrecta`).
- `S=<texto>` → al revés: marca como **éxito** si aparece ese texto (ej. `Dashboard`, `Logout`).

> **Consejo:** intercepta el login con Burp para copiar la **ruta exacta**, los **nombres reales** de los campos (`user`/`pass` pueden llamarse `username`/`password`) y el mensaje de error literal. Un solo carácter mal en la cadena hace que Hydra marque todo como válido o todo como inválido.

### Con HTTPS, puerto o cabeceras

```bash
# HTTPS -> usa https-post-form
hydra -l admin -P rockyou.txt <IP> https-post-form "/login:username=^USER^&password=^PASS^:F=inválid"

# Puerto distinto y cabeceras (cookie/CSRF token) con H=
hydra -l admin -P rockyou.txt -s 8080 <IP> http-post-form \
  "/login:user=^USER^&pass=^PASS^:F=error:H=Cookie: PHPSESSID=abc123"
```

---

## Formularios Web (GET)

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <IP> \
  http-get-form "/login.php:user=^USER^&pass=^PASS^:F=Login Failed"
```

> **Nota:** usa `-f` para detener Hydra en cuanto encuentre una credencial válida, y `-V` para ver cada intento. Autenticación HTTP básica (la ventana emergente del navegador) usa el módulo `http-get` a secas, no `http-get-form`.

---

# Metasploit

Metasploit dispone de módulos auxiliares para comprobar credenciales sobre distintos servicios.

## SSH

```text
search ssh_login
```

## FTP

```text
search ftp_login
```

## SMB

```text
search smb_login
```

---

# John the Ripper

Herramienta utilizada para recuperar contraseñas a partir de hashes.

## Archivos ZIP

Extraer el hash:

```bash
zip2john <archivo.zip> > hash
```

Crackear la contraseña:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

---

## Bases de datos KeePass (.kdbx)

Extraer el hash:

```bash
keepass2john <archivo.kdbx> > hash
```

Crackear la contraseña:

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

---

## Especificar el formato del hash

Si John no detecta automáticamente el formato:

Identificar el tipo de hash:

```bash
hash-identifier
```

Ejecutar John indicando el formato:

```bash
john --format=<formato_hash> --wordlist=/usr/share/wordlists/rockyou.txt hash
```

---

# Diccionarios útiles

```text
/usr/share/wordlists/rockyou.txt
```

```text
/usr/share/wordlists/metasploit/unix_users.txt
```

```text
/usr/share/seclists/
```

---

# Herramientas relacionadas

| Herramienta       | Descripción                                                        |
| ----------------- | ------------------------------------------------------------------ |
| `Hydra`           | Ataques de diccionario y fuerza bruta contra múltiples protocolos. |
| `John the Ripper` | Recuperación de contraseñas a partir de hashes.                    |
| `hash-identifier` | Identificación del tipo de hash.                                   |
| `zip2john`        | Extrae hashes de archivos ZIP.                                     |
| `keepass2john`    | Extrae hashes de bases de datos KeePass.                           |
| `Metasploit`      | Framework con módulos auxiliares para comprobar credenciales.      |

---

> **Consejo:** Antes de lanzar un ataque de fuerza bruta, intenta obtener credenciales mediante enumeración, archivos de configuración, recursos compartidos o reutilización de contraseñas. Esto suele ser mucho más rápido y genera menos ruido que un ataque de diccionario.
