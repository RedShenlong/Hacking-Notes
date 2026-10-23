# RedShenlong — Hacking Notes 🐉

![Format](https://img.shields.io/badge/Format-Markdown%20(.md)-blue.svg)
![Lang](https://img.shields.io/badge/Idioma-Espa%C3%B1ol-red.svg)
![OS](https://img.shields.io/badge/Entorno-Linux%20%2F%20Arch-lightgrey.svg)
![Status](https://img.shields.io/badge/Estado-Activo-green.svg)
![License](https://img.shields.io/badge/Conocimiento-Libre%20y%20Gratuito-success.svg)

> **Manual abierto, gratuito y en español para aprender hacking ético desde cero.**
> Una base de conocimiento y *cheat sheet* de seguridad ofensiva: desde el primer día hasta preparar certificaciones como la **OSCP**.

Este repositorio nace de una idea simple: **el conocimiento debe ser libre**. Aquí encontrarás la metodología, las técnicas y los comandos que necesitas para empezar de cero, resolver máquinas (DockerLabs, Hack The Box, TryHackMe) y crecer como profesional — sin pagar nada.

---

## 🚀 ¿Por dónde empiezo?

¿Prefieres un plan de estudios estructurado, módulo a módulo? → **[📚 Temario del Curso](./00%20-%20Empezar%20Aqu%C3%AD/Temario%20del%20Curso.md)**.

Si acabas de llegar, lee estas cuatro páginas **en orden**. Son el corazón del manual:

| # | Página | Para qué |
| - | ------ | -------- |
| 1 | [🧭 Metodología de Pentesting](./00%20-%20Empezar%20Aqu%C3%AD/Metodolog%C3%ADa%20de%20Pentesting.md) | **Cómo comprometer cualquier máquina**, paso a paso. La página más importante. |
| 2 | [🗺️ Roadmap desde Cero](./00%20-%20Empezar%20Aqu%C3%AD/Roadmap%20desde%20Cero.md) | La ruta de 0 a hacker: qué aprender y en qué orden. |
| 3 | [🎓 Certificaciones](./00%20-%20Empezar%20Aqu%C3%AD/Certificaciones.md) | Qué certificación sacar según el sector (OSCP, CPTS, OSWE…). |
| 4 | [🧪 Laboratorios para Practicar](./00%20-%20Empezar%20Aqu%C3%AD/Laboratorios%20para%20Practicar.md) | Dónde practicar legal y gratis. |
| 5 | [🚩 Resolver una Máquina Paso a Paso](./00%20-%20Empezar%20Aqu%C3%AD/Ejemplo%20Resolver%20una%20M%C3%A1quina%20Paso%20a%20Paso.md) | Un ejemplo completo de 0 a root encadenando las notas. |
| 6 | [🏢 Comprometer un Dominio Active Directory](./00%20-%20Empezar%20Aqu%C3%AD/Ejemplo%20Comprometer%20un%20Dominio%20Active%20Directory.md) | Walkthrough de un entorno AD de 0 a Domain Admin. |

Y para montar tu entorno: [⚙️ Montar tu Laboratorio](./00%20-%20Empezar%20Aqu%C3%AD/Montar%20tu%20Laboratorio.md). Chuleta rápida: [⌨️ Comandos Esenciales](./00%20-%20Empezar%20Aqu%C3%AD/Comandos%20Esenciales.md). ¿Te pierdes con algún término? → [📖 Glosario](./00%20-%20Empezar%20Aqu%C3%AD/Glosario.md).

Para ir más allá: [🧠 Mentalidad para Máquinas Difíciles](./00%20-%20Empezar%20Aqu%C3%AD/Mentalidad%20para%20M%C3%A1quinas%20Dif%C3%ADciles.md) · [🃏 Trucos y Atajos poco Conocidos](./00%20-%20Empezar%20Aqu%C3%AD/Trucos%20y%20Atajos%20poco%20Conocidos.md).

---

## 🗂️ Estructura del repositorio

Las notas siguen el **flujo lógico de una auditoría**, igual que cuando atacas una máquina real:

```text
Hacking-Notes/
├── 00 - Empezar Aquí/              → Metodología, roadmap, certs, labs
├── Reconocimiento/                 → Descubrir hosts, puertos y servicios
├── Enumeración y Explotación       → Atacar cada protocolo/servicio
│   de Protocolos/
├── Técnicas de Hacking             → Shells, netcat, fuerza bruta, transfer
│   Fundamentales/
├── Vulnerabilidades/
│   ├── Web/                        → OWASP Top 10 (SQLi, XSS…)
│   └── APIs/                       → OWASP API Top 10 + laboratorios
├── Escalada de Privilegios/        → De usuario a root / SYSTEM
└── Active Directory/               → Pentesting de dominios Windows
```

### 🔍 Reconocimiento
- [Encontrar IPs en mi Red](./Reconocimiento/Encontrar%20IPs%20en%20mi%20Red.md)
- [Escaneo de Puertos](./Reconocimiento/Escaneo%20de%20Puertos.md)
- [Uso de scripts en Nmap](./Reconocimiento/Uso%20de%20scripts%20en%20nmap.md)
- [Enumeración de Directorios](./Reconocimiento/Enumeraci%C3%B3n%20de%20Directorios.md)

### 🌐 Enumeración y Explotación de Protocolos
- [SMB](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20SMB.md) · [SAMBA](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20SAMBA.md) · [RDP](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20RDP.md) · [SNMP](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20SNMP.md)
- [FTP](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20FTP.md) · [SSH](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20SSH.md) · [DNS](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20DNS.md) · [SMTP](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20SMTP.md) · [LDAP](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20LDAP.md) · [NFS](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20NFS.md) · [WinRM](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/WinRM.md) · [Redis](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Redis.md)
- [MySQL](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Bases%20de%20Datos%20MySQL.md) · [MSSQL](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Bases%20de%20Datos%20MSSQL.md) · [PostgreSQL](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Bases%20de%20Datos%20PostgreSQL.md)
- [RPC / rpcbind](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Protocolo%20RPC.md) · [WordPress y CMS](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/WordPress%20y%20CMS.md)
- [Tomcat](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Tomcat.md) · [Web Shell ASPX / IIS](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Web%20Shell%20ASPX%20Microsoft%20IIS%20%281%29.md) · [HotFixes Windows](./Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/Enumeraci%C3%B3n%20de%20HotFixes%20Windows.md)

### ⚙️ Técnicas Fundamentales
- [Shells y estabilización de TTY](./T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Shells%20y%20TTY.md) · [Netcat](./T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Netcat.md)
- [Pivoting y Tunneling](./T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Pivoting%20y%20Tunneling.md)
- [Fuerza Bruta](./T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Fuerza%20Bruta.md) · [Cracking de Hashes](./T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Cracking%20de%20Hashes.md)
- [Compartir Archivos por la Red](./T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Compartir%20Archivos%20por%20la%20Red.md)

### 🧰 Herramientas
- [Burp Suite (Guía Completa)](./Herramientas/Burp%20Suite%20%28Gu%C3%ADa%20Completa%29.md) · [Wireshark](./Herramientas/Wireshark.md) · [tmux (Flujo de Trabajo)](./Herramientas/tmux%20%28Flujo%20de%20Trabajo%29.md)

### 💥 Vulnerabilidades
- **Web:** [SQL Injection](./Vulnerabilidades/Web/SQL%20Injection.md) · [XSS](./Vulnerabilidades/Web/XSS.md) · [LFI / RFI](./Vulnerabilidades/Web/LFI%20y%20RFI.md) · [Command Injection](./Vulnerabilidades/Web/Command%20Injection.md) · [SSTI](./Vulnerabilidades/Web/SSTI.md) · [Subida de Archivos](./Vulnerabilidades/Web/Subida%20de%20Archivos.md) · [IDOR](./Vulnerabilidades/Web/IDOR.md) · [CSRF](./Vulnerabilidades/Web/CSRF.md) · [CORS](./Vulnerabilidades/Web/CORS.md) · [Deserialización](./Vulnerabilidades/Web/Deserializaci%C3%B3n.md) · [Open Redirect](./Vulnerabilidades/Web/Open%20Redirect.md)
- **APIs:** [Introducción](./Vulnerabilidades/APIs/00%20-%20Introducci%C3%B3n.md) · [Enumeración](./Vulnerabilidades/APIs/01%20-%20Enumeraci%C3%B3n.md) · [Burp](./Vulnerabilidades/APIs/02%20-%20Burp%20Suite.md) · [Postman](./Vulnerabilidades/APIs/03%20-%20Postman.md) · [JWT](./Vulnerabilidades/APIs/04%20-%20JWT.md)
  - Vulns: [BOLA](./Vulnerabilidades/APIs/Vulnerabilidades/BOLA.md) · [BFLA](./Vulnerabilidades/APIs/Vulnerabilidades/BFLA.md) · [Mass Assignment](./Vulnerabilidades/APIs/Vulnerabilidades/Mass%20Assignment.md) · [Excessive Data Exposure](./Vulnerabilidades/APIs/Vulnerabilidades/Excessive%20Data%20Exposure.md) · [Rate Limiting](./Vulnerabilidades/APIs/Vulnerabilidades/Rate%20Limiting.md) · [SSRF](./Vulnerabilidades/APIs/Vulnerabilidades/SSRF.md) · [XXE](./Vulnerabilidades/APIs/Vulnerabilidades/XXE.md) · [SQLi](./Vulnerabilidades/APIs/Vulnerabilidades/SQL%20Injection.md) · [API Keys](./Vulnerabilidades/APIs/Vulnerabilidades/API%20Keys.md)
  - Labs: [VAmPI](./Vulnerabilidades/APIs/Vulnerabilidades/Laboratorios/VAmPI.md) · [crAPI](./Vulnerabilidades/APIs/Vulnerabilidades/Laboratorios/OWASP%20crAPI.md) · [Juice Shop](./Vulnerabilidades/APIs/Vulnerabilidades/Laboratorios/OWASP%20Juice%20Shop.md) · [DVGA](./Vulnerabilidades/APIs/Vulnerabilidades/Laboratorios/DVGA.md)

### 🔺 Escalada de Privilegios
- [Linux](./Escalada%20de%20Privilegios/Linux.md) · [Windows](./Escalada%20de%20Privilegios/Windows.md)

### 🏢 Active Directory
- [Introducción y Metodología](./Active%20Directory/00%20-%20Introducci%C3%B3n.md) · [Kerberoasting y AS-REP](./Active%20Directory/Kerberoasting%20y%20AS-REP%20Roasting.md) · [BloodHound](./Active%20Directory/BloodHound.md) · [Movimiento Lateral](./Active%20Directory/Movimiento%20Lateral.md) · [DCSync y Volcado de Credenciales](./Active%20Directory/DCSync%20y%20Volcado%20de%20Credenciales.md)

---

## 🔎 Buscar contenido rápido

```bash
# Con ripgrep (recomendado)
rg -i "kerberoasting"

# Con grep
grep -rni "SUID" .
```

---

## 🤝 Contribuir

Este es un proyecto vivo y de código abierto. Si quieres aportar una técnica, corregir un error o añadir una máquina resuelta, los *pull requests* son bienvenidos. La idea es construir, entre todos, **el manual de hacking en español más completo y gratuito**.

El estilo de las notas es consistente: título, explicación breve, comandos en bloques, tabla de "Leyenda"/"Herramientas relacionadas" y un bloque final de **Consejo**.

---

## ⚠️ Aviso Legal (Disclaimer)

> **IMPORTANTE**
>
> Todo el contenido de este repositorio tiene fines **exclusivamente educativos, de investigación y de uso en laboratorios autorizados** (TryHackMe, Hack The Box, DockerLabs, VulnHub o entornos propios).
>
> El uso de estas técnicas contra sistemas de terceros **sin autorización expresa y por escrito es ilegal**. Ni el autor ni los colaboradores se hacen responsables del uso indebido de esta información.
>
> Hackea solo lo que te pertenezca o lo que te hayan autorizado a hackear. El objetivo es formar profesionales de la **defensa** y la **seguridad ofensiva ética**.

---

> *"Que viva la educación."* — El conocimiento compartido nos hace a todos más fuertes. 🐉
