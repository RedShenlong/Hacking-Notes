# Protocolo RDP (Remote Desktop Protocol)

Permite establecer una conexión gráfica remota con un sistema Windows. Una de las herramientas más utilizadas en entornos Linux para conectarse a servidores RDP es **xfreerdp**.

## Conexión básica

```bash
xfreerdp /u:<usuario> /p:<contraseña> /v:<IP>:3389
```

### Parámetros

| Parámetro | Descripción |
|-----------|-------------|
| `/u:` | Nombre de usuario. |
| `/p:` | Contraseña del usuario. |
| `/v:` | Dirección IP o nombre del host remoto. |
| `:3389` | Puerto del servicio RDP (por defecto `3389`). |

## Ejemplo

```bash
xfreerdp /u:administrator /p:P@ssw0rd /v:192.168.1.100:3389
```

## Opciones útiles

```bash
# Pantalla completa
xfreerdp /u:<usuario> /p:<contraseña> /v:<IP> /f

# Compartir el portapapeles
xfreerdp /u:<usuario> /p:<contraseña> /v:<IP> /clipboard

# Ignorar errores de certificados
xfreerdp /u:<usuario> /p:<contraseña> /v:<IP> /cert:ignore

# Ajustar resolución
xfreerdp /u:<usuario> /p:<contraseña> /v:<IP> /size:1920x1080

# Compartir una carpeta local
xfreerdp /u:<usuario> /p:<contraseña> /v:<IP> /drive:share,/home/kali/Compartido
```

> **Nota:** En ejercicios de pentesting o laboratorios es habitual utilizar `/cert:ignore` cuando el servidor presenta un certificado autofirmado.
