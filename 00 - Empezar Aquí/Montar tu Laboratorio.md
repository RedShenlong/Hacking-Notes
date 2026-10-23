# Montar tu Laboratorio de Hacking

Antes de atacar nada necesitas tu propio entorno de trabajo: un sistema con las herramientas ofensivas y una forma **aislada** y **legal** de practicar. Esta guía cubre el montaje en **Linux** (incluyendo **Arch**, como este repo) y las buenas prácticas de red.

> **Principio de aislamiento:** tu laboratorio nunca debe poder atacar internet ni tu red doméstica real. Las máquinas vulnerables son, por definición, inseguras: mantenlas en una red cerrada.

---

## Opción A — Kali Linux (lo más estándar)

Kali trae preinstaladas casi todas las herramientas. La forma recomendada de usarlo es en una **máquina virtual**:

1. Instala **VirtualBox** o **VMware**.
2. Descarga la imagen oficial de Kali (`kali.org`) — hay OVA ya listas para importar.
3. Importa, arranca y actualiza:
   ```bash
   sudo apt update && sudo apt full-upgrade -y
   ```
4. Instala el metapaquete grande si quieres todo el arsenal:
   ```bash
   sudo apt install -y kali-linux-large
   ```

---

## Opción B — Arch Linux + BlackArch (este repo)

Si usas Arch, puedes añadir el repositorio **BlackArch** sobre tu sistema sin reinstalar nada:

```bash
# Añadir el repositorio BlackArch
curl -O https://blackarch.org/strap.sh
echo "$(curl -s https://blackarch.org/strap.sh.sig)"   # verifica la firma antes de ejecutar
chmod +x strap.sh
sudo ./strap.sh
sudo pacman -Syu
```

Instalar herramientas por categorías o sueltas:

```bash
# Ver categorías
sudo pacman -Sg | grep blackarch
# Instalar una herramienta concreta
sudo pacman -S nmap ffuf feroxbuster
# Instalar un grupo entero (p. ej. escáneres)
sudo pacman -S blackarch-scanner
```

> Alternativa sin tocar tu sistema: usa Kali en Docker o en una VM y deja tu Arch limpio.

---

## Herramientas imprescindibles (checklist)

| Categoría | Herramientas |
| --------- | ------------ |
| Escaneo | `nmap`, `masscan`, `rustscan` |
| Web / fuzzing | `ffuf`, `feroxbuster`, `gobuster`, `whatweb`, `nikto` |
| Proxy web | **Burp Suite**, `zaproxy` |
| Enumeración de red | `netexec` (nxc), `enum4linux-ng`, `smbclient`, `smbmap` |
| Explotación | `metasploit`, `searchsploit` (exploitdb) |
| Credenciales | `hydra`, `john`, `hashcat`, `hash-identifier` |
| Shells / transfer | `netcat`, `socat`, `python3` |
| Post-explotación | `linpeas`, `winpeas`, `pspy`, `impacket` |
| Active Directory | `bloodhound`, `netexec`, `impacket`, `evil-winrm` |
| Diccionarios | **SecLists**, `rockyou.txt` |

Instalar SecLists y descomprimir rockyou (si no viene):

```bash
# Debian/Kali
sudo apt install -y seclists
# rockyou suele estar comprimido
gunzip /usr/share/wordlists/rockyou.txt.gz 2>/dev/null
ls /usr/share/seclists/
```

---

## Red del laboratorio (aislamiento)

Para máquinas en VM (VulnHub, etc.):

| Modo de red | Uso | Aislado de internet |
| ----------- | --- | ------------------- |
| **Host-Only** | Atacante y víctima se ven entre sí, pero no salen a internet. | ✅ Sí (recomendado para víctimas) |
| **NAT Network** | Varias VMs en una red privada con salida controlada. | Parcial |
| **Bridge** | La VM aparece en tu red real. | ❌ **Nunca** para máquinas vulnerables |

Descubrir la IP de la víctima en la red del lab:

```bash
# Barrido de hosts vivos
nmap -sn 192.168.56.0/24
# o con arp
arp-scan -l
```

Para **DockerLabs** no necesitas VM: las máquinas son contenedores que levantas en local (ver [Laboratorios para Practicar](./Laboratorios%20para%20Practicar.md)).

---

## Organización del trabajo

Crea una estructura por máquina y acostúmbrate a documentar desde el primer comando:

```bash
mkdir -p ~/labs/<maquina>/{nmap,content,exploits,loot}
cd ~/labs/<maquina>
```

Un fichero `notes.md` por máquina con: IP, puertos, servicios/versiones, credenciales, rutas interesantes y "por revisar". Este hábito es **exactamente** lo que necesitarás el día del examen OSCP.

---

> **Consejo:** no te obsesiones con tener "la distro perfecta". Kali en una VM o BlackArch sobre tu Arch sirven de sobra. Lo que marca la diferencia no es la herramienta que tengas instalada, sino que sepas **cuándo y por qué** usar cada una — para eso está la [Metodología](./Metodolog%C3%ADa%20de%20Pentesting.md).
