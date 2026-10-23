# RPC / rpcbind (Remote Procedure Call)

> **Puerto por defecto:** `111/TCP-UDP` (rpcbind/portmapper) · `135/TCP` (MSRPC en Windows)

RPC permite a un programa invocar procedimientos en otra máquina. El **portmapper** (`111`) es el "directorio" que dice en qué puertos viven los servicios RPC (NFS, NIS, etc.). En Windows, **MSRPC** (`135`) es la base de muchos servicios y, junto con SMB, permite enumerar el dominio con `rpcclient`.

---

## Enumeración del portmapper (Linux, puerto 111)

```bash
nmap -sSUC -p111 <IP>
rpcinfo <IP>
rpcinfo -p <IP>
```

`rpcinfo` lista los servicios RPC registrados y sus puertos. Lo más común es descubrir **NFS** (`nfs`, `mountd`) → ve a la nota de [NFS](./Protocolo%20NFS.md).

```text
program vers proto   port  service
100000    4   tcp    111   portmapper
100003    3   tcp   2049   nfs          <- montable
100005    1   udp  20048   mountd
```

> **Consejo:** si `rpcinfo` revela `nfs`/`mountd`, salta directamente a enumerar NFS (`showmount -e <IP>`). El portmapper suele ser el hilo que te lleva a recursos compartidos montables.

---

## MSRPC y rpcclient (Windows / Active Directory, puerto 135/445)

`rpcclient` enumera un dominio Windows por RPC. Prueba primero una **sesión nula** (sin credenciales):

```bash
rpcclient -U "" -N <IP>
rpcclient -U "<usuario>%<pass>" <IP>
```

### Comandos clave dentro de rpcclient

```text
srvinfo                 # info del servidor y versión
enumdomusers            # lista de usuarios del dominio
enumdomgroups           # grupos
querydispinfo           # usuarios con descripción (¡busca contraseñas!)
queryuser 0x<RID>       # detalle de un usuario por su RID
getdompwinfo            # política de contraseñas
lsaenumsid              # SIDs conocidos
lookupnames administrator   # nombre -> SID
```

### RID cycling (sacar usuarios sin lista previa)

Si `enumdomusers` falla pero la sesión nula funciona en parte, enumera por RID:

```bash
for i in $(seq 500 1100); do rpcclient -U "" -N <IP> -c "queryuser 0x$(printf %x $i)" 2>/dev/null | grep "User Name"; done
# o directo con netexec:
netexec smb <IP> -u '' -p '' --rid-brute
```

---

## Qué hacer con lo enumerado

```pipeline
rpcclient / RID brute -> lista de usuarios -> AS-REP / password spraying -> shell
```

La lista de usuarios alimenta los ataques de Active Directory → [Kerberoasting y AS-REP](../Active%20Directory/Kerberoasting%20y%20AS-REP%20Roasting.md) y [LDAP](./Protocolo%20LDAP.md).

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `rpcinfo` | Enumerar servicios RPC registrados (portmapper). |
| `rpcclient` | Enumeración de dominios Windows por MSRPC. |
| `netexec` (`--rid-brute`) | RID cycling para sacar usuarios. |
| `nmap` (`rpcinfo`, `msrpc-*`) | Detección y enumeración. |
| `enum4linux-ng` | Envuelve rpcclient/smbclient para enumerar todo de golpe. |

---

> **Consejo:** en Linux, el puerto 111 es un cartel de "aquí hay NFS/servicios montables". En Windows, `rpcclient` con **sesión nula** es un intento obligatorio: muchas máquinas antiguas o mal configuradas te entregan la lista completa de usuarios del dominio sin credenciales, y de ahí arranca toda la cadena de AD.
