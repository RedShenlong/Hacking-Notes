# Excessive Data Exposure (API3)

**Excessive Data Exposure** ocurre cuando la API devuelve **más datos de los que la aplicación muestra o de los que deberías ver**. El backend entrega el objeto entero y confía en que el frontend "ya filtrará" lo que enseña al usuario. Pero tú lees la respuesta cruda, no la pantalla.

Forma parte de **API3 — Broken Object Property Level Authorization** (la cara "de lectura"; la cara "de escritura" es [Mass Assignment](./Mass%20Assignment.md)).

---

## Cómo se ve

La web solo muestra el nombre y la foto del usuario, pero la API responde con todo:

```http
GET /api/v1/users/1337 HTTP/1.1
```

```json
{
  "id": 1337,
  "username": "diego",
  "full_name": "Diego ...",
  "email": "diego@ejemplo.com",
  "password_hash": "$2y$10$...",      ← ¡no debería estar aquí!
  "mfa_secret": "JBSWY3DPEHPK3PXP",    ← ¡ni esto!
  "credit_card_last4": "4242",
  "is_admin": false,
  "api_token": "sk_live_..."
}
```

El frontend ignora los campos sensibles, pero la respuesta los contiene. Tú los lees directamente.

---

## Dónde buscarlo

- **Endpoints de perfil / listados de usuarios**: hashes de contraseña, tokens, secretos MFA, roles.
- **Objetos de negocio**: campos internos como `internal_notes`, `cost`, `margin`, `is_flagged`.
- **Respuestas de listados** (`GET /api/users`): a veces exponen datos de **todos** los usuarios de golpe.
- **Errores y debug**: trazas, rutas internas, versiones.

---

## Cómo probarlo

1. **Lee siempre la respuesta JSON completa** en Burp, no te fíes de lo que pinta la web.
2. Busca campos sensibles: `password`, `hash`, `token`, `secret`, `key`, `ssn`, `card`, `is_admin`, `role`.
3. Prueba endpoints de **listado** sin filtros: `GET /api/users`, `GET /api/orders` → ¿devuelve datos de otros?
4. Combínalo con [BOLA](./BOLA.md): si además puedes cambiar el `id`, extraes los datos sensibles de **cualquier** usuario.

---

## Relación con otros fallos

| Combinado con | Resultado |
| ------------- | --------- |
| [BOLA](./BOLA.md) | Extraer datos sensibles de todos los usuarios iterando el `id`. |
| [Rate Limiting](./Rate%20Limiting.md) inexistente | Volcar la base de datos entera sin que te frenen. |
| JWT / sesiones | Un `api_token` expuesto te deja suplantar a ese usuario. |

---

## Impacto

Fuga de datos personales (RGPD), credenciales, tokens y secretos. Si la respuesta incluye un `password_hash`, te lo llevas a crackear offline; si incluye un `api_token` o `mfa_secret`, suplantas directamente al usuario.

---

> **Consejo:** este fallo es "gratis": no requiere manipular nada, solo **leer con atención** lo que la API ya te está dando. Acostúmbrate a revisar cada respuesta JSON campo por campo; es donde aparecen hashes y tokens que luego usarás para escalar.

---

## Dónde practicar la Excessive Data Exposure

| Lab | Qué probar | Enlace |
| --- | ---------- | ------ |
| [VAmPI](./Laboratorios/VAmPI.md) | `GET /users/v1` devuelve las contraseñas de todos | https://github.com/erev0s/VAmPI |
| [OWASP crAPI](./Laboratorios/OWASP%20crAPI.md) | Respuestas con datos internos del vehículo/perfil | https://github.com/OWASP/crAPI |

> **Consejo:** lee SIEMPRE la respuesta JSON completa en Burp (no lo que pinta la web). Busca `password`, `hash`, `token`, `secret`, `role`. Combínalo con BOLA para volcar datos de todos los usuarios.
