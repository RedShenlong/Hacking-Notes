*puerto udp*

* Buscamos una calve de comunidad con:
	```
	onesixtyone -c /usr/share/wordlists/rockyou.txt <IP>
	```
* Accedemos con:
	```
	snmpwalk -v 2c -c <clave de comunidad> <IP>
	```
Con esto accederemos a un montón de información como usuarios, contraseñas, dominios etc.
