# Encontrar IPs en mi Red

Antes de comenzar una auditoría es recomendable identificar qué dispositivos están activos en la red local. Existen varias herramientas que permiten descubrir hosts mediante ARP o ICMP.

---

# Descubrimiento con arp-scan

`arp-scan` realiza peticiones ARP para identificar todos los dispositivos activos de la red local.

```bash
arp-scan -I eth0 --localnet
```

**Ventajas:**

* Muy rápido.
* No depende de que ICMP esté habilitado.
* Muestra la dirección MAC y, en muchos casos, el fabricante del dispositivo.

---

# Descubrimiento con netdiscover

Escanea un rango de direcciones utilizando peticiones ARP.

```bash
netdiscover -i eth0 -r <red>
```

Ejemplo:

```bash
netdiscover -i eth0 -r 192.168.1.0/24
```

---

# Descubrimiento con Nmap

Realiza un **Ping Scan** para identificar hosts activos.

```bash
nmap -sn <red> -oN <nombre_archivo>
```

Ejemplo:

```bash
nmap -sn 192.168.1.0/24 -oN descubrimiento.txt
```

---

# Obtener tu dirección IP

Si no conoces la IP de tu máquina:

```bash
ip addr
```

o

```bash
hostname -I
```

---

# Leyenda

| Parámetro          | Descripción                                                                                                           |
| ------------------ | --------------------------------------------------------------------------------------------------------------------- |
| `<red>`            | Sustituye por la red que deseas escanear. Normalmente será la IP de tu equipo cambiando el último octeto por `.0/24`. |
| `<nombre_archivo>` | Nombre del archivo donde Nmap guardará el resultado.                                                                  |

**Ejemplo:**

Si tu dirección IP es:

```text
192.168.1.10
```

La red a escanear será:

```text
192.168.1.0/24
```

---

# Herramientas útiles

| Herramienta   | Descripción                                                    |
| ------------- | -------------------------------------------------------------- |
| `arp-scan`    | Descubre equipos activos mediante peticiones ARP.              |
| `netdiscover` | Descubrimiento de hosts en redes locales mediante ARP.         |
| `nmap`        | Escáner de red para descubrimiento y enumeración de servicios. |
| `ip addr`     | Muestra la configuración de las interfaces de red.             |
| `hostname -I` | Muestra la dirección IP asignada al equipo.                    |

---

> **Consejo:** En redes locales, **arp-scan** suele ser la opción más rápida y fiable para descubrir equipos activos, ya que utiliza peticiones ARP en lugar de depender de respuestas ICMP, que pueden estar bloqueadas por un firewall.
