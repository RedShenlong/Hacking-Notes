# Introducción al Hacking de APIs

Las **APIs** (Application Programming Interfaces) son la columna vertebral de las aplicaciones modernas: webs, apps móviles y servicios se comunican a través de ellas. Donde antes había un formulario web, hoy hay una API detrás. Por eso son una de las superficies de ataque más grandes y, a menudo, peor protegidas.

---

## ¿Qué es una API (desde el punto de vista ofensivo)?

Una API expone **endpoints** (rutas) que aceptan peticiones y devuelven datos, normalmente en **JSON**. El tipo más común es la **API REST**, que usa métodos HTTP:

| Método HTTP | Acción habitual        |
| ----------- | ---------------------- |
| `GET`       | Leer / consultar       |
| `POST`      | Crear                  |
| `PUT`/`PATCH` | Modificar            |
| `DELETE`    | Borrar                 |

Ejemplo de petición y respuesta:

```http
GET /api/v1/users/1337 HTTP/1.1
Host: api.ejemplo.com
Authorization: Bearer eyJhbGciOi...
```

```json
{
  "id": 1337,
  "username": "diego",
  "email": "diego@ejemplo.com",
  "role": "user"
}
```

Otros estilos que encontrarás: **GraphQL** (un único endpoint, el cliente pide exactamente los campos que quiere) y **SOAP** (más antiguo, basado en XML).

---

## Por qué las APIs fallan tanto

El problema central: **el servidor confía en el cliente**. Muchas APIs asumen que las peticiones vendrán de "su" app oficial y se saltan validaciones del lado servidor. Pero tú puedes construir **cualquier** petición a mano. De ahí salen fallos como:

- Acceder a datos de otros usuarios cambiando un `id` (BOLA/IDOR).
- Llamar a funciones de administrador que la app "no te muestra" (BFLA).
- Enviar campos extra que el servidor acepta sin filtrar (Mass Assignment).

---

## OWASP API Security Top 10 (el mapa de lo que vas a buscar)

Es la lista de referencia de los fallos más críticos en APIs. Cada uno tiene su nota en [`Vulnerabilidades/`](./Vulnerabilidades/):

| ID    | Nombre | Nota |
| ----- | ------ | ---- |
| API1  | Broken Object Level Authorization | [BOLA](./Vulnerabilidades/BOLA.md) |
| API2  | Broken Authentication | [JWT](./04%20-%20JWT.md) |
| API3  | Broken Object Property Level Authorization | [Excessive Data Exposure](./Vulnerabilidades/Excessive%20Data%20Exposure.md) / [Mass Assignment](./Vulnerabilidades/Mass%20Assignment.md) |
| API4  | Unrestricted Resource Consumption | [Rate Limiting](./Vulnerabilidades/Rate%20Limiting.md) |
| API5  | Broken Function Level Authorization | [BFLA](./Vulnerabilidades/BFLA.md) |
| API6  | Unrestricted Access to Sensitive Business Flows | Lógica de negocio |
| API7  | Server Side Request Forgery | [SSRF](./Vulnerabilidades/SSRF.md) |
| API8  | Security Misconfiguration | [API Keys](./Vulnerabilidades/API%20Keys.md), [XXE](./Vulnerabilidades/XXE.md) |
| API9  | Improper Inventory Management | Versiones viejas (`/api/v1` vs `/api/v2`) |
| API10 | Unsafe Consumption of APIs | Confianza ciega en APIs de terceros |

---

## Flujo de trabajo para auditar una API

1. **Descubrir** la API y sus endpoints → [Enumeración](./01%20-%20Enumeraci%C3%B3n.md).
2. **Entender la autenticación**: ¿tokens JWT, API keys, cookies? → [JWT](./04%20-%20JWT.md), [API Keys](./Vulnerabilidades/API%20Keys.md).
3. **Interceptar y manipular** peticiones con [Burp Suite](./02%20-%20Burp%20Suite.md) o [Postman](./03%20-%20Postman.md).
4. **Probar cada endpoint** contra el OWASP API Top 10.
5. **Practicar** en laboratorios dedicados → [Laboratorios](./Vulnerabilidades/Laboratorios/).

---

> **Consejo:** cuando audites una app móvil o una SPA (web de una sola página), abre siempre las **herramientas de desarrollador del navegador** (pestaña *Network*) o intercepta el tráfico con Burp. Ahí verás todas las llamadas a la API real, que es donde está la acción.
