# Glosario de Términos

Si empiezas desde cero, estas son las palabras que vas a oír todo el rato. Tenlas a mano hasta que se te queden solas.

---

## Conceptos generales

| Término | Qué significa |
| ------- | ------------- |
| **Pentest / Pentesting** | *Penetration testing*: auditoría autorizada en la que simulas un ataque para encontrar fallos. |
| **Red Team** | Equipo ofensivo que emula a un adversario real, con sigilo y evasión. |
| **Blue Team** | Equipo defensivo: detecta, responde y mitiga ataques. |
| **CTF** | *Capture The Flag*: reto de seguridad donde el objetivo es encontrar "flags". |
| **Flag** | Cadena (p.ej. `flag{...}` o `user.txt`) que prueba que comprometiste algo. |
| **Scope / Alcance** | Lo que tienes **permiso** para atacar. Salirte de él es ilegal. |
| **CVE** | Identificador público de una vulnerabilidad concreta (ej. CVE-2021-4034). |
| **0-day** | Vulnerabilidad sin parche conocido públicamente. |
| **Exploit** | Código o técnica que aprovecha una vulnerabilidad. |
| **Payload** | La "carga" que ejecutas tras explotar (p.ej. una reverse shell). |
| **PoC** | *Proof of Concept*: demostración mínima de que un fallo es explotable. |

---

## Fases de un ataque

| Término | Qué significa |
| ------- | ------------- |
| **Reconocimiento (Recon)** | Recopilar información del objetivo. |
| **Enumeración** | Extraer detalles de cada servicio (usuarios, versiones, rutas). |
| **Footprinting** | Mapear la superficie expuesta de la organización/objetivo. |
| **Foothold / Acceso inicial** | La primera ejecución de comandos o shell en el objetivo. |
| **Escalada de privilegios (Privesc)** | Pasar de usuario normal a root/administrador. |
| **Movimiento lateral** | Saltar de una máquina a otra dentro de la red. |
| **Pivoting** | Usar una máquina comprometida como puente a una red interna. |
| **Post-explotación** | Lo que haces tras entrar: loot, persistencia, limpieza. |
| **Loot** | Datos valiosos que recoges (credenciales, flags, claves). |

---

## Shells y acceso

| Término | Qué significa |
| ------- | ------------- |
| **Shell** | Acceso a una línea de comandos del sistema objetivo. |
| **Reverse shell** | La víctima se conecta a ti para darte la shell. |
| **Bind shell** | La víctima abre un puerto y tú te conectas. |
| **Web shell** | Script en el servidor web que ejecuta comandos vía HTTP. |
| **TTY** | Terminal "de verdad" (con historial, autocompletado, Ctrl+C). |
| **RCE** | *Remote Code Execution*: ejecutar comandos en el servidor remoto. |
| **LPE** | *Local Privilege Escalation*: escalar privilegios ya dentro. |

---

## Web y aplicaciones

| Término | Qué significa |
| ------- | ------------- |
| **OWASP** | Organización que publica el *Top 10* de fallos web/API. |
| **Endpoint** | Una ruta/URL que acepta peticiones (p.ej. `/api/users`). |
| **Payload** | Entrada maliciosa que envías a un parámetro. |
| **WAF** | *Web Application Firewall*: filtra peticiones maliciosas. |
| **Fuzzing** | Probar muchas entradas automáticamente (rutas, parámetros, valores). |
| **Proxy** | Intermediario (Burp) que te deja ver y modificar el tráfico. |

---

## Credenciales y criptografía

| Término | Qué significa |
| ------- | ------------- |
| **Hash** | Huella irreversible de un dato (p.ej. de una contraseña). |
| **Crackear** | Recuperar el texto original de un hash probando candidatos. |
| **Diccionario / Wordlist** | Lista de contraseñas candidatas (ej. `rockyou.txt`). |
| **Fuerza bruta** | Probar muchas combinaciones hasta acertar. |
| **Pass-the-Hash (PtH)** | Autenticarte con el hash NTLM sin conocer la contraseña. |
| **Salt** | Valor aleatorio que se añade a una contraseña antes de hashearla. |

---

## Windows / Active Directory

| Término | Qué significa |
| ------- | ------------- |
| **Active Directory (AD)** | Sistema de Microsoft para gestionar usuarios y equipos de un dominio. |
| **Domain Controller (DC)** | Servidor que gobierna el dominio (guarda todas las credenciales). |
| **Kerberos** | Protocolo de autenticación de AD (tickets). |
| **NTLM** | Protocolo/hash de autenticación de Windows. |
| **Kerberoasting** | Pedir tickets de cuentas de servicio para crackear su contraseña offline. |
| **SYSTEM** | La cuenta con más privilegios en Windows (equivalente a root). |

---

## Herramientas que oirás nombrar

| Herramienta | Para qué |
| ----------- | -------- |
| **Nmap** | Escanear puertos y servicios. |
| **Burp Suite** | Interceptar y atacar tráfico web. |
| **Metasploit** | Framework de explotación con módulos listos. |
| **Hydra / John / Hashcat** | Fuerza bruta y crackeo de contraseñas. |
| **BloodHound** | Mapear caminos de ataque en Active Directory. |
| **linpeas / winpeas** | Enumeración automática para escalar privilegios. |

---

> **Consejo:** no intentes memorizar todo esto de golpe. Vuelve a esta página cuando te cruces con un término que no conozcas. En pocas semanas de practicar máquinas, este vocabulario será tan natural que ni lo pensarás.
