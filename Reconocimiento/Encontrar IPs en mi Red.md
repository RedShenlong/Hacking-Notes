### ***arp-scan***

```
arp-scan -I eth0 --localnet
```

---
### ***netdiscover***

```
netdiscover -i eth0 -r <Ipmodificada>
```

---
### ***nmap***

```
nmap -sn <Ipmodificada> -oN <nombre>
```

---

### **Leyenda**

* **`<Ipmodificada>`**:Esto debes sustituirlo por la IP de tu máquina atacante, pero el último octal cámbialo por `.0/24`. Por ejemplo, si tu IP es `192.168.1.10`, usarías `192.168.1.0/24`.
