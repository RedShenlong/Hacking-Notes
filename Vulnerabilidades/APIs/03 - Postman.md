# Postman para APIs

**Postman** es un cliente de APIs pensado para **construir y organizar peticiones** de forma cómoda. Mientras Burp destaca interceptando tráfico, Postman brilla cuando trabajas con una API documentada (Swagger/OpenAPI) y quieres lanzar peticiones estructuradas, reutilizar tokens y encadenar llamadas.

---

## Conceptos básicos

| Concepto | Qué es |
| -------- | ------ |
| **Request** | Una petición HTTP (método + URL + cabeceras + cuerpo). |
| **Collection** | Un grupo de peticiones organizadas (p. ej. toda una API). |
| **Environment** | Conjunto de variables (`{{base_url}}`, `{{token}}`) que cambias sin tocar cada petición. |
| **Variables** | Valores reutilizables; evitan repetir la URL o el token en cada request. |

---

## Montar una petición

1. Elige el método (`GET`, `POST`…) y escribe la URL.
2. En **Authorization**, elige el tipo (Bearer Token, API Key, Basic Auth) y pega tu token.
3. En **Headers**, añade por ejemplo `Content-Type: application/json`.
4. En **Body → raw → JSON**, escribe el cuerpo de la petición.
5. **Send**.

```json
// Ejemplo de Body para crear un usuario
{
  "username": "test",
  "password": "Test1234!",
  "role": "admin"
}
```

> El campo `"role": "admin"` del ejemplo es justo el tipo de prueba de **Mass Assignment**: envías un campo extra a ver si el servidor lo acepta.

---

## Importar una API entera (Swagger/OpenAPI)

Si encontraste un `openapi.json` o `swagger.json` durante la [enumeración](./01%20-%20Enumeraci%C3%B3n.md), impórtalo directamente:

**Import → File/Link** → pega la URL del `openapi.json`.

Postman genera **automáticamente** una colección con **todos** los endpoints, sus parámetros y ejemplos. A partir de ahí solo tienes que ir probando cada uno.

---

## Variables de entorno (reutilizar el token)

Crea un *Environment* con variables para no repetir datos:

```text
base_url = http://<IP>/api/v1
token    = eyJhbGciOi...
```

Y úsalas en las peticiones:

```http
GET {{base_url}}/users/me
Authorization: Bearer {{token}}
```

Cuando el token caduque, lo cambias en un solo sitio.

> **Automatizar el login:** en la pestaña **Tests** de la petición de login puedes guardar el token de la respuesta en la variable automáticamente:
> ```javascript
> pm.environment.set("token", pm.response.json().access_token);
> ```
> Así, cada vez que hagas login, todas las demás peticiones quedan autenticadas solas.

---

## Postman + Burp (lo mejor de ambos)

Puedes enrutar el tráfico de Postman a través de Burp para registrar e inspeccionar todo:

**Settings → Proxy → Add custom proxy** → `127.0.0.1:8080`.

Desactiva la verificación de certificado SSL en Postman (**Settings → General → SSL certificate verification: OFF**) para que funcione con el certificado de Burp.

Así construyes peticiones cómodas en Postman y las analizas/repites en Burp.

---

## Postman vs. Burp: ¿cuándo cada uno?

| Situación | Herramienta |
| --------- | ----------- |
| Interceptar tráfico de una app web/móvil | **Burp** |
| Trabajar con una API documentada (Swagger) | **Postman** |
| Probar fallos de autorización masivamente | **Burp** (Autorize) |
| Encadenar login → acción → logout con variables | **Postman** |
| Fuzzing / fuerza bruta intensiva | `ffuf` / Burp Intruder |

---

> **Consejo:** usa Postman para **entender y organizar** la API (sobre todo si tienes su Swagger), y Burp para **atacarla** en serio. No son rivales: muchos pentesters construyen la petición en Postman y la mandan a Burp Repeater para manipularla.
