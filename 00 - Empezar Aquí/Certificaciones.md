# Certificaciones de Ciberseguridad (por sector y nivel)

Una certificación no te hace hacker, pero **valida tu nivel** ante una empresa y te obliga a estudiar de forma estructurada. Aquí tienes qué certificación tiene sentido según dónde quieras trabajar y en qué momento de tu carrera estés.

> **Aviso importante:** elige la certificación por **lo que vas a aprender y por el sector al que apuntas**, no por coleccionar siglas. Una OSCP bien trabajada vale más que diez certificados teóricos de opción múltiple.

---

## Mapa rápido: ¿por dónde empiezo?

```text
PRINCIPIANTE          OFENSIVA JUNIOR        OFENSIVA SENIOR / ESPECIALISTA
────────────          ───────────────        ──────────────────────────────
eJPT  ──►  CPTS / PNPT  ──►  OSCP  ──►  OSEP (Red Team) / OSWE (Web) / CRTO (AD)
                              │
Fundamentos (CompTIA        Web: eWPT / BSCP (PortSwigger)
Security+) opcional         Cloud: CARTP / cert. específicas del proveedor
```

---

## Nivel entrada (primera certificación)

### eJPT (eLearnSecurity Junior Penetration Tester)

- **Para quién:** tu **primer** certificado práctico. Demuestra que sabes hacer una auditoría básica de principio a fin.
- **Formato:** examen práctico con preguntas, en laboratorio. Sin informe.
- **Dificultad:** baja. Ideal para coger confianza.
- **Por qué sí:** barato, práctico, buena rampa hacia el OSCP.

### CompTIA Security+

- **Para quién:** si buscas un **empleo generalista** de seguridad (SOC, cumplimiento) o te lo pide una oferta. Muy valorada en entornos corporativos y gobierno.
- **Formato:** teórica (test). No es de hacking ofensivo.
- **Por qué:** abre puertas de RR.HH., pero **no** te enseña a comprometer máquinas.

---

## Ofensiva — nivel intermedio (la liga del OSCP)

### OSCP (Offensive Security Certified Professional) ⭐

**La certificación de referencia en pentesting.** Si tu objetivo es ser pentester, esta es "la" meta a medio plazo.

- **Qué cubre:** enumeración, explotación web, escalada de privilegios Linux y Windows, y **Active Directory** (desde la actualización del temario).
- **Formato:** examen **práctico de 24 horas** comprometiendo máquinas + **24 horas para entregar un informe** profesional. Hay puntos por máquina y un umbral para aprobar.
- **Dificultad:** alta y exigente, pero muy alcanzable con preparación. Su lema es *"Try Harder"*.
- **Cómo prepararla:**
  1. Domina la [Metodología de Pentesting](./Metodolog%C3%ADa%20de%20Pentesting.md) de este repo hasta hacerla en automático.
  2. Practica mucha máquina: DockerLabs, HTB (lista "OSCP-like"), ProvingGrounds.
  3. Aprende a **escribir informes** desde el principio: en el examen, un informe malo suspende aunque hayas comprometido las máquinas.
  4. Interioriza la escalada de privilegios Linux **y** Windows, y lo básico de AD.
- **Importante:** **no se permite Metasploit** (salvo en una sola máquina) ni herramientas automáticas de explotación. Debes saber hacerlo "a mano".

### CPTS (HTB Certified Penetration Testing Specialist)

- **Para quién:** alternativa moderna y muy completa al OSCP, de Hack The Box. Más larga y con fuerte componente de AD y pivoting.
- **Formato:** examen práctico extenso con informe.
- **Por qué:** excelente relación calidad/precio y contenido muy actual. Cada vez más reconocida.

### PNPT (Practical Network Penetration Tester)

- **Para quién:** enfocado en pentesting **realista**, con fuerte peso de Active Directory y **presentación del informe en directo**.
- **Por qué:** muy práctica y con un enfoque cercano a una auditoría real.

---

## Ofensiva — nivel avanzado / especialista

### OSWE (Offensive Security Web Expert)

- **Sector:** **seguridad web avanzada / whitebox**. Explotación revisando código fuente.
- **Para quién:** si te apasiona la web y el bug hunting a nivel de código.

### OSEP (Offensive Security Experienced Penetration Tester)

- **Sector:** **Red Team / evasión**. Bypass de antivirus/EDR, técnicas avanzadas de AD, pivoting.
- **Para quién:** el paso natural tras el OSCP si vas a Red Team.

### CRTO (Certified Red Team Operator)

- **Sector:** **Red Team con Cobalt Strike**, muy centrada en Active Directory y operaciones reales.
- **Por qué:** una de las mejores certificaciones de Red Team, muy práctica.

### BSCP (Burp Suite Certified Practitioner)

- **Sector:** **web**. De los creadores de Burp Suite y la Web Security Academy.
- **Por qué:** barata, 100 % práctica y respaldada por el mejor material web gratuito que existe (PortSwigger Academy).

---

## Por sector (resumen)

| Si quieres dedicarte a… | Ruta de certificaciones recomendada |
| ----------------------- | ----------------------------------- |
| **Pentesting generalista** | eJPT → OSCP / CPTS |
| **Red Team**            | OSCP → OSEP / CRTO |
| **Seguridad web / Bug Bounty** | PortSwigger Academy → BSCP → OSWE |
| **Active Directory / interno** | CPTS / PNPT → CRTO |
| **Cloud**               | Certificación del proveedor (AWS/Azure) + CARTP u ofensiva cloud |
| **Empleo corporativo / SOC** | CompTIA Security+ → certs azules (Blue Team) |
| **Exploiting / binario** | OSED (del cursus OSCE³ de Offensive Security) |

---

## Preguntas frecuentes

**¿Necesito una certificación para empezar a aprender?**
No. Aprende y practica gratis (este repo, PortSwigger, DockerLabs, TryHackMe). La certificación llega cuando ya sabes hacer las cosas; sirve para **demostrarlo**, no para aprenderlas de cero.

**¿Cuál es "la mejor"?**
No existe "la mejor" en abstracto. Para entrar en pentesting, el camino más reconocido hoy es **eJPT → OSCP**. A partir de ahí, especializas.

**¿Las certificaciones caducan?**
Algunas sí (se renuevan cada ciertos años o con créditos de formación continua). Comprueba la política vigente de cada una antes de pagar.

---

> **Consejo:** antes de gastar dinero en un examen, mide tu nivel resolviendo máquinas de dificultad equivalente. Si resuelves máquinas "OSCP-like" de forma consistente y sin mirar writeups, estás listo para el OSCP. La certificación debería **confirmar** lo que ya sabes, no ser el sitio donde lo aprendes por primera vez.
