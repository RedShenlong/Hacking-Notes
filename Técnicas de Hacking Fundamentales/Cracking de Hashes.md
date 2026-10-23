# Cracking de Hashes

Recuperar la contraseña a partir de su **hash** es una tarea constante: hashes de bases de datos, de ZIP/documentos, de Kerberos, de Windows (NTLM)... Esta nota reúne cómo **identificar** un hash y crackearlo con **hashcat** y **John the Ripper**.

> Para extraer hashes de servicios ver [Fuerza Bruta](./Fuerza%20Bruta.md) y, en AD, [DCSync](../Active%20Directory/DCSync%20y%20Volcado%20de%20Credenciales.md).

---

## Paso 1 — Identificar el hash

Antes de crackear necesitas saber **qué tipo** es:

```bash
hashid '<hash>'
hash-identifier          # interactivo
nth --text '<hash>'      # name-that-hash (muy bueno, sugiere modos de hashcat)
```

Pistas rápidas por formato:

| Aspecto | Tipo | hashcat `-m` |
| ------- | ---- | ------------ |
| `$1$...` | MD5-crypt (Unix) | 500 |
| `$6$...` | SHA512-crypt (Unix, `/etc/shadow`) | 1800 |
| `$2a$`/`$2b$` | bcrypt | 3200 |
| `$y$`/`$7$` | yescrypt / scrypt | (ver docs) |
| 32 hex | MD5 | 0 |
| 40 hex | SHA1 | 100 |
| 64 hex | SHA256 | 1400 |
| `aad3b...:31d6c...` | NTLM (Windows) | 1000 |
| `$krb5asrep$` | AS-REP (Kerberos) | 18200 |
| `$krb5tgs$` | TGS (Kerberoasting) | 13100 |
| `$NT$...` / NetNTLMv2 | NetNTLMv2 (Responder) | 5600 |
| `$P$`/`$H$` | phpBB/WordPress (MD5) | 400 |

---

## Paso 2a — hashcat (GPU, el más rápido)

```bash
# Ataque de diccionario
hashcat -m <modo> -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# Diccionario + reglas (muy efectivo)
hashcat -m <modo> -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Fuerza bruta por máscara (ej. 8 caracteres alfanuméricos)
hashcat -m <modo> -a 3 hash.txt '?l?l?l?l?l?l?l?l'

# Ver resultados ya crackeados
hashcat -m <modo> hash.txt --show
```

Máscaras: `?l` minúscula, `?u` mayúscula, `?d` dígito, `?s` símbolo, `?a` todo.

> **Consejo:** las **reglas** (`best64.rule`, `rockyou-30000.rule`) multiplican tu diccionario aplicando mutaciones (mayúsculas, añadir números/símbolos). Casi siempre revientan muchas más contraseñas que el diccionario a secas. Úsalas por defecto.

---

## Paso 2b — John the Ripper

```bash
# Autodetecta el formato
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Especificar formato si no lo detecta
john --format=<formato> --wordlist=rockyou.txt hash.txt

# Con reglas
john --wordlist=rockyou.txt --rules hash.txt

# Ver crackeadas
john --show hash.txt
```

### Utilidades "2john" (extraer el hash de un archivo)

```bash
zip2john archivo.zip > hash        # ZIP protegido
rar2john archivo.rar > hash
ssh2john id_rsa > hash             # clave SSH con passphrase
keepass2john base.kdbx > hash      # KeePass
office2john documento.docx > hash  # Office
pdf2john documento.pdf > hash      # PDF
```

---

## /etc/shadow (combinar passwd + shadow)

```bash
unshadow /etc/passwd /etc/shadow > combinado.txt
john --wordlist=rockyou.txt combinado.txt
# o hashcat con el hash $6$ directamente (modo 1800)
```

---

## Hashes de Windows (SAM / NTDS)

El formato volcado es `usuario:RID:LM:NTLM:::`. Crackea la parte **NTLM** (modo 1000):

```bash
hashcat -m 1000 -a 0 ntlm_hashes.txt rockyou.txt -r best64.rule
```

Recuerda: muchos hashes NTLM **no hace falta crackearlos**, se usan tal cual con Pass-the-Hash (ver [Movimiento Lateral](../Active%20Directory/Movimiento%20Lateral.md)).

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `hashcat` | Crackeo acelerado por GPU (el más rápido). |
| `john` | Crackeo versátil, autodetección y utilidades `*2john`. |
| `hashid` / `name-that-hash` | Identificar el tipo de hash. |
| `unshadow` | Combinar `/etc/passwd` + `/etc/shadow`. |
| diccionarios/reglas | `rockyou.txt`, SecLists, `best64.rule`. |

---

> **Consejo:** el flujo es: **identificar** (`nth`/`hashid`) → elegir el **modo** correcto → diccionario **con reglas** (`rockyou + best64`) → si no cae, máscaras o diccionarios más grandes. Y antes de perder horas crackeando un NTLM, piensa si puedes **reutilizarlo directamente** con Pass-the-Hash: a veces no necesitas la contraseña en claro.
