# Laboratorio: OWASP Juice Shop

**OWASP Juice Shop** es, probablemente, la aplicación vulnerable más famosa del mundo. Es una tienda online moderna (Angular + Node.js + API REST) con **más de 100 retos** que cubren el OWASP Top 10 web **y** muchos fallos de API. Incluye un **scoreboard** que registra tu progreso, lo que la hace muy adictiva para aprender.

Repositorio: `juice-shop/juice-shop`.

---

## Montaje

Con Docker:

```bash
docker pull bkimminich/juice-shop
docker run -d -p 3000:3000 bkimminich/juice-shop
```

Abre `http://localhost:3000`.

El **scoreboard** (la lista de retos) está oculto a propósito: encontrarlo es el primer reto. Pista: prueba `http://localhost:3000/#/score-board`.

---

## Categorías de retos

Juice Shop organiza los retos por dificultad (1 a 6 estrellas) y categoría:

| Categoría | Ejemplos | Nota relacionada |
| --------- | -------- | ---------------- |
| Injection | SQLi en el login, NoSQLi | [SQL Injection](../SQL%20Injection.md) |
| Broken Authentication | JWT, reset de contraseña | [JWT](../../04%20-%20JWT.md) |
| Broken Access Control | ver cestas/pedidos de otros | [BOLA](../BOLA.md), [BFLA](../BFLA.md) |
| Sensitive Data Exposure | ficheros expuestos, backups | [Excessive Data Exposure](../Excessive%20Data%20Exposure.md) |
| XSS | reflejado, almacenado, DOM | (ver `Vulnerabilidades/Web/XSS`) |
| XXE | subida de ficheros XML | [XXE](../XXE.md) |
| Improper Input Validation | Mass Assignment | [Mass Assignment](../Mass%20Assignment.md) |

---

## Retos clave para empezar (API y acceso)

1. **Login Admin (SQLi):** en el email del login prueba `' OR 1=1 -- -`.
2. **Ver la cesta de otro usuario (BOLA):** intercepta `GET /rest/basket/{id}` y cambia el `id`.
3. **JWT:** examina el token tras loguearte; hay retos de firma/algoritmo.
4. **Scoreboard:** accede a `/#/score-board`.
5. **Ficheros expuestos:** revisa `/ftp` — hay backups y documentos accesibles.

---

## Ayuda y seguimiento

- El libro oficial gratuito **"Pwning OWASP Juice Shop"** documenta todos los retos (úsalo solo cuando te atasques).
- Dentro de la app, el menú de ayuda da pistas progresivas por reto.

---

> **Consejo:** usa Juice Shop para entrenar el reflejo de **interceptar y leer la API** detrás de una SPA moderna. Aunque la interfaz sea bonita, todas las acciones pasan por `/rest/` y `/api/`: abre Burp, mira esas llamadas y ahí están los fallos. Es exactamente la mentalidad que necesitas para auditar aplicaciones reales.
