# Protocolo SSH (Secure Shell)

> **Puerto por defecto:** `22/TCP`

SSH permite administrar un sistema de forma remota y cifrada. En un pentest rara vez es el **punto de entrada** (está bien protegido); casi siempre es la **salida**: cuando consigues unas credenciales o una clave privada en otro servicio, SSH es por donde entras de forma estable.

---

## Enumeración

```bash
nmap -p22 -sCV <IP>
# Métodos de autenticación admitidos (¿password? ¿publickey?)
nmap -p22 --script ssh-auth-methods --script-args="ssh.user=root" <IP>
```

La **versión** de OpenSSH puede delatar el sistema y, a veces, un CVE:

| Versión / caso | Nota |
| -------------- | ---- |
| OpenSSH < 7.7  | Enumeración de usuarios (CVE-2018-15473): distinguir usuarios válidos. |
| Banner con `Ubuntu`/`Debian` | Ayuda a identificar la distribución y versión. |

---

## Acceso con credenciales

```bash
ssh <usuario>@<IP>
ssh <usuario>@<IP> -p <puerto>
```

Si el servidor es antiguo y el cliente rechaza sus algoritmos:

```bash
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa <usuario>@<IP>
```

---

## Acceso con clave privada (lo más habitual)

Si encuentras una `id_rsa` (en un FTP, en un share, en la web, en un backup):

```bash
chmod 600 id_rsa
ssh -i id_rsa <usuario>@<IP>
```

### Clave privada protegida con passphrase

Crackéala offline:

```bash
ssh2john id_rsa > hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

> **Consejo:** cuando consigas acceso a un sistema, revisa siempre `~/.ssh/` de cada usuario: `id_rsa` (clave privada para entrar como ese usuario) y `authorized_keys` (si puedes escribir aquí, añades tu propia clave y entras sin contraseña).

---

## Fuerza bruta

```bash
hydra -l <usuario> -P /usr/share/wordlists/rockyou.txt ssh://<IP>
netexec ssh <IP> -u usuarios.txt -p passwords.txt
```

> **Nota:** SSH es lento ante fuerza bruta y suele tener bloqueos (fail2ban). Úsalo solo con un usuario válido y una wordlist pequeña/dirigida. Prioriza siempre credenciales encontradas por enumeración.

---

## Técnicas útiles con SSH ya dentro

### Túneles (pivoting)

SSH es una navaja suiza para pivotar hacia la red interna:

```bash
# Local port forwarding: accede a un servicio interno de la víctima
ssh -L 8080:127.0.0.1:8080 <usuario>@<IP>

# Dynamic (proxy SOCKS) para usar con proxychains
ssh -D 1080 <usuario>@<IP>
```

Más detalle en la nota de **Pivoting y Tunneling**.

### Ejecutar un comando puntual sin shell interactiva

```bash
ssh <usuario>@<IP> "id; hostname; sudo -l"
```

---

## Abuso de claves y configuraciones

- **`authorized_keys` escribible** → añade tu clave pública y entras como ese usuario.
- **`~/.ssh/config` / known_hosts** → revela otros hosts a los que pivotar.
- **ForwardAgent / agente SSH reenviado** → si el usuario reenvía su agente, puedes saltar a otras máquinas con sus claves.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `ssh` | Cliente y túneles. |
| `ssh2john` + `john` | Crackear passphrase de clave privada. |
| `hydra` / `netexec` | Fuerza bruta (con cautela). |
| `sshpass` | Automatizar login con contraseña en scripts. |

---

> **Consejo:** no pienses en SSH como un objetivo a romper, sino como **la llave que abres con lo que encuentras en otros sitios**. La secuencia típica de una máquina es: vulneras la web → encuentras una credencial o una `id_rsa` → entras por SSH de forma estable → escalas privilegios. Cuando tengas usuario+contraseña de cualquier servicio, **pruébalos en SSH**: la reutilización es constante.
