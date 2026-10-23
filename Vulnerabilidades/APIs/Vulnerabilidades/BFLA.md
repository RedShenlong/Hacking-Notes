# BFLA — Broken Function Level Authorization (API5)

**BFLA** ocurre cuando puedes invocar **funciones que no te corresponden por tu rol**. Mientras [BOLA](./BOLA.md) es acceder a *objetos* de otros (sus datos), BFLA es acceder a *funciones* de otros (acciones de administrador, operaciones privilegiadas).

Dicho en corto: **un usuario normal ejecutando acciones de admin**.

---

## BOLA vs BFLA

| | BOLA | BFLA |
| --- | ---- | ---- |
| Qué rompes | Acceso a **datos** de otro usuario | Acceso a **funciones** fuera de tu rol |
| Ejemplo | Ver la factura `1338` de otro | Usar `POST /api/admin/users` siendo user |
| Clave | Cambias un **identificador** | Cambias el **endpoint/método/rol** |

---

## Cómo se ve

La app solo muestra el panel de administración a los admins, pero la API no lo comprueba en el backend:

```http
# Como usuario normal, con tu token de usuario normal:
POST /api/admin/users HTTP/1.1
Authorization: Bearer <token_de_usuario_normal>
Content-Type: application/json

{"username":"hacker","role":"admin"}
```

Si lo crea → BFLA. El frontend te ocultaba el botón, pero el endpoint estaba sin proteger.

---

## Dónde buscarlo

- **Endpoints de administración:** `/api/admin/*`, `/api/v1/management/*`, `/internal/*`.
- **Acciones privilegiadas:** crear/borrar usuarios, cambiar roles, ver logs, aprobar pagos.
- **Métodos HTTP alternativos:** un `GET` quizá esté permitido pero `PUT`/`DELETE` no se validan.

Para descubrir endpoints de admin ocultos, haz [enumeración](../01%20-%20Enumeraci%C3%B3n.md) con diccionarios que incluyan `admin`, `manage`, `internal`, etc.

---

## Cómo probarlo

1. **Usa dos cuentas:** una de admin (si la tienes) y una de usuario normal.
2. Captura con Burp las peticiones que hace el **admin** al operar.
3. **Repite esas mismas peticiones con el token del usuario normal.** Si funcionan → BFLA.
4. Si no tienes cuenta de admin, intenta **adivinar** los endpoints de administración y llámalos con tu token de usuario.

### Trucos de bypass frecuentes

| Técnica | Ejemplo |
| ------- | ------- |
| Cambiar el método | `GET /api/admin/users` prohibido, pero `POST` no se valida. |
| Variación de ruta | `/api/admin/users` vs `/api/Admin/users`, `/api/v1/../admin/users`. |
| Cabecera de rol falsa | Añadir `X-Role: admin` o `X-Original-URL: /admin`. |
| Verbo HTTP override | Cabecera `X-HTTP-Method-Override: DELETE`. |
| Versión antigua | `/api/v1/admin/...` sin protección aunque `v2` sí la tenga. |

---

## Impacto

Toma de control total: crear usuarios administradores, borrar datos, modificar configuraciones, acceder a toda la información de la plataforma.

---

> **Consejo:** combina BFLA con la [enumeración de endpoints](../01%20-%20Enumeraci%C3%B3n.md). Muchas veces el endpoint de admin **existe** (devuelve `403` o `401`) pero una pequeña variación de ruta, método o cabecera lo deja pasar. Un `403` no es un "no": es un "existe, sigue intentando".

---

## Dónde practicar el BFLA

| Lab | Qué probar | Enlace |
| --- | ---------- | ------ |
| [OWASP crAPI](./Laboratorios/OWASP%20crAPI.md) | Acceder a informes/funciones de mecánico sin serlo | https://github.com/OWASP/crAPI |
| [VAmPI](./Laboratorios/VAmPI.md) | Borrar/modificar a otros usuarios siendo normal | https://github.com/erev0s/VAmPI |
| PortSwigger — API testing | Labs gratis | https://portswigger.net/web-security/api-testing |

> **Consejo:** usa la extensión **Autorize** de Burp con la sesión de un usuario de bajo privilegio para detectar automáticamente qué funciones de admin te deja invocar.
