# RedShenlong: Hacking Notes (Knowledge Base)

![Format](https://img.shields.io/badge/Format-Markdown%20\(.md\)-blue.svg)
![OS](https://img.shields.io/badge/Environment-Linux%20/%20Arch-lightgrey.svg)
![Status](https://img.shields.io/badge/Status-Active%20/%20Updated-green.svg)

Base de conocimientos personal y cuaderno de campo estructurado en archivos Markdown (`.md`) enfocado en seguridad ofensiva, metodologías de pentesting y administración de sistemas.

Este repositorio actúa como un manual técnico de referencia rápida y una recopilación de *cheat sheets* listas para su uso en entornos controlados de auditoría.

---

# 📂 Estructura del Repositorio

Las notas están organizadas siguiendo el flujo lógico de una auditoría de seguridad y la resolución de laboratorios (CTFs).

```text
.
├── reconocimiento/
├── enumeracion/
├── vulnerabilidades/
├── tecnicas/
├── herramientas/
└── README.md
```

### Contenido principal

* **🔍 Reconocimiento**

  * Descubrimiento de activos.
  * Escaneo de puertos.
  * Enumeración inicial.

* **🌐 Enumeración**

  * SMB / SAMBA
  * RDP
  * SNMP
  * Tomcat
  * IIS
  * Servicios Windows y Linux.

* **💥 Vulnerabilidades**

  * OWASP Top 10
  * SSRF
  * JWT
  * APIs
  * Race Conditions
  * Deserialización Python/YAML
  * Vulnerabilidades lógicas

* **⚙️ Técnicas Fundamentales**

  * Tratamiento de TTY.
  * Estabilización de shells.
  * Netcat.
  * Fuerza bruta.
  * Técnicas de evasión.

---

# 🚀 Uso Local

Al estar escritas en Markdown nativo, las notas pueden consultarse tanto desde la terminal como desde cualquier editor compatible.

## Clonar el repositorio

```bash
git clone https://github.com/RedShenlong/Hacking-Notes.git
cd Hacking-Notes
```

## Buscar contenido rápidamente

Con `grep`:

```bash
grep -rnw "vulnerabilidades/" -e "SQL"
```

O utilizando `ripgrep`:

```bash
rg "SQL"
```

---

# ⚠️ Disclaimer

> **IMPORTANTE**
>
> Todo el contenido de este repositorio tiene fines exclusivamente educativos, de investigación y de uso en laboratorios autorizados (como TryHackMe, Hack The Box o entornos propios).
>
> El uso de estas técnicas contra sistemas sin autorización expresa es ilegal. El autor no se hace responsable del uso indebido de la información contenida en este repositorio.
