# Protocolo SAMBA (SMB)

**Server Message Block (SMB)** es un protocolo de compartición de archivos e impresoras utilizado principalmente en sistemas Windows, aunque también es compatible con sistemas Linux mediante **Samba**. Durante una auditoría es habitual enumerar recursos compartidos, usuarios, permisos y posibles vulnerabilidades antes de intentar su explotación.

---

# Enumeración

## Identificar el servicio

```bash
nmap -Pn -sV -p139,445 <IP>
```

```bash
nmap --script smb-protocols,smb-os-discovery -p139,445 <IP>
```

## Enumerar recursos compartidos

### smbclient

```bash
smbclient -L //<IP> -N
```

```bash
smbclient -L //<IP> -U <usuario>
```

### enum4linux

```bash
enum4linux <IP>
```

```bash
enum4linux -a <IP>
```

### enum4linux-ng

```bash
enum4linux-ng <IP>
```

### crackmapexec

```bash
crackmapexec smb <IP>
```

## Enumerar usuarios

```bash
rpcclient -U "" -N <IP>
```

Dentro de `rpcclient`:

```text
enumdomusers
querydispinfo
enumdomgroups
```

## Acceder a un recurso compartido

```bash
smbclient //<IP>/<recurso> -N
```

```bash
smbclient //<IP>/<recurso> -U <usuario>
```

---

# Explotación

## Acceso con credenciales

```bash
smbclient //<IP>/<recurso> -U <usuario>
```

## Descargar archivos

```text
get archivo.txt
mget *
```

## Subir archivos

```text
put shell.aspx
```

## Montar un recurso SMB

```bash
sudo mount -t cifs //<IP>/<recurso> /mnt/smb -o username=<usuario>,password=<contraseña>
```

## Ejecución remota (credenciales válidas)

### Impacket - psexec

```bash
psexec.py <dominio>/<usuario>:<contraseña>@<IP>
```

### Impacket - smbexec

```bash
smbexec.py <dominio>/<usuario>:<contraseña>@<IP>
```

### Impacket - wmiexec

```bash
wmiexec.py <dominio>/<usuario>:<contraseña>@<IP>
```

### NetExec (CrackMapExec)

```bash
netexec smb <IP> -u <usuario> -p <contraseña>
```

```bash
netexec smb <IP> -u <usuario> -p <contraseña> --shares
```

```bash
netexec smb <IP> -u <usuario> -p <contraseña> --sessions
```

```bash
netexec smb <IP> -u <usuario> -p <contraseña> --sam
```

---

# Vulnerabilidades comunes

## EternalBlue (MS17-010)

```bash
nmap --script smb-vuln-ms17-010 -p445 <IP>
```

## Enumeración de vulnerabilidades SMB

```bash
nmap --script smb-vuln* -p445 <IP>
```

---

# Herramientas útiles

| Herramienta | Descripción |
|------------|-------------|
| `smbclient` | Cliente SMB para acceder a recursos compartidos. |
| `enum4linux` | Enumeración de usuarios, grupos y recursos SMB. |
| `enum4linux-ng` | Versión mejorada de enum4linux. |
| `rpcclient` | Enumeración mediante RPC. |
| `NetExec` | Framework para auditorías SMB, WinRM, LDAP, MSSQL, etc. |
| `Impacket` | Suite de herramientas para autenticación y ejecución remota. |
| `nmap` | Detección de servicios y vulnerabilidades SMB. |

---

> **Nota:** SMB suele ser uno de los primeros servicios a enumerar en una auditoría de Active Directory, ya que puede revelar usuarios, recursos compartidos, credenciales expuestas y vectores de movimiento lateral.
