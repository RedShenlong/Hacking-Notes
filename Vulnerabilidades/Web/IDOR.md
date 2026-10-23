# IDOR (Insecure Direct Object Reference)

**IDOR** ocurre cuando una web usa un identificador que tú controlas (un `id`, un nombre de archivo, un número de pedido) para darte acceso a un objeto **sin comprobar que sea tuyo**. Cambias el identificador y accedes a datos o acciones de otros usuarios. Es la versión "web clásica" de [BOLA](../APIs/Vulnerabilidades/BOLA.md).

---

## Cómo se ve

```text
http://<IP>/perfil?id=1001          -> cambia a 1002
http://<IP>/factura/4567.pdf        -> prueba 4568.pdf
http://<IP>/mensajes?user=diego     -> prueba otro usuario
POST /api/cuenta  {"account_id": 7} -> cambia el 7
```

Si al cambiar el valor ves información **de otra persona**, es IDOR.

---

## Dónde buscarlo

Cualquier parámetro que referencie un objeto:

- En la **URL**: `?id=`, `?user=`, `?file=`, `?order=`, `?doc=`.
- En el **cuerpo** (JSON/form): `{"user_id": ...}`.
- En **cabeceras** o **cookies**: `X-User-Id: ...`.
- En nombres de **archivos** descargables (`/download?file=report_1001.pdf`).

---

## Cómo probarlo

1. Crea (si puedes) **dos cuentas**: usuario A y usuario B.
2. Con la sesión de **A**, accede a un objeto tuyo y anota su `id`.
3. Repite la petición con el `id` de **B** (manteniendo la sesión de A). Si funciona → IDOR.
4. Prueba también **sin sesión** por si el endpoint ni valida login.

### Enumeración masiva

Si los IDs son numéricos y secuenciales, extrae en masa con Burp Intruder o ffuf:

```bash
ffuf -u "http://<IP>/factura/FUZZ.pdf" \
  -b "session=<tu_cookie>" \
  -w <(seq 1000 2000) \
  -mc 200 -o idor.json
```

---

## Variantes y bypass

| Técnica | Idea |
| ------- | ---- |
| Cambiar el **método** | `GET` protegido pero `POST`/`PUT` no. |
| **IDs codificados** | Decodifica el valor (Base64, hex) antes de iterar: `MTAwMQ==` = 1001. |
| **Parameter pollution** | `?id=TU_ID&id=OTRO_ID`. |
| **IDs "predecibles"** | UUID v1 o secuencias temporales a veces son adivinables. |
| **Wrapping/nesting** | IDs dentro de JSON anidado: `{"data":{"id":1002}}`. |
| **Acciones, no solo lectura** | Cambiar el email/contraseña de otra cuenta por su `id`. |

---

## Impacto

Según el objeto: fuga de datos personales de todos los usuarios, descarga de facturas/documentos ajenos, modificación o borrado de recursos de otros, y hasta toma de cuentas si permite cambiar credenciales por `id`.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `Burp Suite` (Repeater/Intruder) | Cambiar IDs y enumerar en masa. |
| `Autorize` (Burp) | Detectar IDOR/BOLA automáticamente con dos sesiones. |
| `ffuf` | Iterar identificadores secuenciales. |

---

> **Consejo:** IDOR es de los fallos más fáciles de encontrar y más frecuentes: en cuanto veas un número en la URL o en una petición, **súmale o réstale 1**. Si los IDs no son secuenciales (UUID aleatorios), busca formas de **filtrarlos** (que la app te los muestre en listados, respuestas o mensajes de error) antes de rendirte.

---

## Dónde practicar el IDOR

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Academy — Access control (IDOR) | Labs gratis | https://portswigger.net/web-security/access-control/idor |
| OWASP Juice Shop | Descargable (Docker) | https://github.com/juice-shop/juice-shop |
| TryHackMe — "IDOR" | Sala guiada | https://tryhackme.com |
| VulnHub / DockerLabs | Máquinas completas | https://www.vulnhub.com · https://dockerlabs.es |

## Writeup guiado: IDOR en PortSwigger (lab de access control)

Lab: *"Insecure direct object references"* de la Web Security Academy (gratis).

```flow
Observar :: Una función de chat descarga tu transcripción por un fichero numerado.
Interceptar :: Capturar la petición de descarga con Burp.
Manipular :: Cambiar el número del fichero por otro.
Obtener :: Leer la transcripción de otro usuario (y su contraseña).
```

### Paso 1 — Observar

En el lab, la función **"View transcript"** descarga un archivo tipo `/download-transcript/2.txt`. El número es un identificador directo a un objeto.

### Paso 2 — Interceptar y manipular

Captura la petición con Burp, mándala a **Repeater** y cambia `2.txt` por `1.txt`:

```http
GET /download-transcript/1.txt HTTP/1.1
```

### Paso 3 — Obtener

La respuesta contiene la transcripción de **otro usuario**, que incluye su contraseña. Inicias sesión como él → lab resuelto.

> **Consejo:** este patrón (cambiar un número en una descarga/consulta) es idéntico en máquinas reales y en APIs ([BOLA](../APIs/Vulnerabilidades/BOLA.md)). Si los IDs no son secuenciales, usa **Autorize** de Burp con dos sesiones para detectar automáticamente a qué objetos ajenos puedes acceder.
