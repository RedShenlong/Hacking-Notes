# Mentalidad para Máquinas Difíciles

La diferencia entre quien resuelve una máquina *Hard* de Hack The Box y quien se rinde **casi nunca es saber una técnica más**. Es la **forma de pensar**: cómo gestionar la información, cómo no caer en agujeros, y cuándo cambiar de enfoque. Esto es lo que no suele estar escrito en los writeups (que te enseñan *qué* funcionó, no *cómo pensar cuando no sabes qué funciona*).

---

## La verdad incómoda: no estás atascado, estás enumerando a medias

El 95 % de los "no sé cómo seguir" no se resuelven con una técnica nueva, sino **revisando algo que ya tenías delante**. Antes de buscar un exploit raro, pregúntate con honestidad:

- ¿He leído **todo** el código fuente y el JavaScript de la web, no por encima?
- ¿He hecho fuzzing con **varios diccionarios** (no solo el medium)?
- ¿He probado **cada parámetro** contra cada clase de inyección?
- ¿He vuelto a enumerar **después** de conseguir el último acceso/credencial?
- ¿He mirado los puertos **UDP** y los servicios internos (`127.0.0.1`)?

> **Consejo:** si llevas una hora atascado, no abras Metasploit; **vuelve a enumerar el servicio que ya estabas mirando** con más profundidad. La vía casi siempre está ahí, en algo que leíste rápido y descartaste.

---

## Gestiona la información como un detective

Las máquinas difíciles te dan **mucha** información dispersa. Quien gana es quien la **conecta**, no quien recuerda más comandos.

- **Anótalo TODO** en un fichero por máquina: cada usuario, versión, ruta, cadena rara, nombre propio. Lo que no anotas, lo pierdes.
- Mantén una lista de **"pistas sin usar"**: cosas que viste y aún no has aprovechado (un nombre de usuario en un comentario, un puerto raro, un dominio en el certificado). Cuando te atasques, repásala.
- Cruza fuentes: un **nombre** del FTP + una **web** de login = intento de usuario. Un **dominio** del certificado TLS = vhost que fuzzear. Una **versión** = exploit que buscar.

```flow
Recopilar :: Vuelca cada dato en tu fichero de notas, sin filtrar.
Conectar :: Cruza datos de servicios distintos (nombre + login, versión + CVE).
Priorizar :: Ataca primero la pista más prometedora, hasta agotarla.
Reiniciar :: Con cada acceso nuevo, vuelve a enumerar como si fueras otro.
```

---

## Rabbit holes: cómo detectarlos y salir

Un **rabbit hole** (madriguera) es una pista que parece el camino pero no lleva a nada; los creadores de máquinas los ponen a propósito. Señales de que estás en uno:

- Llevas mucho tiempo y cada paso requiere algo cada vez más rebuscado.
- Necesitas un exploit "a medida" muy improbable para un lab.
- La dificultad de lo que intentas **no encaja** con el nivel de la máquina.

**Cómo salir:** marca por dónde ibas, **aparca** esa vía (no la borres) y vuelve a tu lista de pistas sin usar. A menudo la vía buena es más simple de lo que estabas intentando.

> **Consejo:** regla de los **dos intentos serios**. Si una vía te ha fallado dos veces de forma distinta, probablemente es un rabbit hole o te falta un dato previo. Cambia de pista y vuelve luego con más contexto.

---

## El principio de proporcionalidad (dificultad esperada)

Calibra tu esfuerzo según el nivel de la máquina:

| Nivel | Qué esperar |
| ----- | ----------- |
| Easy | Versión vulnerable obvia, credenciales por defecto, 1-2 pasos. |
| Medium | Encadenar 2-3 cosas; algo de enumeración fina; reutilización de credenciales. |
| Hard | Cadenas largas, pivoting, fallos lógicos, "chaining" de vulns, a veces código a revisar. |
| Insane | Técnicas poco comunes, exploits modificados, varios pivotes, lógica retorcida. |

Si en una *Easy* estás escribiendo un exploit a medida, **te has pasado de largo**: la vía es más simple. Si en una *Insane* esperas que `sudo -l` te lo dé todo, **te quedas corto**.

---

## La enumeración web profunda (donde se gana el juego)

La web es el vector nº1 y donde más gente enumera mal. Niveles de profundidad que casi nadie agota:

1. **Visual**: léela como un humano (textos, nombres, "powered by", pies de página).
2. **Código fuente + JS**: endpoints ocultos, comentarios, claves, rutas de API.
3. **Fuzzing en capas**: directorios → dentro de cada hallazgo, fuzzear otra vez → extensiones según la tecnología (`.php`, `.bak`, `.old`, `.txt`, `.zip`).
4. **Vhosts y subdominios**: por `Host:` header; muchas webs esconden paneles en vhosts.
5. **Parámetros ocultos**: `arjun`, `param miner`; a veces el parámetro vulnerable no está enlazado.
6. **Estados**: ¿qué cambia si te registras / inicias sesión / cambias de rol? Enumera de nuevo **autenticado**.

> **Consejo (poco conocido):** cuando una web no da nada, cambia el diccionario. `raft-*` encuentra cosas que `directory-list` no, y viceversa. Y fuzzea **también archivos** (`-x php,txt,bak,zip,old,~`), no solo directorios: un `index.php.bak` con el código fuente te resuelve la máquina.

---

## Cuándo parar de "atacar" y empezar a "leer"

En máquinas difíciles, muchas veces el salto no es un exploit, es **entender la lógica** de la aplicación:

- ¿Qué hace realmente ese endpoint? ¿Confía en algún dato del cliente?
- ¿Hay un flujo de negocio que puedas romper (precios, roles, estados, orden de pasos)?
- ¿Puedes leer el **código fuente** (LFI, `.git` expuesto, backup)? Si sí, **léelo**: ahí está la vulnerabilidad lógica servida.

---

## Rutina anti-bloqueo (pégatela en la pared)

Cuando no sepas seguir, recorre esto **en orden**:

1. Relee **tus notas** y la lista de pistas sin usar.
2. ¿He reenumerado **tras el último avance**?
3. ¿He agotado la **web** en los 6 niveles de arriba?
4. ¿He buscado **cada versión** en `searchsploit` y en Google con `exploit`/`CVE`?
5. ¿He probado **reutilizar** todas las credenciales en todos los servicios?
6. ¿He mirado **UDP** y servicios en `localhost` (para pivotar)?
7. Cambia de **diccionario** y repite el paso que diste por hecho.
8. Explica la máquina **en voz alta** (o por escrito) como si se la contaras a alguien: el "pato de goma" funciona.

> **Consejo final:** la habilidad que de verdad separa a los buenos es la **tolerancia a la frustración estructurada**: no rendirse, pero tampoco repetir lo mismo esperando otro resultado. Cada vez que te atasques y salgas, tu "sentido arácnido" mejora. Por eso se dice *Try Harder* — no significa "machaca más", significa **"enumera mejor y piensa distinto"**.
