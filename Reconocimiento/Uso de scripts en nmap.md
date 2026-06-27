# Uso de Scripts en Nmap (NSE)

Los **Nmap Scripting Engine (NSE)** permiten ampliar las capacidades de Nmap para realizar tareas de enumeración, detección de vulnerabilidades, autenticación y recopilación de información sobre los servicios encontrados.

---

# Scripts de vulnerabilidades

Ejecuta los scripts de las categorías **`vuln`** y **`safe`** sobre un puerto concreto.

```bash
nmap --script "vuln and safe" -p<PUERTO> <IP>
```

Ejemplo:

```bash
nmap --script "vuln and safe" -p80 192.168.1.10
```

---

# Scripts por servicio

## HTTP

```bash
nmap --script "http*" -p80,443 <IP>
```

## FTP

```bash
nmap --script "ftp*" -p21 <IP>
```

## SMB

```bash
nmap --script "smb*" -p445 <IP>
```

## SSH

```bash
nmap --script "ssh*" -p22 <IP>
```

## SNMP

```bash
nmap --script "snmp*" -p161 -sU <IP>
```

---

# Categorías más utilizadas

| Categoría   | Descripción                                           |
| ----------- | ----------------------------------------------------- |
| `safe`      | Scripts seguros y no intrusivos.                      |
| `default`   | Scripts ejecutados con `-sC`.                         |
| `discovery` | Obtención de información del objetivo.                |
| `auth`      | Comprobaciones relacionadas con autenticación.        |
| `brute`     | Ataques de fuerza bruta contra servicios compatibles. |
| `vuln`      | Detección de vulnerabilidades conocidas.              |
| `malware`   | Detección de posibles infecciones.                    |
| `exploit`   | Scripts que intentan explotar vulnerabilidades.       |

---

# Buscar scripts disponibles

Listar todos los scripts instalados:

```bash
ls /usr/share/nmap/scripts/
```

Buscar un servicio concreto:

```bash
ls /usr/share/nmap/scripts/ | grep http
```

```bash
ls /usr/share/nmap/scripts/ | grep smb
```

---

# Actualizar la base de datos de scripts

Si se añaden nuevos scripts:

```bash
sudo nmap --script-updatedb
```

---

# Leyenda

| Parámetro         | Descripción                                                              |
| ----------------- | ------------------------------------------------------------------------ |
| `--script`        | Especifica el script o categoría de scripts que se ejecutará.            |
| `"vuln and safe"` | Ejecuta únicamente los scripts de vulnerabilidades considerados seguros. |
| `-p<PUERTO>`      | Puerto o puertos sobre los que se ejecutarán los scripts.                |
| `<IP>`            | Dirección IP del objetivo.                                               |

---

# Ejemplos útiles

Detectar vulnerabilidades SMB:

```bash
nmap --script smb-vuln* -p445 <IP>
```

Enumerar un servidor HTTP:

```bash
nmap --script http-enum -p80 <IP>
```

Comprobar si un servidor es vulnerable a MS17-010:

```bash
nmap --script smb-vuln-ms17-010 -p445 <IP>
```

---

> **Consejo:** Evita ejecutar scripts NSE sobre **todos los puertos (`-p-`)**, ya que incrementa considerablemente el tiempo de escaneo. Lo habitual es identificar primero los puertos abiertos con un escaneo inicial y, después, lanzar los scripts únicamente sobre los servicios detectados.
