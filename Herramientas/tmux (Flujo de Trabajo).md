# tmux (flujo de trabajo en terminal)

**tmux** es un multiplexor de terminal: divide una terminal en varios paneles, mantiene sesiones vivas aunque cierres la ventana, y es **imprescindible** en pentest. ¿Por qué? Porque un listener de reverse shell dentro de tmux **no se pierde** si tu terminal se cierra, y porque trabajar con notas + escaneo + shell a la vez multiplica tu velocidad.

---

## Por qué importa en hacking

```flow
No perder shells :: El listener vive en una sesión tmux; cierras la ventana y sigue ahí.
Paralelismo :: Un panel escanea, otro enumera, otro tiene la shell, otro las notas.
Scroll y copia :: Revisas salidas largas (linpeas) sin perderlas.
Persistencia :: Reconectas a la sesión tras una caída de VPN.
```

---

## Supervivencia: lo mínimo que necesitas

El **prefijo** por defecto es `Ctrl+b` (se pulsa, se suelta, y luego la tecla de acción).

```text
tmux                      # iniciar una sesión
tmux new -s htb           # sesión con nombre
tmux ls                   # listar sesiones
tmux a -t htb             # reconectar (attach) a 'htb'
Ctrl+b  d                 # "detach": salir dejando la sesión viva
```

### Paneles (divisiones)

```text
Ctrl+b  %        # dividir en vertical (izq/der)
Ctrl+b  "        # dividir en horizontal (arriba/abajo)
Ctrl+b  <flecha> # moverte entre paneles
Ctrl+b  x        # cerrar el panel actual
Ctrl+b  z        # zoom (pantalla completa del panel) / volver
Ctrl+b  {  }     # mover el panel de sitio
```

### Ventanas (pestañas)

```text
Ctrl+b  c        # nueva ventana
Ctrl+b  n / p    # siguiente / anterior
Ctrl+b  <número> # ir a la ventana N
Ctrl+b  ,        # renombrar la ventana
```

### Scroll y copia

```text
Ctrl+b  [        # entrar en modo scroll (flechas / PgUp); 'q' para salir
```

> **Consejo:** para revisar la salida larga de `linpeas` o un escaneo, entra en modo scroll con `Ctrl+b [`. Sin tmux (o con una reverse shell cruda) esa salida se te va y toca relanzarla.

---

## El montaje clásico de pentest

Una ventana con 4 paneles:

```pipeline
Panel 1: nmap/escaneo -> Panel 2: enumeración (ffuf/web) -> Panel 3: la shell -> Panel 4: notas (vim)
```

```bash
tmux new -s maquina
# Ctrl+b %  (divide)   Ctrl+b "  (divide otra vez)   ... hasta tener tu rejilla
```

---

## El truco que salva shells

Pon tu **listener** dentro de tmux antes de lanzar la reverse shell:

```bash
tmux new -s shells
nc -lvnp 443
# Ctrl+b d  -> te sales, pero el nc sigue escuchando
```

Si tu VPN cae o cierras la terminal, la shell recibida **sigue viva**. Reconectas con `tmux a -t shells` y ahí está. Esto evita el dolor nº1 del novato: perder un foothold costoso por un cierre accidental.

---

## Configuración recomendada (~/.tmux.conf)

```bash
# Ratón: seleccionar paneles y hacer scroll con la rueda
set -g mouse on
# Más historial de scroll
set -g history-limit 10000
# Numerar ventanas desde 1
set -g base-index 1
# Recargar config: Ctrl+b r
bind r source-file ~/.tmux.conf \; display "Recargado"
```

Aplica con `tmux source-file ~/.tmux.conf` o reinicia tmux.

---

## Herramientas relacionadas

| Herramienta | Descripción |
| ----------- | ----------- |
| `tmux` | Multiplexor de terminal (sesiones, paneles, persistencia). |
| `screen` | Alternativa clásica (más simple, menos flexible). |
| `pwncat-cs` | Listener que además persiste y estabiliza shells. |
| `vim` / `nano` | Editor en un panel para las notas de la máquina. |

---

> **Consejo:** no hace falta dominar tmux entero: con `new -s`, `a -t`, `d` (detach), dividir paneles (`%` y `"`) y el modo scroll (`[`) ya trabajas como un profesional. El hábito que de verdad cambia tu juego es **lanzar siempre el listener dentro de tmux**: nunca más perderás una shell por cerrar una ventana.
