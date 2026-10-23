# Temario del Curso

Este es el **plan de estudios** completo, ordenado como un curso profesional de hacking ético (inspirado en la progresión de las mejores academias: fundamentos → ofensiva → especialización). Sigue los módulos en orden si empiezas de cero; si ya tienes base, salta al que necesites.

Cada módulo enlaza a las notas del cuaderno que lo cubren.

```pipeline
Módulo 0: Bases -> M1: Recon -> M2: Enumeración -> M3: Web -> M4: Explotación -> M5: Post/Privesc -> M6: Active Directory -> M7: Pro
```

---

## Módulo 0 — Fundamentos y entorno

Antes de atacar nada: entiende el terreno y monta tu laboratorio.

- [Roadmap desde Cero](./Roadmap%20desde%20Cero.md) — qué aprender y en qué orden.
- [Montar tu Laboratorio](./Montar%20tu%20Laboratorio.md) — Kali / BlackArch, red aislada, arsenal.
- [Glosario](./Glosario.md) — el vocabulario imprescindible.
- [Comandos Esenciales](./Comandos%20Esenciales.md) — la chuleta que tendrás siempre abierta.
- [Herramientas: tmux](../Herramientas/tmux%20%28Flujo%20de%20Trabajo%29.md) — flujo de trabajo en terminal.

> **Objetivo del módulo:** tener un Kali/BlackArch listo, moverte con soltura por la terminal y entender red/Linux básicos.

---

## Módulo 1 — Metodología y Reconocimiento

La mentalidad y el primer contacto con la máquina.

- [Metodología de Pentesting](./Metodolog%C3%ADa%20de%20Pentesting.md) — **la página clave**: cómo comprometer cualquier máquina.
- [Encontrar IPs en mi Red](../Reconocimiento/Encontrar%20IPs%20en%20mi%20Red.md)
- [Escaneo de Puertos](../Reconocimiento/Escaneo%20de%20Puertos.md)
- [Uso de scripts en Nmap](../Reconocimiento/Uso%20de%20scripts%20en%20nmap.md)
- [Enumeración de Directorios](../Reconocimiento/Enumeraci%C3%B3n%20de%20Directorios.md)

> **Objetivo:** ante una IP, saber qué puertos/servicios tiene y trazar un plan.

---

## Módulo 2 — Enumeración de Servicios

Por cada puerto, saber qué sacar. Estudia los que más caen primero (SMB, HTTP, FTP, SSH).

- Carpeta completa de [Protocolos](../Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/): SMB/SAMBA, FTP, SSH, DNS, SNMP, SMTP, LDAP, RPC, NFS, RDP, WinRM, MySQL/MSSQL/PostgreSQL, Redis, Tomcat, IIS, WordPress/CMS.
- [Herramientas: Wireshark](../Herramientas/Wireshark.md) — leer tráfico y entender protocolos.

> **Objetivo:** agotar cada servicio (credenciales por defecto, versiones vulnerables, fugas de info).

---

## Módulo 3 — Hacking Web

El vector de entrada número uno. Aquí se gana la mayoría de máquinas.

- [Herramientas: Burp Suite](../Herramientas/Burp%20Suite%20%28Gu%C3%ADa%20Completa%29.md) — tu herramienta principal en web.
- OWASP Top 10 en [Vulnerabilidades/Web](../Vulnerabilidades/Web/): SQLi, XSS, LFI/RFI, Command Injection, SSTI, Subida de Archivos, IDOR, CSRF, CORS, Open Redirect, Deserialización.
- APIs en [Vulnerabilidades/APIs](../Vulnerabilidades/APIs/): BOLA, BFLA, Mass Assignment, JWT, SSRF, XXE y laboratorios (crAPI, Juice Shop, VAmPI, DVGA).

> **Objetivo:** detectar y explotar las vulns web, y llevarlas a RCE. Practica con los labs que trae cada nota.

---

## Módulo 4 — Explotación y Acceso Inicial

Convertir una vulnerabilidad en una shell.

- [Shells y TTY](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Shells%20y%20TTY.md) — reverse/bind shells y estabilización.
- [Netcat](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Netcat.md)
- [Compartir Archivos por la Red](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Compartir%20Archivos%20por%20la%20Red.md)
- [Fuerza Bruta](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Fuerza%20Bruta.md) y [Cracking de Hashes](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Cracking%20de%20Hashes.md)

> **Objetivo:** foothold estable y cómodo, listo para enumerar por dentro.

---

## Módulo 5 — Post-explotación y Escalada de Privilegios

De usuario a root/SYSTEM.

- [Escalada de Privilegios: Linux](../Escalada%20de%20Privilegios/Linux.md)
- [Escalada de Privilegios: Windows](../Escalada%20de%20Privilegios/Windows.md)

> **Objetivo:** control total de la máquina y recoger credenciales para la siguiente.

---

## Módulo 6 — Active Directory

El entorno corporativo. Imprescindible para el OSCP moderno y el mundo laboral.

- [Introducción y Metodología de AD](../Active%20Directory/00%20-%20Introducci%C3%B3n.md)
- [Kerberoasting y AS-REP Roasting](../Active%20Directory/Kerberoasting%20y%20AS-REP%20Roasting.md)
- [BloodHound](../Active%20Directory/BloodHound.md)
- [Movimiento Lateral](../Active%20Directory/Movimiento%20Lateral.md)
- [DCSync y Volcado de Credenciales](../Active%20Directory/DCSync%20y%20Volcado%20de%20Credenciales.md)

> **Objetivo:** de un usuario de dominio a Domain Admin, siguiendo el camino de BloodHound.

---

## Módulo 7 — Nivel Pro y Preparación de Certificaciones

Pivoting, mentalidad avanzada y el salto a las certificaciones.

- [Pivoting y Tunneling](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Pivoting%20y%20Tunneling.md)
- [Mentalidad para Máquinas Difíciles](./Mentalidad%20para%20M%C3%A1quinas%20Dif%C3%ADciles.md)
- [Trucos y Atajos poco Conocidos](./Trucos%20y%20Atajos%20poco%20Conocidos.md)
- [Certificaciones](./Certificaciones.md) — OSCP, CPTS, OSWE y rutas por sector.

> **Objetivo:** resolver máquinas Hard/Insane y estar listo para el examen que elijas.

---

## Práctica transversal (todo el curso)

La teoría sin práctica no sirve. En paralelo a todos los módulos:

- [Laboratorios para Practicar](./Laboratorios%20para%20Practicar.md) — dónde descargar máquinas y practicar gratis.
- [Checklist de Máquina](/hacking-notes/checklist) — marca cada paso al atacar una máquina.
- Walkthroughs completos: [Máquina Linux paso a paso](./Ejemplo%20Resolver%20una%20M%C3%A1quina%20Paso%20a%20Paso.md) y [Dominio Active Directory](./Ejemplo%20Comprometer%20un%20Dominio%20Active%20Directory.md).

---

> **Consejo:** no estudies el curso "de corrido" como un libro. Avanza por los módulos 0-3 y **en cuanto puedas, empieza a hacer máquinas** (Módulo de práctica). La práctica te irá diciendo qué teoría te falta, y volverás a por ella con un propósito claro: así se aprende el triple de rápido.
