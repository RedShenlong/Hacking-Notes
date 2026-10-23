# Laboratorios para Practicar

Hackear se aprende hackeando (en entornos autorizados). Aquí tienes dónde practicar **legal y gratis**, ordenado de más fácil a más realista, y cómo levantar las máquinas en tu propio equipo.

> **Solo ataca máquinas diseñadas para ello o que tú mismo hayas montado.** Todas las plataformas de esta página te dan permiso explícito para atacar sus objetivos. Atacar sistemas de terceros sin autorización es delito.

---

## Plataformas online

| Plataforma | Nivel | Idioma | Coste | Notas |
| ---------- | ----- | ------ | ----- | ----- |
| **DockerLabs** (dockerlabs.es) | Inicial–medio | Español | Gratis | Máquinas que levantas en local con Docker. Ideal para empezar. |
| **TryHackMe** | Inicial | Inglés | Freemium | Rutas guiadas, muy didáctico. |
| **Hack The Box** | Medio–alto | Inglés | Freemium | Realista. Usa máquinas retiradas con writeups al principio. |
| **VulnHub** | Variado | Inglés | Gratis | Máquinas descargables (OVA) para VirtualBox/VMware. |
| **PortSwigger Web Security Academy** | Inicial–alto | Inglés | Gratis | El mejor recurso de **web** que existe. |
| **OverTheWire** | Inicial | Inglés | Gratis | Wargames de terminal/Linux (empieza por *Bandit*). |
| **ProvingGrounds** (Offensive Security) | Medio–alto | Inglés | De pago | Muy "OSCP-like". Bueno para preparar el examen. |
| **Root-Me** | Variado | Multi (ES) | Freemium | Gran variedad de retos por categorías. |

---

## DockerLabs (recomendado para empezar)

Máquinas vulnerables en español que se levantan en tu propio equipo con Docker. Perfectas para aplicar la [metodología](./Metodolog%C3%ADa%20de%20Pentesting.md).

### Flujo típico

```bash
# 1. Descarga el .zip de la máquina desde dockerlabs.es y descomprímelo
unzip NombreMaquina.zip -d NombreMaquina
cd NombreMaquina

# 2. Levanta la máquina (cada lab trae su script)
sudo bash auto_deploy.sh <fichero.tar>   # el nombre exacto lo indica cada máquina
```

La máquina queda escuchando en una IP local (normalmente del rango `172.17.0.0/16` de Docker). Descúbrela:

```bash
# Ver la IP del contenedor recién levantado
sudo docker ps
sudo docker inspect <id_contenedor> | grep IPAddress
```

A partir de ahí, atacas esa IP como cualquier máquina: empiezas por el [escaneo de puertos](../Reconocimiento/Escaneo%20de%20Puertos.md).

> **Consejo:** si no sabes la IP, un barrido rápido de tu red Docker también sirve: ver [`Encontrar IPs en mi Red.md`](../Reconocimiento/Encontrar%20IPs%20en%20mi%20Red.md).

---

## Máquinas de VulnHub (OVA en local)

1. Descarga el fichero `.ova` de vulnhub.com.
2. Impórtalo en **VirtualBox** o **VMware** (`Archivo → Importar servicio virtualizado`).
3. Configura la red en modo **Host-Only** o **NAT Network** para aislar el lab de internet.
4. Arranca la víctima y tu Kali en la **misma red**, y descubre la IP de la víctima con `arp-scan` o `nmap -sn`.

> Nunca pongas una máquina vulnerable en modo "Bridge" expuesta a tu red real o a internet.

---

## Cómo sacarle el máximo a cada máquina

1. **Intenta en solitario** hasta atascarte de verdad (30–60 min sin progreso).
2. Cuando te atasques, **relee la [chuleta de "¿Y ahora qué?"](./Metodolog%C3%ADa%20de%20Pentesting.md#chuleta-mental-y-ahora-qu%C3%A9)** antes de ir a un writeup.
3. Si sigues bloqueado, mira **un solo paso** del writeup, no la solución completa.
4. Al terminar, **escribe tu propio writeup**: qué puertos, qué te llevó al foothold, cómo escalaste. Es donde se consolida el aprendizaje.
5. Apunta en este repo cualquier técnica nueva que hayas usado.

---

## Rutas sugeridas por objetivo

| Objetivo | Plataformas |
| -------- | ----------- |
| Empezar de cero | OverTheWire Bandit → DockerLabs → TryHackMe |
| Dominar web | PortSwigger Academy (todos los labs) |
| Preparar OSCP | DockerLabs + HTB (lista OSCP-like) + ProvingGrounds |
| Active Directory | Máquinas/rutas de AD en HTB y TryHackMe |

---

> **Consejo:** mantén un directorio por máquina con tus notas y la salida de `nmap`. Reutilizarás esas notas una y otra vez, y el hábito de documentar es exactamente lo que necesitarás el día del examen OSCP.

---

## Enlaces directos (descarga y acceso)

| Plataforma | Qué es | Enlace |
| ---------- | ------ | ------ |
| DockerLabs | Máquinas en español, se levantan en local con Docker | https://dockerlabs.es |
| VulnHub | Máquinas descargables en formato OVA | https://www.vulnhub.com |
| TryHackMe | Salas guiadas, ideal para empezar | https://tryhackme.com |
| Hack The Box | Máquinas realistas (online) | https://www.hackthebox.com |
| PortSwigger Web Security Academy | Labs de web gratis (lo mejor para web) | https://portswigger.net/web-security |
| picoCTF | CTF gratuito, muy didáctico | https://picoctf.org |
| OverTheWire | Wargames de terminal/Linux (empieza por Bandit) | https://overthewire.org/wargames |
| Root-Me | Retos por categorías (hay en español) | https://www.root-me.org |
| DVWA | App web vulnerable (Docker) para practicar vulns | https://github.com/digininja/DVWA |
| OWASP Juice Shop | App moderna vulnerable (Docker) | https://github.com/juice-shop/juice-shop |
| CyberDefenders | Retos Blue Team / forense (pcaps) | https://cyberdefenders.org |

### Montar las apps web vulnerables (rápido)

```bash
# DVWA
docker run --rm -it -p 80:80 vulnerables/web-dvwa
# OWASP Juice Shop
docker run --rm -p 3000:3000 bkimminich/juice-shop
```

---

## Qué practicar según la técnica

Mapa rápido para ir directo a practicar lo que acabas de estudiar:

| Tema / vulnerabilidad | Dónde practicarlo |
| --------------------- | ----------------- |
| OWASP Web (SQLi, XSS, LFI, upload…) | PortSwigger Academy + DVWA (cada [nota de Web](../Vulnerabilidades/Web/) trae su writeup) |
| APIs (BOLA, JWT, Mass Assignment…) | crAPI, Juice Shop, VAmPI, DVGA (ver [Laboratorios de APIs](../Vulnerabilidades/APIs/Vulnerabilidades/Laboratorios/)) |
| Enumeración de protocolos / servicios | DockerLabs, VulnHub (*Basic Pentesting*, *DC-1*) |
| Escalada de privilegios Linux | TryHackMe *Linux PrivEsc*, DockerLabs, [GTFOBins](https://gtfobins.github.io/) |
| Escalada de privilegios Windows | TryHackMe *Windows PrivEsc*, HTB retiradas de Windows |
| Active Directory | TryHackMe *Attacktive Directory*, HTB rutas de AD, GOAD (lab local) |
| Análisis forense / tráfico | CyberDefenders, Malware-Traffic-Analysis (pcaps) |
| Fundamentos de terminal/Linux | OverTheWire *Bandit* |

> **GOAD** (*Game of Active Directory*) es un laboratorio de AD completo y gratuito que puedes montar en local para practicar todo el Módulo 6: https://github.com/Orange-Cyberdefense/GOAD

---

> **Consejo:** combina los dos tipos de práctica. Los **labs de una sola vulnerabilidad** (PortSwigger, DVWA) te enseñan la técnica aislada y a fondo; las **máquinas completas** (DockerLabs, VulnHub, HTB) te enseñan a *encadenarlas* bajo presión, que es lo que de verdad evalúa el OSCP. Empieza por el lab aislado y, cuando lo domines, llévalo a una máquina entera.
