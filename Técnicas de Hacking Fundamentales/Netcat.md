# Netcat (nc)

**Netcat** es una herramienta muy utilizada durante auditorías de seguridad para establecer conexiones TCP/UDP, escuchar en puertos, transferir archivos y verificar la conectividad entre dos equipos.

---

# Listener

Abrir un puerto en escucha para recibir una conexión entrante.

```bash id="nzz13j"
nc -nlvp 443
```

**Parámetros:**

| Parámetro | Descripción                 |
| --------- | --------------------------- |
| `-n`      | No realiza resolución DNS.  |
| `-l`      | Modo escucha (Listen).      |
| `-v`      | Salida detallada (Verbose). |
| `-p`      | Puerto donde escuchar.      |

---

# Comprobar conectividad

Verificar si un puerto está abierto:

```bash id="j3usm3"
nc -nv <IP> <PUERTO>
```

Ejemplo:

```bash id="jlwm5z"
nc -nv 192.168.1.10 80
```

---

# Transferencia de archivos

## Enviar un archivo

Máquina receptora:

```bash id="g6lffo"
nc -nlvp 4444 > archivo.txt
```

Máquina emisora:

```bash id="fj60fj"
nc <IP> 4444 < archivo.txt
```

---

# Estabilizar una TTY (Linux)

Si se obtiene una shell limitada, puede mejorarse con:

```bash id="3e1sll"
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Después:

```text id="ib6vb5"
CTRL + Z
```

Y en la máquina atacante:

```bash id="cbb5tw"
stty raw -echo; fg
```

Finalmente:

```bash id="zbgzgy"
export TERM=xterm
```

---

# Reverse Shell Generator

Una forma cómoda de generar comandos adaptados al sistema operativo y lenguaje disponible es utilizar:

```text id="pqglnz"
https://www.revshells.com/
```

Permite generar comandos para:

* Bash
* PowerShell
* PHP
* Python
* Perl
* Java
* Netcat
* Socat
* Ruby
* Awk
* Tcl
* Groovy
* Y muchos más.

---

# Herramientas relacionadas

| Herramienta   | Descripción                                                      |
| ------------- | ---------------------------------------------------------------- |
| `nc` (Netcat) | Conexiones TCP/UDP, listeners y transferencia de archivos.       |
| `ncat`        | Implementación de Netcat incluida con Nmap.                      |
| `socat`       | Alternativa más potente para redirección de conexiones y shells. |
| `python3`     | Útil para estabilizar una TTY interactiva.                       |

---

> **Consejo:** El puerto `443` suele utilizarse con frecuencia durante laboratorios porque normalmente está permitido por los firewalls. No obstante, puedes utilizar cualquier puerto que esté permitido entre la máquina atacante y el objetivo.
