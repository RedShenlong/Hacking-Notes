
## Nmap: Escaneo con Scripts de Vulnerabilidad y Seguridad

 ```
 nmap --script "vuln and safe" -p<puerto> <IP>
 ```

---
### Leyenda 
* **`--script "vuln and safe"`**: Ejecuta scripts **`vuln`** (buscan vulnerabilidades) y **`safe`** (seguros, no intrusivos).
* **`-p<puerto>`**: Define el **puerto o puertos específicos** (ej: `-p80`). Evita `-p-` con scripts.
* **`<IP>`**: La **IP del objetivo**.
