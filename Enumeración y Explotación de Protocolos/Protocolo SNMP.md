# Protocolo SNMP (Simple Network Management Protocol)

> **Puerto por defecto:** `161/UDP`

SNMP (Simple Network Management Protocol) es un protocolo utilizado para monitorizar y administrar dispositivos de red como routers, switches, impresoras, servidores y equipos IoT. Una configuración insegura o una **community string** débil puede permitir obtener una gran cantidad de información sensible del sistema.

---

# Enumeración

## Descubrir la Community String

Probar un diccionario de posibles claves de comunidad utilizando **onesixtyone**.

```bash
onesixtyone -c /usr/share/wordlists/rockyou.txt <IP>
```

También es habitual utilizar listas específicas de comunidades SNMP:

```bash
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt <IP>
```

---

## Enumerar información con snmpwalk

Una vez obtenida la **community string**, se puede consultar toda la información publicada por el servicio.

```bash
snmpwalk -v2c -c <community_string> <IP>
```

Ejemplo:

```bash
snmpwalk -v2c -c public 192.168.1.100
```

---

## Enumerar un OID concreto

Consultar únicamente una rama del árbol SNMP:

```bash
snmpwalk -v2c -c <community_string> <IP> <OID>
```

Ejemplo:

```bash
snmpwalk -v2c -c public <IP> 1.3.6.1.2.1.1
```

---

# Información interesante

Durante una auditoría es habitual encontrar:

* Usuarios del sistema.
* Nombres de host.
* Dominios.
* Interfaces de red.
* Direcciones IP.
* Procesos en ejecución.
* Software instalado.
* Información del sistema operativo.
* Configuración de dispositivos de red.
* Contactos y ubicación del dispositivo.
* Rutas y tablas de red.

---

# Herramientas útiles

| Herramienta   | Descripción                                                  |
| ------------- | ------------------------------------------------------------ |
| `onesixtyone` | Descubre community strings mediante fuerza bruta.            |
| `snmpwalk`    | Enumera información del árbol SNMP.                          |
| `snmp-check`  | Obtiene un informe resumido de la información SNMP.          |
| `nmap`        | Detecta el servicio SNMP y ejecuta scripts NSE relacionados. |

---

# Enumeración con Nmap

Detectar el servicio:

```bash
nmap -sU -p161 <IP>
```

Enumerar mediante scripts NSE:

```bash
nmap -sU -p161 --script snmp* <IP>
```

---

> **Consejo:** Las *community strings* más comunes son `public`, `private` y `manager`. Siempre prueba primero estas antes de lanzar un ataque de fuerza bruta, ya que siguen siendo frecuentes en laboratorios y entornos mal configurados.
