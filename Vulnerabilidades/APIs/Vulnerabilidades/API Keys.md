# API Keys y Secretos Expuestos (API8)

Las **API keys** son claves que identifican y autorizan a un cliente ante una API o servicio. Cuando se **filtran** (en el código fuente, en repositorios, en respuestas, en apps móviles) o se **gestionan mal**, un atacante puede suplantar al cliente legítimo, consumir servicios de pago o acceder a datos.

Se enmarca en **API8 — Security Misconfiguration**.

---

## Dónde se filtran las API keys

| Lugar | Cómo encontrarlo |
| ----- | ---------------- |
| **JavaScript del frontend** | Lee los `.js`; las keys suelen estar "hardcodeadas". |
| **Repositorios Git** | `.git` expuesto, historial del repo, `.env` subido por error. |
| **Apps móviles (APK/IPA)** | Descompilar la app y buscar cadenas. |
| **Respuestas de la API** | Ver [Excessive Data Exposure](./Excessive%20Data%20Exposure.md). |
| **Cabeceras / errores** | Debug activado, trazas verbosas. |
| **Documentación / Postman públicos** | Colecciones compartidas con tokens reales. |

### Buscar secretos en ficheros y código

```bash
# En el JS de una web
curl -s http://<IP>/main.js | grep -iE 'api[_-]?key|secret|token|bearer|aws|AKIA'

# En un repositorio local o descargado
grep -rniE '(api[_-]?key|secret|passwd|password|token|AKIA[0-9A-Z]{16})' .

# Herramientas dedicadas
trufflehog filesystem ./repo
gitleaks detect --source ./repo
```

### Patrones reconocibles

| Servicio | Patrón de la clave |
| -------- | ------------------ |
| AWS Access Key | `AKIA...` (20 caracteres) |
| Google API | `AIza...` |
| Stripe (live) | `sk_live_...` |
| Slack | `xox[baprs]-...` |
| GitHub token | `ghp_...` |
| JWT | `eyJ...` (ver [JWT](../04%20-%20JWT.md)) |

---

## Qué hacer con una API key encontrada

1. **Identifica a qué servicio pertenece** por su prefijo/formato.
2. **Comprueba su validez y alcance** sin causar daño:
   ```bash
   # Ejemplo: validar unas credenciales AWS filtradas
   aws sts get-caller-identity
   ```
3. **Enumera qué permite** (lectura de datos, acciones de pago, administración).

> Solo contra objetivos autorizados. Usar una clave filtrada contra un servicio real de un tercero es ilegal, aunque la clave "funcione".

---

## Malas prácticas que explotar

- **Keys sin expiración** ni rotación.
- **Keys con permisos excesivos** (una key de "solo lectura" que en realidad puede escribir).
- **Misma key para dev y producción**.
- **Falta de restricción por IP/dominio/referrer**.
- **Keys en la URL** (`?api_key=...`), que quedan en logs e historial.

---

## Impacto

Según la clave: acceso a datos, consumo de servicios de pago a costa de la víctima (facturas enormes en servicios cloud/IA), toma de control de infraestructura (claves AWS), o suplantación total del cliente legítimo.

---

> **Consejo:** cuando audites una SPA o una app móvil, lo primero es **descargar y peinar el JavaScript / el APK** en busca de secretos. Es uno de los hallazgos más rápidos y de mayor impacto. En CTFs, una API key en un fichero `.js` o en `.env` suele ser el foothold directo.

---

## Dónde practicar la búsqueda de API keys / secretos

| Lab | Qué probar | Enlace |
| --- | ---------- | ------ |
| [OWASP Juice Shop](./Laboratorios/OWASP%20Juice%20Shop.md) | Secretos en el JS/código y ficheros expuestos | https://github.com/juice-shop/juice-shop |
| trufflehog / gitleaks | Práctica sobre repos reales | https://github.com/trufflesecurity/trufflehog |

> **Consejo:** ante una SPA o app móvil, lo primero es peinar el **JavaScript** (o el APK) buscando `api_key`, `secret`, `AKIA`, `sk_live_`. Es uno de los hallazgos más rápidos y de mayor impacto.
