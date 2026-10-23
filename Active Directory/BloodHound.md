# BloodHound (mapear el camino a Domain Admin)

**BloodHound** recolecta información del dominio y la representa como un **grafo**: usuarios, grupos, equipos, sesiones y permisos. Su magia es responder a la pregunta *"desde donde estoy, ¿cómo llego a Domain Admin?"* dibujando la ruta más corta. Convierte un Active Directory enorme y confuso en un mapa de "haz esto, luego esto".

> Contexto en [Introducción a Active Directory](./00%20-%20Introducci%C3%B3n.md).

---

## Flujo general

```flow
Recolectar :: Un "ingestor" vuelca el dominio (usuarios, grupos, ACLs, sesiones).
Importar :: Arrastras los .json/.zip a la interfaz de BloodHound.
Consultar :: Usas las consultas preconstruidas y marcas tu usuario como "owned".
Seguir la ruta :: BloodHound dibuja el camino hacia Domain Admin.
```

---

## Paso 1 — Recolectar datos

Necesitas **unas credenciales de dominio** (cualquiera). Dos formas:

### Desde Linux (remoto) — bloodhound-python

```bash
bloodhound-python -u '<user>' -p '<pass>' -d dominio.local -ns <DC_IP> -c All --zip
```

### Con netexec

```bash
netexec ldap <DC_IP> -u <user> -p <pass> --bloodhound --collection All --dns-server <DC_IP>
```

### Desde Windows (dentro de la víctima) — SharpHound

```powershell
.\SharpHound.exe -c All
# o la versión PowerShell
Import-Module .\SharpHound.ps1 ; Invoke-BloodHound -CollectionMethod All
```

> **Consejo:** `-c All` recoge casi todo, pero las **sesiones** (quién tiene sesión abierta en cada equipo) son clave para el movimiento lateral y a veces requieren privilegios. Si una recolección sale pobre, repítela cuando tengas más acceso.

---

## Paso 2 — Importar y marcar

1. Arranca BloodHound (con Neo4j detrás) o usa **BloodHound CE**.
2. Arrastra el `.zip`/`.json` generado a la ventana.
3. Busca tu usuario, clic derecho → **Mark as Owned** (marca lo que ya controlas).
4. (Opcional) marca objetivos de alto valor como *High Value*.

---

## Paso 3 — Consultas que resuelven máquinas

En la pestaña *Analysis* tienes consultas prehechas; las más útiles:

| Consulta | Para qué |
| -------- | -------- |
| **Shortest Paths to Domain Admins** | El camino directo al objetivo. |
| **Shortest Paths from Owned Principals** | Desde lo que ya controlas, ¿a dónde llego? |
| **Find Kerberoastable Accounts** | Cuentas con SPN → [Kerberoasting](./Kerberoasting%20y%20AS-REP%20Roasting.md). |
| **Find AS-REP Roastable Users** | Cuentas sin preauth → AS-REP. |
| **Find Computers with Unconstrained Delegation** | Delegación abusable. |
| **Shortest Paths to High Value Targets** | Rutas a objetos críticos. |

---

## Paso 4 — Interpretar las aristas (edges)

Las flechas del grafo son **relaciones abusables**. Las más comunes:

| Edge | Significado y abuso |
| ---- | ------------------- |
| `MemberOf` | Pertenencia a un grupo (heredas sus permisos). |
| `GenericAll` / `GenericWrite` | Control total/escritura sobre un objeto → cambiar su contraseña, añadir SPN, etc. |
| `WriteDacl` / `Owns` | Puedes modificar los permisos del objeto → darte `GenericAll`. |
| `ForceChangePassword` | Puedes cambiar la contraseña de ese usuario. |
| `AddMember` | Puedes meterte (o meter a alguien) en un grupo. |
| `CanRDP` / `CanPSRemote` | Puedes conectarte a ese equipo (RDP/WinRM). |
| `HasSession` | Un usuario tiene sesión ahí → roba su token/credenciales si comprometes el equipo. |
| `AllowedToDelegate` | Delegación Kerberos abusable. |

> **Consejo:** clic derecho sobre cualquier arista → **Help**: BloodHound te da el **comando exacto** para abusar de esa relación (con PowerView, impacket o bloodyAD). No tienes que memorizar los abusos; la propia herramienta te los dicta.

---

## Ejemplo de ruta típica

```pipeline
Usuario owned -> GenericWrite sobre SVC -> set SPN + Kerberoast -> crackear -> MemberOf grupo privilegiado -> DCSync -> Domain Admin
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `bloodhound-python` | Ingestor remoto desde Linux. |
| `SharpHound` | Ingestor desde Windows (exe/ps1). |
| `netexec --bloodhound` | Recolección integrada. |
| `BloodHound` / `BloodHound CE` | Interfaz del grafo. |
| `bloodyAD` / `PowerView` | Ejecutar los abusos que indica cada edge. |

---

> **Consejo:** BloodHound es la herramienta que más vas a mirar en AD. El método ganador: consigue una credencial → recolecta con `-c All` → marca tu usuario como *owned* → ejecuta **"Shortest Paths from Owned Principals"** → sigue las flechas usando el **Help** de cada edge. No ataques el dominio a ciegas: deja que BloodHound te dibuje el camino.
