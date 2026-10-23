# Laboratorio: OWASP crAPI

**crAPI** (*completely ridiculous API*) es el laboratorio oficial de **OWASP** para aprender seguridad de APIs. Simula la plataforma de un taller de coches (usuarios, vehículos, mecánicos, tienda, chat) y está diseñado para cubrir **todo el OWASP API Security Top 10** en un escenario realista con frontend, API y base de datos.

Repositorio: `OWASP/crAPI`.

---

## Montaje

crAPI se levanta con Docker Compose (trae varios servicios: web, API, mailhog, base de datos):

```bash
curl -o docker-compose.yml https://raw.githubusercontent.com/OWASP/crAPI/main/deploy/docker/docker-compose.yml
docker compose -f docker-compose.yml pull
docker compose -f docker-compose.yml up -d
```

- Frontend web: `http://localhost:8888`
- **MailHog** (buzón de correo falso, clave para el reto): `http://localhost:8025`

> crAPI pide verificación por email al registrarte: los correos llegan a **MailHog**, no a un correo real. Ábrelo para ver los enlaces/OTP.

---

## Retos por categoría (OWASP API Top 10)

| Reto | Fallo | Pista |
| ---- | ----- | ----- |
| Ver el vehículo de otro usuario | [BOLA](../BOLA.md) | Identificadores de vehículo. |
| Acceder a informes de otros mecánicos | [BFLA](../BFLA.md) | Endpoints de mecánico. |
| Robar cupones / saldo | Mass Assignment / lógica | [Mass Assignment](../Mass%20Assignment.md) |
| Leer datos sensibles | [Excessive Data Exposure](../Excessive%20Data%20Exposure.md) | Respuestas del perfil/vehículo. |
| Resetear contraseña por fuerza bruta de OTP | [Rate Limiting](../Rate%20Limiting.md) | OTP sin límite de intentos. |
| SSRF en la subida de mecánico | [SSRF](../SSRF.md) | Campo que acepta URL. |
| JWT mal validado | [JWT](../../04%20-%20JWT.md) | Firma/algoritmo. |
| NoSQL / SQL injection | [SQL Injection](../SQL%20Injection.md) | Filtros y login. |

---

## Recorrido sugerido

1. **Regístrate** y confirma el correo vía MailHog (`:8025`).
2. **Intercepta** todo el tráfico con [Burp](../../02%20-%20Burp%20Suite.md) mientras navegas la plataforma.
3. **Añade un vehículo**: observa el flujo y busca el identificador → intenta ver el coche de otro ([BOLA](../BOLA.md)).
4. **Reset de contraseña**: localiza el endpoint del OTP y comprueba el rate limiting ([Rate Limiting](../Rate%20Limiting.md)).
5. **Tienda / cupones**: prueba Mass Assignment y lógica de negocio para conseguir saldo gratis.
6. **Mecánico**: busca el SSRF en el campo que recibe una URL.

---

> **Consejo:** crAPI es el laboratorio más completo y realista para APIs; si vas en serio con el hacking de APIs, resuélvelo entero. Lleva la cuenta de qué punto del OWASP API Top 10 cubre cada reto: te servirá como repaso estructurado antes de una auditoría real.
