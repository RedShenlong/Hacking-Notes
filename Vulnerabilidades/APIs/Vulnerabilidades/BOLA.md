# BOLA — Broken Object Level Authorization (API1)

**BOLA** (conocido en web clásica como **IDOR**) es la vulnerabilidad **número 1** del OWASP API Top 10 y la más común con diferencia. Ocurre cuando la API usa un identificador que tú controlas (un `id`, un `uuid`, un nombre) para devolver un objeto, pero **no comprueba que ese objeto sea tuyo**.

Resultado: cambias el identificador y accedes a datos (o acciones) de **otros usuarios**.

---

## Cómo se ve

Pides tu propio perfil:

```http
GET /api/v1/users/1337 HTTP/1.1
Authorization: Bearer <tu_token>
```

Cambias el `id` por otro:

```http
GET /api/v1/users/1338 HTTP/1.1
Authorization: Bearer <tu_token>      ← sigue siendo TU token
```

Si te devuelve los datos del usuario `1338`, es BOLA. El servidor comprobó que **estás autenticado**, pero no que ese recurso **te pertenezca**.

---

## Dónde buscarlo

Cualquier endpoint que reciba un identificador de objeto:

```text
GET    /api/orders/1001            → ¿puedo ver pedidos de otros?
GET    /api/users/42/invoices      → ¿facturas ajenas?
DELETE /api/posts/55               → ¿borro posts de otros?
PUT    /api/account/7/email        → ¿cambio el email de otra cuenta?
GET    /api/documents/9f8a...uuid  → ¿y con UUIDs?
```

Fíjate en identificadores que aparezcan **en la URL, en el cuerpo JSON, en parámetros o en cabeceras** (`X-User-Id`).

---

## Cómo probarlo

1. **Crea dos cuentas** (usuario A y usuario B) si puedes.
2. Con la sesión de **A**, haz una petición a un objeto de **A** y anota el `id`.
3. Repite la petición con el token de A pero el `id` de **B**. Si obtienes los datos de B → BOLA.
4. Prueba también **sin token** (a veces el endpoint ni pide autenticación).

### Fuerza bruta de identificadores

Si los IDs son numéricos y secuenciales, enuméralos en masa con Burp Intruder o `ffuf`:

```bash
ffuf -u http://<IP>/api/v1/users/FUZZ \
  -H "Authorization: Bearer <token>" \
  -w <(seq 1 5000) \
  -mc 200 -o bola.json
```

> Si los identificadores son **UUIDs aleatorios**, la fuerza bruta no sirve; busca formas de **filtrarlos** (que la API te los revele en otra respuesta, en listados, en mensajes de error).

### Automatizar con Autorize

La extensión **Autorize** de Burp repite cada petición que haces como admin con la sesión de un usuario sin privilegios y te marca dónde el acceso **no** se bloqueó. Ideal para barrer BOLA en toda la API de golpe.

---

## Variantes a no olvidar

- **Cambiar el método:** quizá `GET /api/users/1338` falle pero `DELETE` no esté protegido.
- **IDs en campos anidados** del JSON: `{"order":{"user_id":1338}}`.
- **Codificaciones:** prueba el ID en Base64, hex, o con "wrapping" (`1337` → `%31%33%33%37`).
- **Parameter pollution:** `?id=1337&id=1338`.

---

## Impacto

Desde fuga masiva de datos personales de todos los usuarios hasta modificación/borrado de recursos ajenos. Es la causa de muchísimas brechas reales de datos.

---

> **Consejo:** BOLA es lo **primero** que debes probar en cualquier API. Es trivial de comprobar (cambiar un número) y de los fallos más frecuentes. En DockerLabs/HTB con APIs, un `id` secuencial en la URL es casi siempre una invitación.

---

## Dónde practicar el BOLA

| Lab | Qué probar | Enlace |
| --- | ---------- | ------ |
| [VAmPI](./Laboratorios/VAmPI.md) | Leer el "libro secreto" de otro usuario por su id | https://github.com/erev0s/VAmPI |
| [OWASP crAPI](./Laboratorios/OWASP%20crAPI.md) | Ver el vehículo / perfil de otro usuario | https://github.com/OWASP/crAPI |
| [OWASP Juice Shop](./Laboratorios/OWASP%20Juice%20Shop.md) | Ver la cesta (basket) de otro usuario | https://github.com/juice-shop/juice-shop |
| PortSwigger — Access control / API testing | Labs gratis | https://portswigger.net/web-security/api-testing |

```flow
Capturar :: Interceptar una petición a un objeto tuyo (GET /users/v1/tuuser).
Cambiar :: Sustituir el identificador por el de otro usuario.
Confirmar :: Si recibes sus datos, es BOLA.
```

> **Consejo:** empieza por **VAmPI** (es el más simple) y luego crAPI para el escenario realista. El paso a paso completo está en sus notas de laboratorio.
