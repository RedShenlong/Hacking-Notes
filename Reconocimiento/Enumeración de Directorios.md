### ***gobuster***

```
gobuster dir -u http://<IP>/ -t 100 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
```

---
### ***wfuzz***

```
wfuzz -c -t 200 --hc=404 --hl=<L que se repita> -w /usr/share/wordlists/SecLists/Discovery/Wed-Content/directory-list-lowecase-2.3-mediud.txt https://<IP>/FUZZ
```

---
### ***fuff***

```
ffuf -c -t 200 -w /usr/share/SecList/Discovery/Wed-Content/directory-list-2.3-mediud.txt -u https://<IP>/FUZZ
```

---
### ***dirsearch***

```
dirsearch -u http://<IP>/ -w /usr/share/wordlists/SecLists/Discovery/Wed-Content/directory-list-lowecase-2.3-mediud.txt
```

---
### ***dirb***

```
dirb http://<IP>
```

---
### **Leyenda**

- **`-u <URL>`**: El **objetivo** (URL, ej: `http://<IP>/` o `https://<IP>/FUZZ`).
- **`-w <wordlist>`**: Ruta al **diccionario** de palabras.
- **`-t <hilos>`**: Número de **hilos paralelos** para velocidad.
- **`-c`**: Salida con **colores**.
- **`--hc=404`**: **Oculta** respuestas con **código HTTP 404** (Not Found).
- **`--hl=<L>`**: **Oculta** respuestas con **`<L>` líneas** (identifica el valor observando).
- **`FUZZ`**: Marcador en la URL para **inyectar palabras** (para `wfuzz`, `ffuf`).
- **`-x <exts>`**: Busca **extensiones específicas** (ej: `txt,php`).