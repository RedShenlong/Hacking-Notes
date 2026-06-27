# Escaneo de Puertos con Nmap

El escaneo de puertos es el primer paso de cualquier auditoría de red. Permite identificar los servicios expuestos y constituye la base para la posterior fase de enumeración y explotación.

---

# Escaneo TCP

Escanea los **65535 puertos TCP**, detecta servicios, versiones y ejecuta los scripts básicos de Nmap.

```bash id="grzmgo"
nmap \
-p- \
--open \
-sS \
-sC \
-sV \
--min-rate 2000 \
-n \
-vvv \
-Pn \
<IpVictima> \
-oN <nombre>
```

---

# Escaneo UDP

Escanea los **200 puertos UDP más comunes**.

```bash id="bkdf4f"
nmap \
-sU \
--top-ports 200 \
--min-rate 2000 \
-Pn \
<IpVictima> \
-oN <nombre>
```

> **Nota:** El escaneo UDP suele ser mucho más lento que el TCP y puede producir falsos positivos debido a la ausencia de respuesta de muchos servicios.

---

# Escaneo dirigido a puertos encontrados

Una vez identificados los puertos abiertos, es recomendable realizar un escaneo más detallado únicamente sobre ellos.

```bash id="d2v9hz"
nmap \
-sCV \
-p<PUERTOS> \
<IpVictima> \
-oN servicios.txt
```

Ejemplo:

```bash id="hrvhvf"
nmap -sCV -p22,80,443 192.168.1.10
```

---

# Leyenda

| Parámetro         | Descripción                                                 |
| ----------------- | ----------------------------------------------------------- |
| `-p-`             | Escanea los **65535 puertos TCP**.                          |
| `--open`          | Muestra únicamente los puertos abiertos.                    |
| `-sS`             | Escaneo SYN (Half-Open Scan). Es el modo TCP más utilizado. |
| `-sU`             | Escaneo de puertos UDP.                                     |
| `-sC`             | Ejecuta los scripts NSE por defecto.                        |
| `-sV`             | Detecta la versión de los servicios encontrados.            |
| `--top-ports 200` | Escanea los 200 puertos más frecuentes.                     |
| `--min-rate 2000` | Envía como mínimo 2000 paquetes por segundo.                |
| `-n`              | No realiza resolución DNS.                                  |
| `-vvv`            | Incrementa el nivel de detalle de la salida.                |
| `-Pn`             | Asume que el host está activo y omite el ping previo.       |
| `-oN <nombre>`    | Guarda el resultado en formato de texto plano.              |

---

# Scripts NSE útiles

Detección de vulnerabilidades:

```bash id="o2wjwg"
nmap --script vuln <IP>
```

Enumerar un servicio concreto:

```bash id="p32od7"
nmap --script "ftp*"
```

```bash id="u7ry2r"
nmap --script "http*"
```

```bash id="b22dn7"
nmap --script "smb*"
```

Listar todos los scripts disponibles:

```bash id="q2s49m"
ls /usr/share/nmap/scripts/
```

---

# Guardar resultados en varios formatos

```bash id="lvom8t"
nmap -oA escaneo <IP>
```

Se generarán:

```text id="xxt5f5"
escaneo.nmap
escaneo.gnmap
escaneo.xml
```

---

# Herramientas relacionadas

| Herramienta | Descripción                                                  |
| ----------- | ------------------------------------------------------------ |
| `nmap`      | Escáner de puertos y servicios más utilizado.                |
| `masscan`   | Escáner extremadamente rápido para grandes redes.            |
| `rustscan`  | Descubre puertos rápidamente y delega la enumeración a Nmap. |

---

> **Consejo:** Un flujo muy utilizado consiste en realizar primero un escaneo rápido de todos los puertos (`-p-`) para identificar los abiertos y, posteriormente, ejecutar un segundo escaneo con `-sCV` únicamente sobre esos puertos. De este modo se obtiene información más detallada reduciendo el tiempo total del análisis.
