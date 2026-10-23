# Wireshark (análisis de tráfico de red)

**Wireshark** captura y analiza el tráfico de red paquete a paquete. En CTFs y pentest sirve para **leer protocolos sin cifrar** (credenciales en claro), analizar **capturas `.pcap`** que te dan como reto, entender un protocolo propietario y depurar tus propios ataques. Su versión de terminal es **tshark**.

---

## Captura básica

```bash
# GUI: elige la interfaz (eth0/tun0) y pulsa el aletón azul.
# Terminal (tshark):
tshark -i tun0 -w captura.pcap          # capturar a archivo
tshark -r captura.pcap                   # leer un pcap
sudo tcpdump -i tun0 -w captura.pcap     # alternativa ligera para capturar
```

> **Consejo:** en un lab, captura en tu interfaz del VPN (`tun0`) mientras lanzas el ataque: verás exactamente qué envías y qué responde la máquina, ideal para depurar exploits que "no funcionan".

---

## Filtros de visualización (los que de verdad usas)

Se escriben en la barra superior (filtros de *display*, distintos de los de captura):

```text
http                      # solo HTTP
http.request              # solo peticiones
dns                       # consultas DNS
ftp                       # FTP (usuario/clave en claro)
tcp.port == 445           # SMB
ip.addr == 10.10.10.5     # tráfico de/hacia una IP
tcp.flags.syn == 1 && tcp.flags.ack == 0   # intentos de conexión (escaneo)
frame contains "password" # cualquier paquete que contenga "password"
tcp.stream eq 3           # aislar una conversación TCP concreta
```

Operadores: `==`, `!=`, `&&`, `||`, `contains`, `matches` (regex).

---

## Extraer credenciales y datos

- **Follow TCP/HTTP Stream**: clic derecho en un paquete → *Follow → TCP Stream*. Reconstruye toda la conversación (verás el login completo de FTP/HTTP/Telnet en claro).
- **Credenciales automáticas**: menú *Edit → Preferences → Protocols* o, más rápido, con herramientas:
  ```bash
  # Extraer credenciales de un pcap
  tshark -r captura.pcap -Y "http.authbasic" 
  # Herramientas dedicadas
  pcredz -f captura.pcap          # saca credenciales de muchos protocolos
  ```
- **Protocolos con credenciales en claro**: HTTP básico, FTP, Telnet, POP3, IMAP, SNMP (community string), SMB (hashes NetNTLM).

> **Consejo:** en retos "forenses" con un `.pcap`, lo primero es `Statistics → Protocol Hierarchy` para ver qué protocolos hay, y `File → Export Objects → HTTP` para sacar archivos transferidos (imágenes, binarios, documentos con la flag).

---

## Exportar objetos y archivos

`File → Export Objects → HTTP / SMB / FTP-DATA` extrae los archivos que viajaron por la red. Muy usado en CTF forense para recuperar la imagen/documento/binario con la flag.

```bash
# Con tshark: exportar objetos HTTP
tshark -r captura.pcap --export-objects http,/tmp/salida
```

---

## Estadísticas útiles

| Menú | Para qué |
| ---- | -------- |
| `Statistics → Protocol Hierarchy` | Qué protocolos hay y su volumen. |
| `Statistics → Conversations` | Quién habla con quién (IPs y puertos). |
| `Statistics → Endpoints` | Lista de hosts involucrados. |
| `Statistics → HTTP → Requests` | Todas las URLs solicitadas. |

---

## Descifrar TLS (si tienes la clave)

Si dispones de la clave privada del servidor o de las *session keys* (`SSLKEYLOGFILE`), Wireshark descifra HTTPS: *Preferences → Protocols → TLS → (Pre)-Master-Secret log filename*.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `wireshark` | Análisis gráfico de paquetes. |
| `tshark` | Wireshark por línea de comandos (scripting, servidores). |
| `tcpdump` | Captura ligera sin GUI. |
| `pcredz` / `NetworkMiner` | Extracción automática de credenciales y archivos. |

---

> **Consejo:** Wireshark brilla en dos escenarios: (1) **retos forenses** con un `.pcap` → protocolo jerarquía + follow stream + export objects; (2) **depurar tus ataques** → captura en `tun0` y mira qué pasa de verdad por el cable cuando un exploit o una reverse shell "no conecta". Saber leer tráfico te hace entender los protocolos de verdad, no solo lanzar comandos.

---

## Dónde practicar con Wireshark

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| Malware-Traffic-Analysis.net | Ejercicios con pcaps reales | https://www.malware-traffic-analysis.net |
| CyberDefenders | Retos Blue Team / forense (pcaps) | https://cyberdefenders.org |
| picoCTF — categoría *Forensics* | Retos gratis | https://picoctf.org |
| HTB — Sherlocks (DFIR) | Retos de análisis | https://www.hackthebox.com |
| Wireshark Sample Captures | pcaps de ejemplo oficiales | https://wiki.wireshark.org/SampleCaptures |

> **Consejo:** descárgate un pcap de *Malware-Traffic-Analysis* y practica el flujo: `Statistics → Protocol Hierarchy` → *Follow TCP Stream* en lo sospechoso → `File → Export Objects` para sacar los archivos. Es exactamente lo que piden los retos forenses de CTF.
