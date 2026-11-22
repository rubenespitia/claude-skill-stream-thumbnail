# Descripcion de YouTube del directo

Sale junto con cada portada. Se entrega como `.txt` al lado del PNG, con el mismo nombre
cambiando `STREAM_` por `DESC_`, para copiar y pegar sin reformatear.

## Plantilla

```
<Una o dos frases: que se jugo y que paso. Los primeros ~100 caracteres son lo unico que se ve
sin pulsar "mas", y es lo que acompaña al video en busqueda y sugeridos. Nombre del juego ahi.>

CAPITULOS
0:00 - <Juego o seccion>
<mm:ss> - <Juego o seccion>

EN VIVO
Twitch: https://twitch.tv/wantedgull

REDES
Instagram: https://instagram.com/wantedgull
X: https://x.com/WantedGull

MAS DIRECTOS
2026 STREAMING: https://www.youtube.com/playlist?list=PL1lpZJzen8sliXbry7SkBK6OYonAANN9Q

JUEGOS DE ESTE DIRECTO
<Juego> - <estudio> - <web oficial o ficha de tienda>

#Juego1 #Juego2 #Directo
```

El orden no es casual: capitulos arriba porque es lo que se usa, luego los sitios donde seguir
al canal, y el bloque de juegos al final. Son links que sacan de YouTube y solo los busca quien
ya decidio que le interesa el juego.

En portada de un solo juego el bloque de capitulos sigue valiendo: un VOD de dos horas sin
capitulos no se navega.

Sacar los tiempos no es solo para los capitulos: **sirve para saber que paso en el directo**.
El titulo de YouTube miente a menudo. El de Gears E-Day decia "DAY ONE" y parecia el arranque
de la campaña, y en realidad era Acto 5 Capitulo 5 desde el minuto cero, creditos a los 46 y
Horde Siege el resto. El de Dark Souls se llamaba "Esperando el Zelda en Anor Londo" y en Anor
Londo no se pone en ningun momento: acaba subiendo a la torre de la primera campana, medio
juego antes. Escribir el gancho sin mirar el video es inventarselo.

## Sacar los tiempos: frames desde el browser, subtitulos de apoyo

Si el directo es de un juego con zonas y jefes, **empezar por los frames** (seccion de abajo):
dan carteles, menus y nombres de jefe impresos en pantalla, y no dependen de que el streamer
nombre las cosas. Los subtitulos sirven para confirmar y para directos de charla.

**Con yt-dlp no se pueden muestrear frames de un VOD de directo.** Los formatos llegan marcados
como `source=yt_live_broadcast` y `--download-sections` deja `.mp4` de 262 bytes que ffmpeg no
abre:

```
Error opening input: Invalid data found when processing input
ERROR: ffmpeg exited with code 3199971767
```

Probado con `worst`, con `worstvideo[height>=240]` y con `--force-keyframes-at-cuts`: lo mismo.

**Y en un VOD recien emitido puede que ni existan.** A las 12 horas del directo de GTA VI,
`yt-dlp --list-subs` daba `has no automatic captions` y solo ofrecia `live_chat`. Ahi los
capitulos salen enteros de frames; comprobar con `--list-subs` antes de perder tiempo.

Los **subtitulos automaticos** si bajan enteros y rapido aunque el video este throttleado:

```bash
yt-dlp -q --write-auto-sub --sub-lang "es" --skip-download --sub-format vtt URL
```

De ahi, dos pasadas con awk sobre el VTT:

1. **Digest por franjas de 5 min.** Dedupe de las lineas rodantes (`gsub(/<[^>]*>/,"")` y
   saltar la linea si es igual a la anterior) y 300 caracteres por franja. Da la forma general
   del directo en ~40 lineas
2. **Grep con timestamp por palabras clave.** Nombres de zona, jefe y objeto del juego. Esto es
   lo que fija el minuto exacto del corte

El digest solo ubica; los keywords son los que dan la frontera. En el directo de Zelda el
digest decia "por ahi va el Arbol Deku" y el grep daba `0:43:58 La reina goma` y
`0:52:17 La esmeralda`. Validado con el usuario: los capitulos salidos asi se dieron por buenos
sin revisar video.

Ojo con los nombres: el reconocedor los destroza. Gohma sale como "goma", Saria como "Sara".
Meter variantes en el patron.

### Muestreo de frames en VOD de directo: si se puede, desde el browser

Lo que no funciona es `yt-dlp --download-sections`. El reproductor de YouTube **si** entrega los
frames: con la extension de Chrome, se hace seek sobre el `<video>` y se dibuja en un `<canvas>`.
Sale una hoja de contacto entera con **una sola screenshot**, que es mucho mas barato que una
captura por frame.

```js
const v=document.querySelector('video'); v.muted=true;
const seek=t=>new Promise(res=>{const h=()=>{v.removeEventListener('seeked',h);setTimeout(res,400)};
  v.addEventListener('seeked',h); v.currentTime=t;});
// por cada tiempo: await seek(t); ctx.drawImage(v,x,y,400,225); y rotular con ctx.fillText
// al final, meter el canvas en un div fixed inset:0 z-index:999999 y hacer UNA screenshot
```

Detalles que importan:

- Rejilla **4x4 de 400x225**. A 320x180 (5 cols) los carteles de zona y los menus no se leen
- El overlay necesita `max-height:100vh; object-fit:contain`, si no la primera fila se queda
  fuera de viewport y hay que scrollear
- **8 frames por hoja, y en una pestaña nueva.** Con 16 frames sobre un VOD de 3h42 el renderer
  se congela: `Page.captureScreenshot` da timeout a los 30 s aunque `javascript_tool` siga
  respondiendo. Una pestaña que ya se uso para otra cosa se queda colgada incluso tras recargar
- Rotular cada frame con su timestamp **dentro del canvas**, no fiarse del orden
- Estrategia: una pasada cada ~6 min para la forma general, y despues **una pasada de un frame
  por minuto en cada frontera**. Esto no es opcional: en el Zelda #2 la pasada de 6 min dio
  **tres capitulos equivocados** y los noto el usuario. Confundio el Templo del Tiempo con un
  cementerio, el Bosque Kokiri con el Pozo de Kakariko, y fundio el Crater de la Montaña de la
  Muerte con el Templo del Fuego porque los dos son lava. La pasada fina los arreglo todos
- Lo que de verdad da señal, por orden: pantalla de jefe con el nombre impreso, partitura de
  cancion (lleva el nombre en rojo), cartel de descubrimiento de zona, menu de item, lista de
  misiones activas. **El decorado solo engaña**: dos cuevas naranjas no son el mismo sitio
- No hay DRM en los VOD normales, asi que `drawImage` no falla. El canvas queda *tainted* pero
  da igual: solo se muestra, no se leen pixeles

**Esto manda sobre los subtitulos cuando el directo es cotorreo.** En el Zelda #2 el digest de
subtitulos era charla sobre remakes y promociones durante tres horas, y el grep de keywords de
zonas daba casi cero. Los frames dieron la timeline entera: pantallas de jefe con nombre
impreso (`GANON FANTASMA`), menus de item (`Arco de las Hadas`), carteles de descubrimiento
(`Ciudad Goron`), partituras de cancion y la lista de misiones activas. Mas fiable que la
transcripcion y mas rapido.

El panel de transcripcion tambien se lee del DOM sin yt-dlp, util cuando las cookies no se
pueden extraer (Chrome cifra con App-Bound: `--cookies-from-browser` da *Failed to decrypt with
DPAPI*). Abrir *Mostrar transcripcion* y leer `transcript-segment-view-model`; el timestamp esta
en `.ytwTranscriptSegmentViewModelTimestamp` y el panel **no** esta virtualizado, estan las
~1.750 lineas de un directo de tres horas. Procesar el digest dentro del propio JS y devolver
solo el resultado, que `javascript_tool` trunca cerca de 1.500 caracteres.

### Muestreo de frames con yt-dlp: solo para subidas normales

Para un video subido como archivo (no un VOD de directo) el muestreo sigue valiendo:

```bash
yt-dlp -f "worstvideo[height>=240]/worst" --download-sections "*00:36:00-00:36:04" \
  -o "chk_%(section_start)s.%(ext)s" URL
```

Una pasada cada diez minutos ubica los bloques; luego se afina con dos o tres mas en cada
frontera. Montar las capturas en un contact sheet con `hstack`/`vstack` deja verlas de una vez.
**Nada de etiquetar los frames con `drawtext`**: en el ffmpeg de este equipo revienta con
segfault por falta de fontconfig; hay que llevar el orden a mano.

## Que funciona y que no

- **Las dos primeras lineas son el 90% del valor.** Es lo que se indexa y lo que se lee. El
  resto de la descripcion casi nadie lo abre.
- **Capitulos**: el primero tiene que ser `0:00`, minimo tres, minimo 10 segundos cada uno. Si
  no se cumple, YouTube no los activa y no aparece la barra segmentada.
- **En un directo de horas, escribir SIEMPRE `H:MM:SS`.** Esto es lo que mas se rompe. `0:28`
  no es "veintiocho minutos": YouTube lo lee como **28 segundos**, y toda la lista se va al
  primer minuto del video. Pasa con 18 capitulos de golpe y no da ningun error, simplemente
  quedan apelotonados al principio. Los capitulos de Zelda #2 y #3 se publicaron mal por esto y
  hubo que rehacerlos. Escribir `0:28:00`, no `0:28`. Comprobar antes de guardar:
  ```js
  // caps debe ser el numero de capitulos; viejos tiene que dar 0
  ({ caps: (d.match(/^\d:\d\d:\d\d - /gm)||[]).length, viejos: (d.match(/^\d:\d\d - /gm)||[]).length })
  ```
  `MM:SS` solo vale cuando el corte esta de verdad por debajo de la hora (`44:00` = minuto 44),
  pero mezclarlo invita al error: en un directo largo, `H:MM:SS` en todos y listo.
- **Una pausa larga se marca como capitulo**, no se esconde. El espectador agradece saltarla.
- **Hashtags: tres como mucho.** Los tres primeros salen encima del titulo. Mas de 15 y YouTube
  los ignora todos.
- **Links oficiales del juego**: web del estudio o ficha de tienda. Aporta al espectador y
  evita que el nombre del juego quede suelto sin contexto.
- **El link a playlist rinde mas que el link externo**: se queda dentro de YouTube y encadena
  otro video. El link a Twitch va igualmente, porque el VOD es el anzuelo del directo.
- **No funciona**: amontonar keywords, repetir el titulo, listas de tags separadas por comas,
  descripciones copiadas tal cual entre videos.

Con un canal pequeño la descripcion no mueve la aguja por si sola: manda la miniatura y el
titulo. La descripcion sirve para que quien ya hizo clic encuentre el resto y para que el VOD
sea navegable.

## Escribir la descripcion en Studio: comprobar el principio

El campo acepta el texto entero de una tacada con `type`, pero al acabar en hashtag deja abierto
el desplegable de sugerencias: se cierra con Escape.

**Si el campo esta vacio, `ctrl+a` se escribe como la letra `a`.** No hay nada que seleccionar,
asi que el atajo entra como caracter y la descripcion queda empezando por `aFinal de...`. Pasa
solo en campos vacios; si ya habia texto, el `ctrl+a` funciona.

Comprobar siempre el resultado antes de guardar:

```js
const walk=(r,out)=>{r.querySelectorAll('*').forEach(e=>{if(e.getAttribute&&e.getAttribute('contenteditable')==='true')out.push(e);if(e.shadowRoot)walk(e.shadowRoot,out);});return out;};
const d=walk(document,[]).map(e=>e.innerText).sort((a,b)=>b.length-a.length)[0]||'';
JSON.stringify({len:d.length, head:d.slice(0,60), caps:(d.match(/^\d:\d\d - /gm)||[]).length})
```

Se arregla con `ctrl+Home` y un `Delete`.

## Añadir el VOD a la playlist del anio

Paso fijo, igual que la portada y los capitulos: el directo entra en la playlist de streams **del
anio en curso**. En Studio, detalles del video → *Listas* → marcar la del anio → Guardar, y
verificar recargando.

Si no se hace, el enlace de la propia descripcion manda al espectador a una lista donde **no
esta** el video que acaba de ver. En enero toca playlist nueva: la regla es el anio en curso, no
la playlist concreta.

## Datos fijos del canal

| Que | Valor |
|---|---|
| Canal | https://youtube.com/@WantedGull |
| Twitch | https://twitch.tv/wantedgull |
| Instagram | https://instagram.com/wantedgull |
| X | https://x.com/WantedGull |
| Playlist directos | `PL1lpZJzen8sliXbry7SkBK6OYonAANN9Q` (2026 STREAMING) |
| Playlist opiniones | `PL1lpZJzen8snacsEKvqxQuRnhd3FwCAP4` (2026 Opiniones) |
