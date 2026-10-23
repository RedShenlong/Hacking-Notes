# Laboratorio: VAmPI

**VAmPI** (*Vulnerable API*) es una API REST deliberadamente vulnerable escrita en Python/Flask. Es ideal para **empezar** con hacking de APIs porque es pequeña, los fallos son claros y tiene un interruptor para activar/desactivar las vulnerabilidades (bueno para comparar el comportamiento seguro vs. vulnerable).

Repositorio: `erev0s/VAmPI`.

---

## Montaje

Con Docker (lo más rápido):

```bash
docker pull erev0s/vampi:latest
docker run -d -p 5000:5000 erev0s/vampi:latest
```

La API queda en `http://localhost:5000`. Comprueba que responde:

```bash
curl http://localhost:5000/
```

> VAmPI arranca por defecto en modo **vulnerable**. También expone su documentación OpenAPI, que te da el mapa de todos los endpoints.

---

## Endpoints principales

```text
GET  /                       → info y estado (vulnerable on/off)
POST /users/v1/register      → registro
POST /users/v1/login         → login (devuelve JWT)
GET  /users/v1               → lista de usuarios
GET  /users/v1/{username}    → datos de un usuario
DELETE /users/v1/{username}  → borrar usuario (¡mira la autorización!)
PUT  /users/v1/{username}/email
GET  /books/v1               → libros
POST /books/v1               → crear libro
GET  /books/v1/{title}       → detalle de libro (secreto por usuario)
```

---

## Vulnerabilidades a practicar

| Fallo | Dónde | Nota |
| ----- | ----- | ---- |
| **SQL Injection** | `/users/v1/login`, filtros | [SQL Injection](../SQL%20Injection.md) |
| **BOLA** | `/books/v1/{title}` revela secretos de otros usuarios | [BOLA](../BOLA.md) |
| **Broken Auth / JWT** | secreto débil, `alg:none` | [JWT](../../04%20-%20JWT.md) |
| **BFLA** | borrar/modificar otros usuarios | [BFLA](../BFLA.md) |
| **Mass Assignment** | registrarse con `"admin":true` | [Mass Assignment](../Mass%20Assignment.md) |
| **Excessive Data Exposure** | `/users/v1` devuelve contraseñas | [Excessive Data Exposure](../Excessive%20Data%20Exposure.md) |

---

## Recorrido sugerido

1. **Registro con Mass Assignment:** regístrate enviando `{"username":"x","password":"x","email":"x@x.com","admin":true}` y comprueba si quedas como admin.
2. **Excessive Data Exposure:** `GET /users/v1` → ¿te devuelve las contraseñas de todos?
3. **JWT:** haz login, decodifica el token y prueba a crackear el secreto o el ataque `none` ([JWT](../../04%20-%20JWT.md)).
4. **BOLA:** con tu token, lee el "libro secreto" de otro usuario cambiando el título.
5. **SQLi:** inyecta en el login (`admin' -- -`).

---

> **Consejo:** VAmPI es tu "hola mundo" de APIs. Haz cada ataque dos veces: una con el modo vulnerable ON y otra con las vulnerabilidades OFF, para ver **cómo cambia la respuesta** cuando el fallo está mitigado. Eso te entrena el ojo para detectarlos en objetivos reales.
