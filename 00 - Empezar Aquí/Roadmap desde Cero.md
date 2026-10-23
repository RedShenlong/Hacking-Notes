# Roadmap: de 0 a Hacker

Esta es la ruta recomendada si empiezas desde cero. No necesitas estudios previos de informática, pero sí **constancia**. El orden importa: cada bloque se apoya en el anterior. No corras a lanzar exploits sin entender la red por debajo.

> **Mentalidad:** hackear es, sobre todo, **entender cómo funcionan las cosas mejor que quien las construyó**. Antes de romper un sistema tienes que saber cómo está hecho. Por eso las bases (redes, Linux, programación) no son opcionales.

---

## Visión general

```pipeline
Fundamentos (Linux · Redes · Windows) -> Scripting (Bash / Python) -> Web + Máquinas -> CTFs -> Certificación (OSCP) -> Especialización
```

---

## Nivel 0 — Fundamentos (1–3 meses)

No puedes auditar lo que no entiendes.

### Linux

Es tu sistema operativo de trabajo. Instala **Kali Linux** o, si usas Arch (como este repo), **BlackArch** sobre tu Arch.

Debes dominar:

- Navegación y ficheros: `cd`, `ls`, `cat`, `find`, `grep`, `less`.
- Permisos: usuarios, grupos, `chmod`, `chown`, SUID.
- Procesos y servicios: `ps`, `systemctl`, `kill`.
- Redes desde la terminal: `ip a`, `ss -tulpn`, `curl`, `wget`.
- Edición en terminal: `nano` y lo básico de `vim`.

> **Recurso:** OverTheWire **Bandit** (https://overthewire.org/wargames/bandit/) es el mejor punto de partida para la terminal, nivel a nivel.

### Redes

Sin esto, Nmap y las reverse shells son magia negra. Entiende:

- Modelo TCP/IP, qué es una IP, una máscara, una puerta de enlace.
- Puertos y el concepto de servicio escuchando en un puerto.
- Diferencia TCP vs UDP.
- Qué es DNS, DHCP, NAT.
- El recorrido de una petición HTTP (cliente → servidor → respuesta).

### Windows (básico)

Muchas máquinas y casi todo el entorno corporativo es Windows/Active Directory.

- CMD y PowerShell básicos.
- Qué es un dominio, un usuario local vs. de dominio.
- Concepto de Active Directory (ya lo profundizarás más adelante).

---

## Nivel 1 — Scripting (en paralelo, continuo)

No necesitas ser programador, pero sí **leer código** y **automatizar**.

- **Bash:** automatizar enumeración, bucles, pipes. (Lo usas desde el día 1.)
- **Python:** modificar exploits, escribir pequeñas herramientas, parsear datos. Es *el* lenguaje del pentester.
- **Lectura, no dominio, de:** PHP, JavaScript, PowerShell. Necesitas entenderlos para auditarlos, no para escribirlos desde cero.

> **Objetivo realista:** ser capaz de coger un exploit de Exploit-DB en Python, entender qué hace y cambiarle la IP, el puerto y el payload.

---

## Nivel 2 — Hacking web (2–4 meses)

La web es el vector de entrada número uno. Aprende el **OWASP Top 10** a fondo:

- SQL Injection
- XSS (Cross-Site Scripting)
- LFI / RFI (Local/Remote File Inclusion)
- Command Injection
- SSRF, SSTI
- File Upload, IDOR, deserialización

> **Práctica:** **PortSwigger Web Security Academy** (https://portswigger.net/web-security) es gratis, oficial de los creadores de Burp Suite, y es el mejor recurso que existe para web. Haz todos los labs.

Toda la teoría y explotación está en [`Vulnerabilidades/Web/`](../Vulnerabilidades/Web/) y [`Vulnerabilidades/APIs/`](../Vulnerabilidades/APIs/).

---

## Nivel 3 — Máquinas y CTFs (continuo, el núcleo)

Aquí es donde de verdad aprendes a comprometer sistemas de principio a fin. Aplica la [Metodología de Pentesting](./Metodolog%C3%ADa%20de%20Pentesting.md).

Orden recomendado de plataformas:

1. **DockerLabs** (https://dockerlabs.es) — máquinas en español, gratis, se levantan en local con Docker. **Perfecto para empezar.**
2. **TryHackMe** — rutas guiadas, muy didáctico para principiantes.
3. **Hack The Box** — máquinas más realistas; usa las "retiradas" con writeups al principio.
4. **VulnHub** — máquinas descargables en OVA.

> Catálogo y cómo montarlas en [Laboratorios para Practicar](./Laboratorios%20para%20Practicar.md).

**Cómo practicar bien:** intenta la máquina tú solo hasta atascarte de verdad (30–60 min sin avanzar). Entonces mira **un paso** del writeup, no la solución entera. Retoma. Al terminar, escribe **tu propio writeup** — enseñar es cuando se consolida.

---

## Nivel 4 — Escalada de privilegios y Active Directory (3–6 meses)

- **Privesc Linux:** SUID, sudo, cron, capabilities, GTFOBins. Ver [`Escalada de Privilegios/Linux.md`](../Escalada%20de%20Privilegios/Linux.md).
- **Privesc Windows:** tokens, servicios, Potato attacks. Ver [`Escalada de Privilegios/Windows.md`](../Escalada%20de%20Privilegios/Windows.md).
- **Active Directory:** Kerberos, enumeración con BloodHound, Kerberoasting, movimiento lateral. Es **imprescindible** para el mundo laboral y para el OSCP moderno.

---

## Nivel 5 — Certificación (cuando domines 1–4)

Una certificación valida tu nivel y abre la puerta laboral. La más reconocida para empezar en ofensiva es la **OSCP**.

> Guía completa en [Certificaciones](./Certificaciones.md).

---

## Nivel 6 — Especialización

Elige según el sector que te guste:

| Camino            | De qué va |
| ----------------- | --------- |
| **Red Team**      | Simular adversarios reales, evasión, C2, sigilo. |
| **Web / API / Bug Bounty** | Encontrar fallos en aplicaciones, ganar recompensas. |
| **Active Directory / Interno** | Pentesting corporativo, dominios Windows. |
| **Cloud** (AWS/Azure/GCP) | Auditoría de infraestructura cloud. |
| **Mobile**        | Android / iOS. |
| **Hardware / IoT / Radio** | Dispositivos físicos, firmware, RF. |
| **Exploiting / Binary** | Desarrollo de exploits, pwn, ingeniería inversa. |

---

## Hábitos que marcan la diferencia

- **Toma notas siempre** (este repo es un ejemplo). Lo que no anotas, lo olvidas.
- **Escribe writeups** de cada máquina que resuelvas.
- **Lee writeups** de otros: aprenderás técnicas que no se te habrían ocurrido.
- **Sigue la actualidad:** cuentas de X/Twitter de investigadores, boletines de CVEs.
- **No memorices comandos, entiende el porqué.** La chuleta (este repo) es para recordar la sintaxis, no para sustituir la comprensión.

---

> **Consejo:** no intentes aprenderlo todo a la vez ni en orden perfecto. Avanza por el nivel 0–2, y en cuanto puedas empieza a hacer máquinas (nivel 3): la práctica te irá señalando qué teoría te falta, y volverás a por ella con un propósito claro. Se aprende mucho más rápido cuando sabes **para qué** necesitas algo.
