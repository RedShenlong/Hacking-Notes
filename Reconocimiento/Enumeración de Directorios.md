# Enumeración de Directorios Web

La enumeración de directorios permite descubrir archivos, carpetas, paneles de administración y recursos ocultos de una aplicación web. Es una de las primeras técnicas que deben realizarse durante una auditoría web.

---

# Gobuster

Una de las herramientas más utilizadas por su velocidad y simplicidad.

```bash
gobuster dir \
-u http://<IP>/ \
-t 100 \
-w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt \
-x txt,php,py,sh
```

---

# Wfuzz

Permite realizar fuzzing utilizando el marcador `FUZZ`.

```bash
wfuzz \
-c \
-t 200 \
--hc=404 \
--hl=<LINEAS> \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
https://<IP>/FUZZ
```

---

# FFUF

Actualmente una de las herramientas más rápidas para enumeración web.

```bash
ffuf \
-c \
-t 200 \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-u https://<IP>/FUZZ
```

Buscar archivos PHP:

```bash
ffuf \
-c \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
-u http://<IP>/FUZZ.php
```

---

# Dirsearch

Permite buscar directorios y archivos con múltiples opciones de filtrado.

```bash
dirsearch \
-u http://<IP>/ \
-w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

---

# Dirb

Herramienta clásica incluida en la mayoría de distribuciones orientadas a pentesting.

```bash
dirb http://<IP>
```

---

# Leyenda

| Parámetro          | Descripción                                                                                     |
| ------------------ | ----------------------------------------------------------------------------------------------- |
| `-u <URL>`         | URL objetivo (`http://<IP>/` o `https://<IP>/FUZZ`).                                            |
| `-w <wordlist>`    | Ruta al diccionario utilizado durante la enumeración.                                           |
| `-t <n>`           | Número de hilos concurrentes.                                                                   |
| `-c`               | Activa la salida con colores.                                                                   |
| `-x <extensiones>` | Busca archivos con extensiones específicas (`php`, `txt`, `jsp`, etc.).                         |
| `FUZZ`             | Marcador que será sustituido por cada palabra del diccionario.                                  |
| `--hc=404`         | Oculta las respuestas HTTP 404.                                                                 |
| `--hl=<L>`         | Oculta respuestas con un número determinado de líneas. Muy útil para eliminar falsos positivos. |

---

# Wordlists recomendadas

```text
/usr/share/seclists/Discovery/Web-Content/
```

Algunas de las más utilizadas:

```text
directory-list-2.3-small.txt
directory-list-2.3-medium.txt
directory-list-2.3-big.txt
raft-small-directories.txt
raft-medium-directories.txt
raft-large-directories.txt
common.txt
```

---

# Herramientas útiles

| Herramienta | Descripción                                                 |
| ----------- | ----------------------------------------------------------- |
| `Gobuster`  | Enumeración rápida de directorios y archivos.               |
| `FFUF`      | Fuzzer web muy rápido y flexible.                           |
| `Wfuzz`     | Fuzzing avanzado sobre parámetros, directorios y cabeceras. |
| `Dirsearch` | Enumeración web con numerosos filtros y opciones.           |
| `Dirb`      | Herramienta clásica para descubrimiento de contenido web.   |

---

> **Consejo:** Si la aplicación devuelve siempre **HTTP 200**, utiliza filtros como `--hl` (líneas), `--fs` (tamaño) o `--fw` (palabras) para eliminar falsos positivos. En aplicaciones modernas suele ser más fiable filtrar por longitud de la respuesta que únicamente por el código HTTP.
