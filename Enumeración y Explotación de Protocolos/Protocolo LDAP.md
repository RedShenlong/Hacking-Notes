# Protocolo LDAP (Lightweight Directory Access Protocol)

> **Puerto por defecto:** `389/TCP` (LDAP) · `636/TCP` (LDAPS, cifrado) · `3268/TCP` (Global Catalog en AD)

LDAP es un protocolo de consulta de servicios de directorio. En la práctica, casi siempre estás ante **Active Directory**: un LDAP mal protegido (o con **bind anónimo**) permite enumerar **usuarios, grupos, equipos y descripciones** del dominio sin credenciales.

---

## Enumeración

```bash
nmap -p389,636,3268 -sCV --script "ldap*" <IP>
```

### Obtener el "naming context" (base DN)

Antes de consultar necesitas la base del árbol (`dc=dominio,dc=local`):

```bash
ldapsearch -x -H ldap://<IP> -s base namingcontexts
```

---

## Bind anónimo (sin credenciales)

Prueba siempre a consultar sin autenticación:

```bash
# Volcar TODO el directorio
ldapsearch -x -H ldap://<IP> -b "dc=dominio,dc=local"

# Solo usuarios
ldapsearch -x -H ldap://<IP> -b "dc=dominio,dc=local" "(objectClass=user)" sAMAccountName

# Solo grupos
ldapsearch -x -H ldap://<IP> -b "dc=dominio,dc=local" "(objectClass=group)" cn
```

> **Consejo:** revisa el campo **`description`** de los usuarios: es un clásico encontrar ahí contraseñas puestas por administradores perezosos ("cuenta temporal, pass: Verano2024"). También `info` y `comment`.

---

## Consulta autenticada (con credenciales)

```bash
ldapsearch -x -H ldap://<IP> -D "usuario@dominio.local" -w '<pass>' \
  -b "dc=dominio,dc=local" "(objectClass=user)"
```

Con **netexec** es mucho más cómodo para AD:

```bash
netexec ldap <IP> -u <usuario> -p <pass> --users
netexec ldap <IP> -u <usuario> -p <pass> --groups
netexec ldap <IP> -u <usuario> -p <pass> --password-not-required   # cuentas sin password
netexec ldap <IP> -u <usuario> -p <pass> --trusted-for-delegation  # delegación
```

---

## Ataques sobre LDAP en Active Directory

Con cualquier credencial válida, LDAP alimenta los ataques clásicos de AD:

```pipeline
LDAP anónimo/credencial -> lista de usuarios -> AS-REP / spraying -> BloodHound -> Domain Admin
```

- **AS-REP Roasting / Kerberoasting**: LDAP te da los usuarios y SPNs (ver *Active Directory · Kerberoasting y AS-REP*).
- **BloodHound**: recolecta todo el directorio por LDAP (ver *Active Directory · BloodHound*).
- **Cuentas con `userAccountControl` peligroso**: sin preauth, password no requerida, delegación.

---

## windapsearch / ldapdomaindump

Alternativas que presentan la información ya ordenada:

```bash
ldapdomaindump -u 'dominio\usuario' -p '<pass>' <IP>    # genera HTML/JSON navegable
windapsearch -d dominio.local --dc-ip <IP> -U           # listar usuarios
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `ldapsearch` | Consultas LDAP manuales (incluye bind anónimo). |
| `netexec ldap` | Enumeración de AD por LDAP (usuarios, grupos, flags). |
| `ldapdomaindump` | Vuelca el dominio a HTML/JSON navegable. |
| `windapsearch` | Enumeración orientada a AD. |
| `BloodHound` | Mapa de ataque a partir de datos LDAP. |

---

> **Consejo:** ante el puerto 389, lo primero es intentar un **bind anónimo** y volcar usuarios + sus `description`. Esa lista de usuarios es la gasolina de todo lo demás en Active Directory (AS-REP, spraying, BloodHound). Si el anónimo falla, en cuanto consigas una credencial cualquiera, vuelve a LDAP con netexec.
