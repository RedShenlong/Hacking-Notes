# Laboratorio: DVGA (Damn Vulnerable GraphQL Application)

**DVGA** es una aplicación **GraphQL** deliberadamente vulnerable. Es el laboratorio de referencia para aprender a atacar APIs GraphQL, que funcionan de forma distinta a las REST: un **único endpoint** (`/graphql`) donde el cliente pide exactamente los datos que quiere mediante *queries* y *mutations*.

Repositorio: `dolevf/Damn-Vulnerable-GraphQL-Application`.

---

## Montaje

```bash
docker pull dolevf/dvga
docker run -d -p 5013:5013 -e WEB_HOST=0.0.0.0 dolevf/dvga
```

Abre `http://localhost:5013`. Trae una consola **GraphiQL** para lanzar consultas cómodamente.

> DVGA tiene niveles de dificultad (Easy/Hard) que cambian las protecciones. Empieza en Easy.

---

## GraphQL en 2 minutos

- Un solo endpoint: `POST /graphql`.
- **Query** = leer datos. **Mutation** = modificar datos.
- El cliente especifica los campos exactos que quiere.

```bash
curl -s -X POST http://localhost:5013/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ pastes { title content public } }"}'
```

---

## Introspección (el primer paso siempre)

Si la introspección está activa, obtienes el **esquema completo**: todos los tipos, queries y mutations disponibles. Es el mapa de ataque.

```bash
curl -s -X POST http://localhost:5013/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{__schema{queryType{name} types{name fields{name}}}}"}'
```

Herramientas: la extensión **InQL** de Burp visualiza el esquema y genera las consultas por ti.

---

## Vulnerabilidades a practicar

| Fallo | Descripción |
| ----- | ----------- |
| **Introspección expuesta** | El esquema entero es público → mapa de ataque regalado. |
| **Inyección (SQLi/OSi)** | Argumentos de queries/mutations no saneados → [SQL Injection](../SQL%20Injection.md), command injection. |
| **IDOR / BOLA** | Acceder a pastes privados de otros por `id` → [BOLA](../BOLA.md). |
| **Falta de rate limiting / DoS** | Consultas anidadas profundas que agotan recursos → [Rate Limiting](../Rate%20Limiting.md). |
| **Batching de queries** | Enviar muchas operaciones en una petición para saltarse límites (fuerza bruta). |
| **Divulgación de información** | Errores verbosos, campos ocultos. |

---

## Recorrido sugerido

1. **Introspección**: saca el esquema y anota todas las queries/mutations.
2. **Lee todos los pastes** (incluidos los `private`) → busca IDOR/BOLA.
3. **Inyección**: prueba payloads en los argumentos (p. ej. un `filter` o un `id`).
4. **Command injection**: DVGA tiene funciones que ejecutan comandos del sistema; busca RCE.
5. **DoS por anidamiento**: construye una query recursiva profunda.

---

> **Consejo:** en GraphQL, lo primero es **siempre** intentar la introspección. Si está activa, tienes el esquema completo y el ataque se vuelve metódico. Si está desactivada, aún puedes adivinar campos por fuerza bruta o fugas en mensajes de error. DVGA es el sitio perfecto para interiorizar esta rutina antes de ver GraphQL en un objetivo real.
