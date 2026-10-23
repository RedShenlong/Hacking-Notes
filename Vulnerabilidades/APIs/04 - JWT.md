# JWT (JSON Web Tokens)

Los **JWT** son el método de autenticación más habitual en APIs modernas. Un JWT es un token que el servidor entrega tras el login y que el cliente envía en cada petición (normalmente en la cabecera `Authorization: Bearer <token>`). Si el token está mal implementado, puedes **falsificar tu identidad** y convertirte en otro usuario o en administrador.

Corresponde a **API2 — Broken Authentication** del OWASP API Top 10.

---

## Anatomía de un JWT

Un JWT son tres partes codificadas en Base64URL separadas por puntos: `header.payload.signature`.

```text
eyJhbGciOiJIUzI1NiJ9 . eyJ1c2VyIjoiZGllZ28iLCJyb2xlIjoidXNlciJ9 . 4pcPy...
      HEADER                        PAYLOAD                         SIGNATURE
```

- **Header:** algoritmo de firma. `{"alg":"HS256","typ":"JWT"}`
- **Payload:** los datos (los *claims*). `{"user":"diego","role":"user"}`
- **Signature:** firma criptográfica que **garantiza que nadie ha modificado** header y payload.

Decodifícalo (las dos primeras partes son Base64, legibles; no están cifradas):

```bash
echo 'eyJ1c2VyIjoiZGllZ28iLCJyb2xlIjoidXNlciJ9' | base64 -d
```

> **Clave mental:** el payload **no es secreto**, cualquiera lo lee. Lo único que impide manipularlo es la **firma**. Todos los ataques a JWT consisten en **poder reescribir el payload y que la firma siga pareciendo válida**.

---

## Ataque 1 — Algoritmo `none`

Algunas librerías aceptan `"alg":"none"`, que significa "token sin firma". Si el servidor lo acepta, puedes poner el payload que quieras sin firmar nada.

1. Cambia el header a `{"alg":"none","typ":"JWT"}`.
2. Modifica el payload (p. ej. `"role":"admin"`).
3. Deja la firma **vacía** (el token termina en un punto).

```text
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJ1c2VyIjoiZGllZ28iLCJyb2xlIjoiYWRtaW4ifQ.
```

---

## Ataque 2 — Confusión de algoritmo (RS256 → HS256)

Si el servidor firma con **RS256** (clave privada/pública) pero no valida bien el algoritmo, puedes cambiarlo a **HS256** y firmar el token usando la **clave pública** (que es, como su nombre indica, pública) como si fuera el secreto HMAC. El servidor verificará con esa misma clave pública y aceptará tu token.

---

## Ataque 3 — Secreto débil (HS256)

Con HS256 la firma depende de un **secreto compartido**. Si ese secreto es débil (`secret`, `123456`, una palabra de diccionario), puedes crackearlo offline y entonces firmar **cualquier** token tú mismo.

```bash
# Crackear el secreto con hashcat (modo 16500 = JWT)
hashcat -a 0 -m 16500 token.jwt /usr/share/wordlists/rockyou.txt

# Alternativa clásica con jwt_tool
python3 jwt_tool.py <TOKEN> -C -d /usr/share/wordlists/rockyou.txt
```

Una vez tengas el secreto, generas un token nuevo con el payload que quieras y la firma será válida.

---

## Otros fallos a comprobar

| Fallo | Qué probar |
| ----- | ---------- |
| **Sin verificación de firma** | Cambia un carácter del payload sin tocar la firma. Si sigue funcionando, el servidor no valida nada. |
| **`kid` injection** | El header `kid` (key id) puede apuntar a un fichero o ser inyectable (path traversal, SQLi) para forzar la clave de verificación. |
| **`jku`/`x5u` controlable** | El header puede indicar una URL con la clave; si la controlas tú, firmas con tu propia clave. |
| **Expiración ignorada** | ¿El servidor respeta el claim `exp`? Prueba un token caducado. |
| **Token no invalidado** | ¿Un token sigue valiendo tras cerrar sesión? |

---

## jwt_tool (la navaja suiza)

```bash
# Analizar y lanzar todas las comprobaciones automáticas
python3 jwt_tool.py <TOKEN> -M at

# Manipular el payload (modo "tamper")
python3 jwt_tool.py <TOKEN> -T

# Forzar el ataque "none"
python3 jwt_tool.py <TOKEN> -X a
```

También puedes usar la web **jwt.io** para decodificar visualmente (no pegues tokens reales de producción) y la extensión **JWT Editor** de Burp.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `jwt_tool` | Análisis y explotación completa de JWT. |
| `hashcat` (modo 16500) | Crackear el secreto de HS256. |
| `JWT Editor` (Burp) | Editar y re-firmar tokens dentro de Burp. |
| `jwt.io` | Decodificador visual (solo tokens de prueba). |

---

> **Consejo:** lo primero que debes hacer con cualquier JWT es **decodificar el payload** y mirar los claims. Si ves algo como `"role":"user"`, `"isAdmin":false` o `"user_id":1000`, ya tienes tu objetivo: lograr que ese valor cambie y que la firma lo acepte.
