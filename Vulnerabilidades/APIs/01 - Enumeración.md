# Enumeración de APIs

Antes de atacar una API necesitas **descubrir sus endpoints** y entender cómo se usan. A diferencia de una web clásica, los endpoints de una API no suelen estar enlazados en ninguna página: hay que descubrirlos.

---

## Paso 1 — Encontrar la API

- **Intercepta el tráfico** de la app web/móvil con [Burp Suite](./02%20-%20Burp%20Suite.md). Cada llamada que haga la aplicación te revela un endpoint real.
- Busca rutas típicas: `/api`, `/api/v1`, `/api/v2`, `/rest`, `/graphql`, `/swagger`, `/openapi.json`.
- Revisa el **JavaScript** de la web: a menudo contiene la lista de endpoints codificada.

```bash
# Extraer posibles rutas de los ficheros JS
curl -s http://<IP>/main.js | grep -oE '"/api/[a-zA-Z0-9/_-]+"'
```

---

## Paso 2 — Buscar documentación expuesta

Muchas APIs exponen su propia documentación, que es un **mapa completo** de todos los endpoints y parámetros. Pruébalos siempre:

```text
/swagger-ui.html
/swagger/index.html
/api/swagger.json
/openapi.json
/api-docs
/graphql        (con introspección activada)
/redoc
```

> Si encuentras un Swagger/OpenAPI, tienes el trabajo hecho a medias: ahí está cada ruta, método, parámetro y modelo de datos.

---

## Paso 3 — Fuzzing de endpoints

Si no hay documentación, descubre rutas por fuerza bruta con diccionarios específicos de APIs:

```bash
ffuf -u http://<IP>/api/v1/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
  -mc 200,201,401,403,405
```

> **Truco clave:** incluye `401` y `403` entre los códigos válidos (`-mc`). Un `403 Forbidden` confirma que el endpoint **existe** aunque no tengas acceso todavía — es exactamente lo que buscas para probar fallos de autorización.

Fuzzing de versiones y de métodos:

```bash
# ¿Existe una versión antigua y más vulnerable?
ffuf -u http://<IP>/api/FUZZ/users -w versiones.txt   # v1, v2, v3, beta, internal...

# ¿Qué métodos acepta un endpoint?
curl -s -X OPTIONS http://<IP>/api/v1/users -i | grep -i allow
```

---

## Paso 4 — GraphQL: introspección

Si la API es GraphQL, intenta la **introspection query**: devuelve el esquema completo (todos los tipos, consultas y mutaciones).

```bash
curl -s -X POST http://<IP>/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{__schema{types{name fields{name}}}}"}'
```

Herramientas: **InQL** (extensión de Burp) y **GraphQL Voyager** para visualizar el esquema.

---

## Paso 5 — Entender cada endpoint

Por cada endpoint descubierto, anota:

- Método(s) HTTP que acepta.
- Parámetros (en la URL, en el cuerpo JSON, en cabeceras).
- Si requiere autenticación y de qué tipo.
- Qué devuelve y con qué código de estado.

Esta "radiografía" es lo que luego atacarás endpoint por endpoint.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `Burp Suite` | Interceptar y mapear todo el tráfico de la API. |
| `ffuf` / `gobuster` | Fuzzing de endpoints y versiones. |
| `kiterunner` | Fuzzing específico de APIs (entiende rutas y métodos). |
| `InQL` | Introspección y ataque de GraphQL desde Burp. |
| `arjun` | Descubrimiento de parámetros ocultos. |

---

> **Consejo:** no te limites a `/api/v1`. Las versiones antiguas (`v1`, `beta`, `internal`, `legacy`) suelen seguir activas y sin parchear mientras el frontend ya usa `v2`. Es el fallo **API9 — Improper Inventory Management**, y es una mina de oro.
