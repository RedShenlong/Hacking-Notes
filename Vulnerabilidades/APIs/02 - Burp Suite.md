# Burp Suite para APIs

**Burp Suite** es el proxy de interceptación por excelencia para auditar aplicaciones web y APIs. Se sitúa entre tu navegador/cliente y el servidor, permitiéndote **ver, detener y modificar** cada petición. Existe una edición **Community** gratuita (suficiente para aprender) y una **Professional** de pago (con escáner y sin límites en Intruder).

---

## Configuración inicial

1. Arranca Burp y ve a la pestaña **Proxy → Proxy settings**. Por defecto escucha en `127.0.0.1:8080`.
2. Configura tu navegador para usar ese proxy. Lo más cómodo es usar el navegador integrado de Burp (**Proxy → Intercept → Open Browser**), que ya viene preconfigurado.
3. Para interceptar **HTTPS**, instala el certificado CA de Burp: visita `http://burp` en el navegador integrado → *CA Certificate* → impórtalo como entidad de confianza.

---

## Las pestañas que de verdad vas a usar

### Proxy → HTTP history

El registro de **todo** el tráfico que ha pasado por Burp. Aquí mapeas la API: filtra por el host objetivo y verás cada endpoint que la aplicación llama.

### Repeater (tu mejor amiga)

Envía una misma petición una y otra vez, modificándola. Es donde probarás manualmente los fallos de API.

- Clic derecho sobre una petición → **Send to Repeater** (`Ctrl+R`).
- Modifica lo que quieras (un `id`, un campo JSON, una cabecera) y pulsa **Send**.
- Compara respuestas lado a lado.

> Flujo típico para **BOLA/IDOR**: capturas `GET /api/users/1337`, la mandas a Repeater, cambias `1337` por `1338` y miras si te devuelve datos de otro usuario.

### Intruder (automatizar peticiones)

Para probar muchos valores en una posición concreta (fuerza bruta de IDs, de credenciales, fuzzing de parámetros).

1. Envía la petición a Intruder (`Ctrl+I`).
2. Marca la posición a variar con `§...§`.
3. En **Payloads**, carga tu lista (números, diccionario, etc.).
4. Lanza el ataque y ordena por **longitud de respuesta** o **código de estado** para detectar las respuestas distintas.

> En la edición Community, Intruder está **limitado en velocidad**. Para ataques grandes, usa `ffuf` por línea de comandos.

### Decoder / Inspector

Codifica y decodifica Base64, URL, hex… Imprescindible para inspeccionar tokens **JWT** (ver [04 - JWT](./04%20-%20JWT.md)) y datos ofuscados.

---

## Extensiones útiles (BApp Store)

| Extensión | Para qué |
| --------- | -------- |
| **InQL** | Auditoría de GraphQL (introspección + ataques). |
| **JWT Editor** | Editar y firmar tokens JWT dentro de Burp. |
| **Autorize** | Detecta automáticamente fallos de autorización (BOLA/BFLA) repitiendo peticiones con otra sesión. |
| **Logger++** | Registro avanzado y búsqueda en todo el tráfico. |
| **Param Miner** | Descubre parámetros y cabeceras ocultas. |

> **Autorize** es oro puro para APIs: configúralo con la sesión de un usuario con **menos** privilegios y navega como admin; te marca en rojo cada endpoint al que el usuario de bajo privilegio **también** podía acceder.

---

## Flujo de trabajo recomendado

1. Navega la aplicación con el proxy activo para **poblar** el HTTP history.
2. Identifica los endpoints interesantes (los que manejan datos de usuario, acciones sensibles).
3. Mándalos a **Repeater** y pruébalos uno a uno contra el OWASP API Top 10.
4. Usa **Autorize** para barrer fallos de autorización de forma masiva.
5. Para fuerza bruta o fuzzing intensivo, salta a `ffuf`/`Intruder`.

---

> **Consejo:** guarda tu proyecto de Burp (`.burp`) por cada máquina/objetivo. Tener el HTTP history y tus pruebas de Repeater guardadas te ahorra rehacer trabajo cuando retomes la auditoría al día siguiente.
