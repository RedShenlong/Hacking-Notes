### ***nmap Puertos TCP***

```
nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn <IpVictima> -oN <nombre>
```

---

### ***nmap Puertos UDP***

```
nmap -sU --top-ports 200 --min-rate=2000 -Pn <IpVictima> -oN <nombre>
```

---
### **Leyenda**

* **`-p-`**: Escanea **todos los 65535 puertos TCP**.
* **`--open`**: Muestra **solo los puertos que están abiertos**.
* **`-sS`**: Realiza un **escaneo SYN sigiloso** (para TCP).
* **`-sU`**: Realiza un **escaneo UDP**.
* **`-sC`**: Ejecuta los **scripts predeterminados** de Nmap.
* **`-sV`**: Intenta **detectar la versión** del servicio en los puertos abiertos.
* **`--top-ports 200`**: Escanea los **200 puertos más comunes** (útil para UDP o escaneos rápidos).
* **`--min-rate 2000`**: Envía paquetes a una **tasa mínima de 2000/segundo** (escaneo rápido, pero ruidoso).
* **`-n`**: **No resuelve nombres DNS**, acelerando el escaneo.
* **`-vvv`**: Muestra una **salida muy detallada** del progreso.
* **`-Pn`**: **No hace ping** para descubrir el host; asume que está activo.
* **`<IpVictima>`**: La **dirección IP del objetivo**.
* **`-oN <nombre>`**: Guarda la salida en un **archivo de texto plano** llamado `<nombre>`.

