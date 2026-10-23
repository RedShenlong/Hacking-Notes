# Burp Suite (guía completa)

**Burp Suite** es el proxy de interceptación estándar para auditar aplicaciones web. Esta guía cubre el flujo completo para CTFs y pentest web; para el enfoque específico de APIs, ver [APIs · Burp Suite](../Vulnerabilidades/APIs/02%20-%20Burp%20Suite.md).

---

## Puesta en marcha

1. **Proxy → Intercept → Open Browser** (navegador preconfigurado, lo más fácil).
2. Para tu navegador normal: proxy `127.0.0.1:8080` + instala el certificado CA visitando `http://burp`.
3. Define el **Target scope** (`Target → Scope → Add`) y marca *"show only in-scope items"* para no ahogarte en ruido.

> **Consejo:** define el scope lo primero. Con `http://<IP>` en scope y el filtro activado, el HTTP history solo muestra lo relevante. Sin esto, el tráfico de Google/extensiones te sepulta.

---

## Las pestañas, por orden de uso

### Proxy → HTTP history
El registro de todo el tráfico. Aquí **mapeas** la app: navégala entera con el proxy activo para poblarlo, luego revisa cada endpoint.

### Repeater (`Ctrl+R`) — tu herramienta principal
Reenvía una petición modificándola. Donde pruebas manualmente SQLi, IDOR, LFI, etc.
- Envía desde HTTP history con `Ctrl+R`.
- Modifica y `Ctrl+Space` para enviar.
- Usa pestañas distintas para comparar respuestas.

### Intruder (`Ctrl+I`) — automatizar
Prueba muchos valores en posiciones marcadas con `§`.
- **Sniper**: un payload en una posición (fuzzing de un parámetro, fuerza bruta de PIN).
- **Cluster bomb**: combinaciones de varias posiciones (user × pass).
- Ordena por **status** o **length** para ver la respuesta distinta.
> En Community, Intruder va **ralentizado**; para ataques grandes usa `ffuf`.

### Decoder / Inspector
Codifica/decodifica Base64, URL, hex, HTML. Imprescindible para JWT y datos ofuscados.

### Comparer
Compara dos respuestas byte a byte (útil para ver qué cambia entre un intento válido y uno inválido).

### Sequencer
Analiza la aleatoriedad de tokens de sesión (¿predecibles?).

---

## Extensiones imprescindibles (BApp Store)

| Extensión | Para qué |
| --------- | -------- |
| **Autorize** | Detecta IDOR/BOLA/BFLA automáticamente usando dos sesiones. |
| **JWT Editor** | Editar y re-firmar tokens JWT. |
| **Param Miner** | Descubre parámetros y cabeceras ocultas. |
| **Logger++** | Registro y búsqueda avanzada en todo el tráfico. |
| **Turbo Intruder** | Ataques masivos y race conditions (muy rápido, con script). |
| **InQL** | Auditoría de GraphQL. |
| **Hackvertor** | Codificaciones/transformaciones inline con etiquetas. |

> **Consejo (poco conocido):** **Autorize** resuelve una categoría entera de fallos sola: configúralo con la cookie de un usuario de **bajo** privilegio y navega como admin; te marca en rojo cada acción que el usuario normal **también** podía hacer (control de acceso roto). Es de lo más rentable que tiene Burp.

---

## Flujos típicos en CTF web

```flow
Mapear :: Navega con el proxy y puebla el HTTP history (define scope).
Marcar :: Identifica endpoints con datos/acciones sensibles.
Probar :: Manda a Repeater y ataca cada clase de vuln (SQLi, IDOR, LFI, upload).
Automatizar :: Intruder/Autorize para fuerza bruta y control de acceso.
Escalar :: ffuf/Turbo Intruder para lo masivo; JWT Editor para tokens.
```

### Trucos concretos
- **Match and Replace** (`Proxy → Options`): reescribe cabeceras al vuelo (p.ej. forzar `X-Forwarded-For: 127.0.0.1` en todas las peticiones).
- **Interceptar la respuesta**: activa *"Intercept responses"* para modificar lo que ve el cliente (saltar validaciones del lado cliente, desbloquear botones).
- **Guardar el proyecto** (`.burp`) por máquina: no pierdes el history ni tus pruebas de Repeater.
- **Copy as curl** (clic derecho): lleva una petición a la terminal o a sqlmap (`sqlmap -r peticion.txt`).

---

## Burp + sqlmap / ffuf

Guarda la petición (`Save item` / *Copy to file*) y pásala a otras herramientas:

```bash
sqlmap -r peticion.txt --batch          # SQLi con la petición exacta (cookies incluidas)
ffuf -request peticion.txt -request-proto http -w wordlist.txt   # fuzzing con la petición completa
```

---

> **Consejo:** domina **Repeater** y **Autorize** antes que nada: cubren el 80 % del trabajo web en CTF (probar vulns a mano y cazar control de acceso roto). Reserva Intruder para lo pequeño y delega lo masivo en `ffuf`. Y acostúmbrate a **interceptar respuestas**: muchas "validaciones" de los labs están solo en el cliente y se saltan editando la respuesta.

---

## Dónde practicar con Burp

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Web Security Academy | Labs gratis (todos se hacen con Burp) | https://portswigger.net/web-security |
| OWASP Juice Shop | App vulnerable (Docker) | https://github.com/juice-shop/juice-shop |
| DVWA | App vulnerable (Docker) | https://github.com/digininja/DVWA |
| PortSwigger — "Burp Suite Certified Practitioner" (BSCP) | Certificación práctica | https://portswigger.net/web-security/certification |

> **Consejo:** la **Web Security Academy** está pensada para hacerse con Burp: cada lab te obliga a interceptar, modificar en Repeater y, a menudo, usar Intruder o el exploit server. Es, con diferencia, la mejor forma gratuita de dominar Burp. Si lo completas, el **BSCP** es tu siguiente paso natural.
