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

## Formularios Web (GET)

```text
<IP> http-get /login.php -s <puerto> -f
```

---

## Formularios Web (POST)

```text
<IP> http-post-form "/login.php:user=^USER^&pass=^PASS^:Login Failed"
```

> **Nota:** Sustituye `Login Failed` por el mensaje de error que devuelve la aplicación cuando las credenciales son incorrectas.

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
