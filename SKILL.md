---
name: stream-thumbnail
description: Prepara y publica la repeticion de un directo de gaming en YouTube - portada a sangre 1280x720 con la API de Canva (un juego o MIX SESSIONS), descripcion con capitulos, etiquetas, categoria y titulo del juego, lista de reproduccion del año y numeracion de episodio. Usar cuando el usuario pida "portada", "miniatura" o "thumbnail" de un directo o stream, cuando pida la descripcion o los capitulos de un directo, cuando haya que numerar episodios cruzando el historial del canal, o cuando pida subir, publicar o completar los datos del VOD de un directo en Studio.
---

# Portadas de directo

Duplicar un diseño de Canva y rellenarlo por API. **No usa brand templates**: requieren
Canva Enterprise y la cuenta tipica no los tiene. `search-brand-templates` devolviendo
`{"items":[]}` no es un error, es lo normal.

## Ciclo por portada

1. `copy-design` de la plantilla, indicando la pagina que sirve de base
2. `read-design` con `open_transaction: true` → devuelve `transaction_id`, `page_id` y los
   `element_id`
3. `edit-design` con `finalize: "keep_open"` y las operaciones: cover, logo, `replace_text` del
   `#N`, `update_title`
4. `edit-design` con `finalize: "commit"` y sin operaciones (sin esto se pierde todo)
5. `export-design` a **JPG 1920x1080, `quality: 95`, `export_quality: "pro"`** (~600 KB, el
   limite de YouTube son 2 MB). El PNG 1280x720 `lossless` sigue valiendo, pero a 1920 los
   textos y formas de la plantilla ganan nitidez real
6. Descargar la URL del export; caduca en horas
7. Escribir el `.txt` de la descripcion al lado de la imagen
8. **Parar y pasar la liga de revision. No subir nada hasta que el usuario confirme**

Los sufijos de `element_id` son identicos en toda copia de la misma pagina; solo cambia el
prefijo, que es el `page_id`. Asi que basta con leerlos una vez y componer:
`{page_id}-{sufijo}`.

## Revision antes de subir: obligatoria

Pedido por el usuario el 2026-11-20, despues de que dos portadas llegaran a YouTube con
problemas que se ven en dos segundos (el badge sobre la cara de Jason, el logo cruzando la
diagonal). **Nunca subir a Studio sin que el usuario haya visto la portada y dicho que si.**

Pasarle las dos ligas:

| Liga | De donde | Para que |
|---|---|---|
| URL del export PNG | respuesta de `export-design` | **los pixeles exactos** que se subirian. Caduca en horas |
| `view_url` del diseño | `read-design` con `filter.fields: ["design_metadata"]` | abrir en Canva y retocar a mano |

La del export es la que manda: es el archivo, no una previsualizacion. El `view_url` **rota**
entre llamadas, asi que hay que pedirlo en el momento, no reusar uno viejo.

Con el "si", subir y **verificar recargando** la pagina de detalles del video. El grid de
`/streams` cachea la miniatura vieja un buen rato: comprobar ahi no vale.

Y la trampa que ya pico una vez: **corregir en Canva y exportar no es subir.** Tras cualquier
arreglo hay que volver a Studio y subir el PNG nuevo, o YouTube se queda con el viejo.

## Formato por defecto: portada a sangre

Desde el 2026-11-18 las portadas de un solo juego se hacen **a sangre**: el key art ocupa los
1280x720 y todo el texto vive en una columna diagonal oscura, **siempre al lado contrario del
sujeto**. Geometria exacta de las dos orientaciones en `reference/layout.md`. El formato
seccionado antiguo (caja de cover a la izquierda, columna a la derecha) solo se usa si el usuario
lo pide.

**Ningun elemento puede tapar una cara.** Por eso el badge LIVE se movio dentro de la columna.
Antes de dar una portada por buena, mirar donde cae cada elemento sobre el arte. Las flechas
rojas de esquina tambien tapan: en Dark Souls cayeron sobre el casco y hubo que moverlas.

**Mix de dos juegos: tambien a sangre, con split vertical diagonal.** Aprobado por el usuario el
2026-11-22. Los dos key art llenan el frame partidos por una costura diagonal oscura con filo
rojo, los dos logos van apilados en la columna (recortados con `&trim=1`, que es lo que de
verdad los hace grandes) y `MIX SESSIONS #N` es un subtitulo pequeño colgado del badge, no un
gancho. Geometria en `reference/layout.md`. Plantilla de la que copiar: `DAHbp8vMb9A`
(`STREAM_2026_10_22_MIX_SESSIONS_11_OVERWATCH_GEARS_EDAY_FULLBLEED`).

## Reglas de contenido

**Prioridad al elegir arte, por encima de todo lo demas: cada juego tiene que aparecer con la
cara de un personaje suyo y con su logo.** Las dos cosas, en el mismo hueco o repartidas entre
arte y logo suelto. Un arte bonito de paisaje, nave o escenario sin personaje no vale aunque
lleve el titulo; y un arte con personaje pero sin titulo tampoco, hay que ponerle el logo
encima. Si el arte candidato no cumple las dos, seguir buscando antes que conformarse.

- **Cover oficial del juego** en la caja grande, no un frame del video. El frame no aporta y
  ademas no se puede subir: `upload-asset-from-url` solo acepta URLs ya publicas.
- **Juego de servicio** (GTA Online, Overwatch, Marvel Rivals, Fortnite…): arte de la
  temporada o evento vigente **en la fecha del directo**, no la cover perenne.
- **Overwatch**: solo season, nunca evento, y con personaje visible. Es una excepcion pedida
  explicitamente.
- **Juego cerrado** (Pokemon, Smash, Barony, R.E.P.O.…): cover perenne, no cambia.
- **Portada y titulo son pareja**: la portada abre la curiosidad, el titulo la cierra, y no
  repiten lo mismo. El titulo empieza por el juego, no por el chiste. La portada lleva 2-3
  palabras de gancho bajo el logo. El `#N` se queda. Medido: los VOD de directo rinden 0,35% de
  CTR contra 2,18% de los videos editados. Ver `reference/ctr.md`.
- Recortar el titulo del juego al encuadrar **se permite**: en el formato a sangre, tanto de un
  juego como de mix, cada juego lleva su logo suelto en la columna. De hecho se prefiere key art
  **limpio, sin titulo impreso**, porque el logo entra aparte y mas legible.

## Cada directo genera DOS videos: horizontal y vertical

El software de streaming manda dos señales a la vez, asi que en Studio salen **dos entradas por
directo**, con el mismo titulo y una duracion que difiere en 2-3 segundos. Lo que cambia es la
relacion de aspecto: una es 854x480 y la otra 480x854.

**Trabajar siempre sobre la horizontal.** La vertical va a privado. El usuario lo pide asi en
cada directo.

Como distinguirlas: **no por el orden del listado ni por la miniatura**, que es 320x180 para las
dos. Abrir cada una en el reproductor y leer el tamaño real del elemento de video:

```js
const v = document.querySelector('video');
({ w: v.videoWidth, h: v.videoHeight })   // 854x480 = horizontal, 480x854 = vertical
```

Es la unica comprobacion fiable. En la serie de Zelda la vertical salio **primera** en el
listado las tres veces, pero eso es casualidad del orden, no una regla.

## Numeracion de episodios

El `#N` sale de cruzar dos fuentes:

- historial de directos del canal (`youtube.com/@CANAL/streams`, leer `ytInitialData` en el
  browser; los items son `lockupViewModel`, no `videoRenderer`)
- los `#N` que ya usan las portadas anteriores

Se numera por juego y en orden cronologico. Si un juego encadena temporadas propias
(Overwatch S1 #01…#06, S2 #07), el siguiente continua la cuenta, no reinicia.

**Directo de dos juegos o mas: `MIX SESSIONS #N`.** Serie propia, con su cuenta aparte. En el
formato a sangre el texto completo va en el **subtitulo bajo el badge** y el elemento del `#N`
suelto se borra. La cuenta de cada juego suelto se queda donde estaba: un mix no gasta episodio
de nadie. Ultimo: `MIX SESSIONS #11`, 2026-10-22 (Overwatch + Gears E-Day, video 112fScUAeS8),
asi que el siguiente mix es el #12.

Ese #11 llego a YouTube **titulado como #10**, igual que el directo anterior: el titulo se
corrigio a mano el 2026-11-22. Si dos VOD comparten numero en el titulo, manda la portada.

El titulo del directo en YouTube **no es fuente fiable** para saber que se jugo: el del 28 de
julio se llamaba "Jugando lo que sea y Sobrepensando Spiderman BND" y su portada es de Overwatch.
Mandan las portadas anteriores, que se leen con `read-design`.

**Antes de numerar, mirar el canal entero, no solo el video que pasan.** Dos directos del mismo
juego en la misma semana son normales: Gears E-Day tuvo el #1 el 06-10-2026, el #2 el
08-10-2026 y el #3 el 21-10-2026, y numerar solo con el segundo a la vista dio un #1 equivocado.
La beta (`E-DAY BETA`, 07-08-2026) no entra en la cuenta de la campaña. Y si el listado de
`/streams` no cuadra con lo que dice el usuario, releerlo: los titulos cambian cuando el los
actualiza.

Si el titulo del directo no dice el juego, bajar los subtitulos automaticos, que van rapido
aunque el video este throttleado, y buscar nombres de personajes o modos:

```bash
yt-dlp --write-auto-sub --sub-lang "es.*" --skip-download --sub-format vtt URL
```

## Detalles que rompen si se ignoran

- `resize_element` **recorta** el relleno, no lo reescala. Para cambiar tamaño de verdad:
  `delete_element` + `insert_fill` con `width`/`height`.
- La caja del logo debe tener **el mismo ratio que el PNG del logo**. Si no, se recorta y se
  come letras.
- `update_fill` recorta centrado y no hay forma de elegir el encuadre por API. Elegir arte
  con el sujeto centrado, o dejar el reencuadre fino para el humano.
- Canva no borra assets por API. Subir solo lo que se vaya a usar.

## Publicar el VOD: no es solo la portada

Una repeticion de directo queda completa cuando tiene **titulo, portada, descripcion con
capitulos, etiquetas, categoria `Videojuegos` + titulo del juego, y la lista de reproduccion del
año en curso**. El paso a paso, con los selectores de Studio y las trampas de cada campo, esta
en `reference/publishing.md`.

Dos cosas que conviene saber antes de empezar:

- El campo **"Titulo del juego (opcional)"** vive bajo *Mostrar mas* → *Categoria*, es un
  combobox con sugerencias (hay que **elegir de la lista**, no basta teclear) y **solo admite un
  juego**: en un mix, el que mas minutos se jugo.
- La portada sigue necesitando el **si del usuario** antes de tocar Studio. El resto de campos
  se pueden dejar preparados.

## Descripcion del video

Cada portada sale con su descripcion de YouTube en un `.txt` al lado del PNG, mismo nombre con
`DESC_` en vez de `STREAM_`. Plantilla y reglas en `reference/description.md`. En los mix los
tiempos los da el usuario; si no los tiene, se sacan de los **subtitulos automaticos**, porque
en un VOD de directo `--download-sections` no sirve y no se pueden muestrear frames.

## Referencias

- `reference/ctr.md` — CTR real medido por tipo de portada, la regla de portada+titulo y como volver a medirlo
- `reference/description.md` — plantilla de descripcion, capitulos y links fijos del canal
- `reference/publishing.md` — publicar el VOD en Studio: todos los campos, en orden, con sus trampas
- `reference/layout.md` — geometria exacta de cada hueco y los tamaños que funcionan
- `reference/asset-sources.md` — de donde sacar covers, logos y arte de temporada
- `reference/canva-api.md` — orden de llamadas, errores y limites
- `assets/games.json` — registro de juegos ya resueltos, con sus URLs
