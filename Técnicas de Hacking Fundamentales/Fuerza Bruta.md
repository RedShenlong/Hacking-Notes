# Hydra

* Contraseña como parámetro desconocido

```
hydra -l <usuario> -P /usr/share/wordlists/rockyou.txt <variable>
```

* Usuario como parámetro desconocido

```
hydra -L /usr/share/wordlists/metasploit/unix_users.txt -p <contraseña> <variable>
```
---
#### Leyenda
+ `<variable>`
	+ **ssh** = `ssh://<IP>` *//intrusión* -> `ssh <usuario><@<IP>`
	+  **ftp** = `ftp://<IP>` *//intrusión* -> `ftp <IP>`
	+  **mysql** = `mysql://<IP>` 
	- **Login web** = `<IP> http-get /login.php -s <puerto> -f`
	- **Login web (POST)** = `<IP> http-post-form "/login.php:user=^USER^&pass=^PASS^:Login Failed"`


---
# Metasploit

+ **ssh** = `search ssh_login`
+ **ftp** = `search ftp_login`

---
# John The Ripper

+ **archivos.zip**
	+ `zip2john <archivo.zip> > hash`
	+ `john --wordlist=/usr/share/wordlists/rockyou.txt hash`
+ **base de datos .kdbx** 
	+ `keepass2john <archivo.kdbx> > hash`
	+ `john --wordlist = /usr/share/wordlists/rockyou.txt hash`
Sino encuentra la contraseña, utiliza la herramienta *hash-identifier* para saber el formato del hash
* `john --format=<formato del hash> --wordlist=/usr/share/wordlists/rockyou.txt hash`







/diningRoom/
/teaRoom/
/artRoom/
/barRoom/
/diningRoom2F/
/tigerStatusRoom/
/galleryRoom/
/studyRoom/
/armorRoom/
/attic/


helmet_key{458493193501d2b94bbab2e727f8db4b}