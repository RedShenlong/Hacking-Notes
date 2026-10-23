# Protocolo DNS (Domain Name System)

> **Puerto por defecto:** `53/UDP` y `53/TCP` (TCP se usa para transferencias de zona y respuestas grandes)

DNS traduce nombres (`maquina.dominio.local`) a direcciones IP. En una auditoría, un servidor DNS mal configurado puede **filtrar todo el mapa de la red interna**: nombres de máquinas, subdominios y servicios que no verías de otra forma.

---

## Enumeración

```bash
nmap -p53 -sCV <IP>
nmap -p53 --script "dns-*" <IP>
```

### Consultas básicas

```bash
# Registros de un dominio
dig @<IP> dominio.local ANY
dig @<IP> dominio.local A
dig @<IP> dominio.local MX      # servidores de correo
dig @<IP> dominio.local NS      # servidores de nombres
dig @<IP> dominio.local TXT     # texto (a veces SPF, claves, pistas)

# Resolución inversa (IP -> nombre)
dig @<IP> -x <IP_objetivo>
```

---

## Transferencia de zona (AXFR) — el premio gordo

Si el servidor permite transferencias de zona a cualquiera, te entrega **todos los registros del dominio** de golpe:

```bash
dig @<IP> dominio.local AXFR
```

Con otras herramientas:

```bash
fierce --domain dominio.local --dns-servers <IP>
dnsrecon -d dominio.local -n <IP> -t axfr
```

> **Consejo:** una transferencia de zona exitosa suele darte el mapa completo: controladores de dominio, servidores web internos, mail, hosts de desarrollo... Es oro. Pruébala siempre que veas el puerto 53 abierto.

---

## Enumeración de subdominios (DNS público / vhosts)

Cuando trabajas con un dominio real o un lab con nombres:

```bash
# Por diccionario
gobuster dns -d dominio.local -r <IP> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Alternativas
dnsenum --dnsserver <IP> dominio.local
```

### Añadir el dominio a /etc/hosts

Muchas máquinas (sobre todo HTB/AD) requieren resolver el nombre para que la web responda bien:

```bash
echo "<IP> dominio.local dc01.dominio.local" | sudo tee -a /etc/hosts
```

> **Nota:** si una web redirige a un nombre de dominio o el certificado TLS menciona uno, añádelo a `/etc/hosts` apuntando a la IP. Es un paso que mucha gente olvida y deja la máquina "atascada".

---

## Fuzzing de vhosts por HTTP (relacionado)

El DNS y los **virtual hosts** van de la mano. Si varios sitios comparten IP, descúbrelos con la cabecera `Host`:

```bash
ffuf -u http://<IP>/ -H "Host: FUZZ.dominio.local" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -fs <tamaño_respuesta_por_defecto>
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `dig` | Consultas DNS manuales (incluida AXFR). |
| `host` / `nslookup` | Consultas rápidas. |
| `dnsrecon` / `dnsenum` / `fierce` | Enumeración y transferencia de zona. |
| `gobuster dns` / `ffuf` | Fuerza bruta de subdominios y vhosts. |

---

> **Consejo:** DNS es a menudo el paso que revela que estás ante un **dominio Active Directory**: si el puerto 53 está en la misma máquina que 88 (Kerberos) y 389 (LDAP), es un Domain Controller. Enumera el DNS para sacar nombres de máquinas y añádelos a `/etc/hosts` antes de atacar AD.
