# Protocolo SMTP (Simple Mail Transfer Protocol)

> **Puerto por defecto:** `25/TCP` · `587/TCP` (submission) · `465/TCP` (SMTPS)

SMTP envía correo entre servidores. En auditorías su mayor valor es la **enumeración de usuarios válidos** del sistema (comandos `VRFY`, `EXPN`, `RCPT TO`), que luego usas para fuerza bruta dirigida o ataques de Active Directory. En algunos casos permite enviar correo (phishing) o encadenar con otras vulnerabilidades.

---

## Enumeración

```bash
nmap -p25 -sCV --script "smtp-*" <IP>
nmap -p25 --script smtp-commands,smtp-enum-users,smtp-open-relay <IP>
```

### Banner e inspección manual

```bash
nc <IP> 25
# o
telnet <IP> 25
```

Dentro, el servidor responde con su banner (software y versión → `searchsploit`).

```text
EHLO attacker          # saludo; lista comandos soportados (VRFY, EXPN...)
```

---

## Enumeración de usuarios

Tres comandos permiten comprobar si un usuario existe:

```text
VRFY root              # ¿existe 'root'?  -> 250/252 = sí, 550 = no
EXPN admin             # expandir alias/lista
RCPT TO:<admin@dominio>   # (tras MAIL FROM) acepta -> existe
```

Automatizado con **smtp-user-enum**:

```bash
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/Names/names.txt -t <IP>
smtp-user-enum -M RCPT -U usuarios.txt -t <IP>       # si VRFY está deshabilitado
```

> **Consejo:** si `VRFY`/`EXPN` están deshabilitados, usa el método **RCPT TO** (`-M RCPT`), que suele seguir funcionando. La lista de usuarios válidos que saques aquí es oro para fuerza bruta en SSH/IMAP o para AS-REP Roasting en AD.

---

## Enviar un correo manualmente (open relay / phishing)

Comprueba si el servidor es un **open relay** (reenvía correo de cualquiera):

```text
HELO x
MAIL FROM:<jefe@empresa.com>
RCPT TO:<victima@empresa.com>
DATA
Subject: Actualización de credenciales

Texto del correo.
.
QUIT
```

Un open relay permite suplantar remitentes (phishing interno). Con `swaks` es más cómodo:

```bash
swaks --to victima@empresa.com --from jefe@empresa.com \
  --server <IP> --body "mensaje" --header "Subject: Importante"
```

---

## Fuerza bruta

```bash
hydra -L usuarios.txt -P /usr/share/wordlists/rockyou.txt smtp://<IP>
```

---

## POP3 / IMAP (correo relacionado)

Si además ves `110` (POP3) o `143` (IMAP), con credenciales puedes **leer el buzón**:

```bash
# POP3 manual
nc <IP> 110
USER <usuario>
PASS <pass>
LIST
RETR 1
```

Los correos suelen contener credenciales, enlaces internos y tokens de recuperación.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `smtp-user-enum` | Enumeración de usuarios (VRFY/EXPN/RCPT). |
| `swaks` | Envío de correo flexible (pruebas de relay/phishing). |
| `nc` / `telnet` | Interacción manual con el servicio. |
| `nmap` (`smtp-*`) | Enumeración y detección de open relay. |

---

> **Consejo:** el valor principal de SMTP en un CTF/pentest es **sacar usuarios válidos**. Enumera con `smtp-user-enum` (prueba VRFY y, si falla, RCPT), y lleva esa lista a tus ataques de credenciales. Revisa también POP3/IMAP: con un usuario y contraseña, el buzón suele contener el siguiente paso.
