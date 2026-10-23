# Falta de Rate Limiting (API4)

**Rate limiting** es el control que limita cuántas peticiones puede hacer un cliente en un tiempo dado. Cuando **no existe o es débil**, puedes abusar de la API con peticiones masivas: fuerza bruta de credenciales, de códigos OTP, enumeración de usuarios o simplemente tumbar el servicio.

Corresponde a **API4 — Unrestricted Resource Consumption**.

---

## Qué permite la ausencia de rate limiting

| Ataque | Por qué es posible sin límite |
| ------ | ----------------------------- |
| **Fuerza bruta de login** | Probar miles de contraseñas sobre un usuario. |
| **Bypass de OTP/2FA** | Un código de 4-6 dígitos es forzable (10.000–1.000.000 intentos). |
| **Enumeración de usuarios** | Distinguir usuarios válidos por la respuesta, en masa. |
| **Password reset abuse** | Pedir/forzar tokens de recuperación. |
| **Denegación de servicio (DoS)** | Saturar endpoints costosos. |
| **Scraping / volcado** | Extraer toda la base de datos vía [BOLA](./BOLA.md). |

---

## Cómo probarlo

Lanza muchas peticiones y observa si en algún momento aparece un `429 Too Many Requests` o un bloqueo:

```bash
# 500 peticiones; si todas responden 200/401 y ninguna 429, no hay rate limit
for i in $(seq 1 500); do
  curl -s -o /dev/null -w "%{http_code}\n" \
    -X POST http://<IP>/api/v1/login \
    -H "Content-Type: application/json" \
    -d '{"user":"admin","password":"wrong'"$i"'"}'
done | sort | uniq -c
```

Con **Burp Intruder** o **ffuf**, observa la columna de código de estado: si nunca aparece `429`, el endpoint es forzable.

### Ejemplo: forzar un OTP de 6 dígitos

```bash
ffuf -u http://<IP>/api/v1/verify-otp \
  -X POST -H "Content-Type: application/json" \
  -d '{"otp":"FUZZ"}' \
  -w <(seq -w 0 999999) \
  -mc 200 -t 50
```

---

## Trucos para saltar un rate limit débil

A veces el límite existe pero está mal implementado:

| Técnica | Cómo |
| ------- | ---- |
| Rotar la IP de origen | Cabeceras `X-Forwarded-For`, `X-Real-IP`, `X-Originating-IP` con valores distintos en cada petición. |
| Variar el caso o la ruta | `/API/v1/login`, añadir `/` final, parámetros basura. |
| Cambiar el identificador | Si limita por usuario, prueba a repartir entre varios. |
| Peticiones concurrentes | Enviar en paralelo antes de que el contador se actualice (relacionado con *race conditions*). |

```http
POST /api/v1/login HTTP/1.1
X-Forwarded-For: 1.2.3.4      ← cambia este valor en cada intento
```

---

## Impacto

Habilita todos los demás ataques de fuerza bruta. Sin rate limiting, un OTP o una contraseña débil dejan de ser una barrera. También permite DoS y scraping masivo de datos.

---

> **Consejo:** antes de lanzar fuerza bruta contra un login o un OTP, haz una prueba rápida de 50-100 peticiones para confirmar que **no** hay rate limiting. Si lo hay, prueba primero los bypass por cabecera `X-Forwarded-For` — es sorprendentemente frecuente que el límite se aplique por IP y confíe en esa cabecera.

---

## Dónde practicar la falta de Rate Limiting

| Lab | Qué probar | Enlace |
| --- | ---------- | ------ |
| [OWASP crAPI](./Laboratorios/OWASP%20crAPI.md) | Fuerza bruta del OTP de recuperación de contraseña | https://github.com/OWASP/crAPI |
| PortSwigger — Authentication | Fuerza bruta y bypass | https://portswigger.net/web-security/authentication |

> **Consejo:** antes de forzar un OTP/login, lanza 50-100 peticiones para confirmar que no hay límite. Si lo hay, prueba el bypass con cabeceras `X-Forwarded-For` distintas en cada intento.
