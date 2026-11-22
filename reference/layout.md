# Geometria de la plantilla

Pagina 1280x720. Coordenadas en pixeles desde la esquina superior izquierda.

## Huecos

| Hueco | Sufijo de element_id | Posicion inicial | Tamaño inicial |
|---|---|---|---|
| Cover del juego | `-LBLlW49sDklgkdh4` | 72, 72 | 560.5 x 548.5 |
| Logo del juego | `-LBkBppHyVxDBd4mt` | 681.9, 191.8 | 547.1 x 307.8 |
| Numero `#N` | `-LBRNkqfYCvW8xyd5` | 826.3, 500.5 | 231.2 x 121.7 |
| `WANTED G.` | `-LBZS0TcM7K4m58lZ` | 955.5, 622.2 | 279.6 x 59.7 |
| Badge LIVE ▶ STREAM | `-LBMMb9d89Kk3XRl0` | 684.6, 68.8 | 514.5 x 116.7 |

Fijos, nunca se tocan: badge LIVE, marco rojo lateral, diagonales, flechas, `WANTED G.`

## Tamaños que funcionan de verdad

Los iniciales son punto de partida, no ley. Estos son los valores tras el ajuste manual sobre
portadas reales, y sirven de referencia mejor:

| Portada | Logo: posicion | Logo: tamaño | Ratio |
|---|---|---|---|
| Overwatch | 713.2, 194.2 | 475.7 x 297.8 | 1.60 |
| Smash Ultimate | 696.5, 237.6 | 526.9 x 206.8 | 2.55 |
| R.E.P.O. | 671.9, 236.6 | 567.3 x 149.7 | 3.79 |
| Combinada, logo superior | 748.8, 207.4 | 483.1 x 271.8 | 1.78 |
| Combinada, logo inferior | 757.4, 317.5 | 387.4 x 217.9 | 1.78 |
| Zelda OoT, con gancho | 745, 155 | 420 x 313.9 | 1.34 |

Lo que se deduce:

- El logo vive en una banda de **470-570 px de ancho**, centrada cerca de x=940
- **El ratio de la caja sigue al del PNG**, no al reves. Poner un logo 2.5:1 en una caja 16:9
  lo recorta por los lados y se come letras
- Los logos generados por API salieron **pequeños**; el ajuste humano los agrando ~1.7x
- Si el logo queda alto, el `#N` sube: en R.E.P.O. paso de y=500.5 a y=443.4

## Texto de gancho bajo el logo

`ctr.md` pide 2-3 palabras de gancho bajo el logo. En la geometria base **no cabe**: el logo
acaba cerca de y=500 y el `#N` esta en y=500.5. Hay que encoger el logo y subirlo. Valores
medidos con el logo de Zelda (ratio 1.34, el caso mas apretado porque llena su marco sin padding):

| Elemento | Posicion | Tamaño | Notas |
|---|---|---|---|
| Logo | 745, 155 | 420 x 313.9 | centro x = 955, acaba en y=468.9 |
| Gancho | 684, 476 | ancho 515 | 40 px, bold, `#ffffff`, centrado |
| `#N` | 877.2, 540 | — | `WANTED G.` empieza en y=622, no se solapa |

Dos cosas de la API que muerden:

- **`add_text` ignora el estilo.** Nace a 16 px, negro y alineado a la izquierda: sobre el fondo
  negro de la plantilla es invisible y parece que la operacion fallo. Hay que encadenar un
  `format_text` detras, en la misma transaccion
- **No se puede elegir `fontRef`.** El gancho sale en la fuente por defecto, distinta de la del
  `#N` (`YAFdJsF6lbU`) y la de `WANTED G.` (`YADZ-awja4Q`). En pantalla no canta, pero si se
  quiere igualar hay que hacerlo a mano en Canva

## Cover: el encuadre lo pone el humano

Las cajas de cover casi no se movieron (±10 px), pero **el encuadre interno si**. En todas las
portadas revisadas el ajuste fue acercar la imagen para llenar el marco y sacar de cuadro el
titulo impreso en la propia cover.

Ese encuadre interno **no se puede tocar por API**: `update_fill` recorta centrado y no acepta
offset. Conclusion practica: la API deja la portada al 90% y el reencuadre es un gesto humano
de diez segundos. No pelearse con esto.

## Variante de temporada: cambiar el color de marca y el fondo

Se hace sobre una **copia** de la portada ya aprobada, nunca encima. Los elementos rojos de la
plantilla y como se tocan:

| Elemento | Sufijo | Operacion |
|---|---|---|
| Panel de rayas diagonales | `-LBskq287thhtGSFS` | `recolor_element` |
| Linea izquierda / superior / inferior | `-LBGQjMW1NpgNvmwq`, `-LB0GGwj1yWcTmVnJ`, `-LBHD4kb8KtWLy9Nt` | `recolor_element` |
| Flechas de las esquinas | `-LBy2BmYVxhRs1y1M`, `-LBFqCBtDxWXVLRs5` | `recolor_element` |

El play rojo del badge LIVE no tiene mapa de recoloring: se queda rojo y no desentona.

El fondo de pagina **no se puede cambiar por API**: no hay operacion de background. Se tapa con
un `insert_fill` de 1280x720 en (0,0) con `opacity: 0.85`, que deja asomar la textura original,
seguido de `layer_element` a `back`. La imagen sale de `generate-image`, que la deja ya como
asset de Canva y evita el problema de que `upload-asset-from-url` solo acepta URLs publicas.
Prompt que funciono para Halloween: base carbon casi negra, rayas diagonales sutiles, resplandor
naranja entrando por la derecha, niebla y chispas, sin personajes ni texto.

Para adorno, `insert_shape` con paths SVG (murcielagos a 70-95 px de ancho, opacidad 0.4-0.55)
en los huecos de la columna derecha. **Nada de circulos concentricos para fingir un degradado**:
se probo con cinco circulos al 7% y salen anillos de borde duro, se ve peor que sin fondo.

## Portada a sangre: FORMATO POR DEFECTO desde 2026-11-18

Estrenada en GTA VI #1 con la portada de 3GordosB como referencia, aprobada por el usuario y
aplicada despues a la serie de Zelda. **Es el formato a usar salvo que el usuario pida otra cosa.**
El arte ocupa los 1280x720 y la caja de cover desaparece.

Pide **key art 16:9 limpio, sin titulo impreso y con el sujeto a un lado**, que deja la otra
mitad libre. Si el arte no cumple, seguir buscando antes de conformarse: el catalogo de Xbox
(`SuperHeroArt`) y las rutas `Marketing2/.../section_overview/game-keyart-*` de nintendo.com son
las dos fuentes que han dado key art limpio.

### Columna a la izquierda (sujeto a la derecha) — GTA VI

| Elemento | Posicion | Tamaño |
|---|---|---|
| Key art | 0, 0 | 1280 x 720, `layer_element` a `back` |
| Panel diagonal oscuro | 0, 0 | `insert_shape` `M0 0 L560 0 L450 720 L0 720 Z`, `#0b0b14`, opacidad 0.55 |
| Badge LIVE | 18, 22 | 460 x 104.4 |
| Gancho | 20, 142 | ancho 440, 72 px bold, `#ffffff`, centrado |
| Logo del juego | 25, 248 | 390 x 301.9 |
| `WANTED G.` | 14, 588 | 279.6 x 59.7 |
| `#N` | 298, 581 | 129.2 x 68 |
| Avatar del canal | 1208, 95 | 59.9 x 63.5 |

Ejemplo: `STREAM_2026_11_18_GTA_VI_01` (`DAHbUtw1WgE`).

### Columna a la derecha (sujeto a la izquierda) — Zelda OoT

**La columna va siempre al lado contrario del sujeto.** No es una variante opcional: es la regla.
Si el personaje esta a la izquierda, se espeja todo.

| Elemento | Posicion | Tamaño |
|---|---|---|
| Key art | -8, 0 | 1296 x 720 (arte 2880x1600 es 1.8, no 1.778: se recorta 8 px por lado) |
| Panel diagonal oscuro | 0, 0 | `M1280 0 L720 0 L830 720 L1280 720 Z` |
| Badge LIVE | 802, 22 | 460 x 104.4 |
| Gancho | 820, 132 | ancho 440, 64 px bold |
| Logo del juego | 850, 278 | 380 x 283.9 |
| `WANTED G.` | 968, 612 | 279.6 x 59.7 |
| `#N` | 832, 605 | 129.2 x 68 |
| Avatar del canal | 20, 20 | 59.9 x 63.5 |

El `WANTED G.` lleva un espacio inicial en el texto, asi que a `left` 990 se sale por la derecha
aunque la caja quepa. 968 es el valor bueno.

Ejemplos: `STREAM_2026_11_06_ZELDA_OCARINA_OF_TIME_01_FULLBLEED` (`DAHbV5llifQ`), `_02_` (`DAHbV_sv9HI`),
`_03_` (`DAHbV7wMN8w`). Los tres comparten arte: se hace uno y se copia cambiando `#N` y gancho.

### Reglas del formato

- Se borran la caja de cover, las **tres lineas rojas** (ya no enmarcan nada) y las flechas
  `>>>>>` de abajo, que sobre el arte no se leen. Se conservan badge LIVE, avatar del canal,
  flechas de esquina y panel de rayas (este se mueve al lado de la columna).
- **Las flechas de esquina tambien tapan.** Las superiores viven en (781.7, -44.9), que en GTA VI
  caia en cielo pero en Dark Souls aterrizo sobre el casco del Caballero Elegido y el usuario lo
  canto. Si el sujeto sube por el centro-derecha, moverlas a `left: 1090` (esquina de verdad, sin
  chocar con el avatar, que empieza en y=95).
- **Nada tapa una cara.** El badge LIVE en su sitio original (684, 69) le tapo los ojos a Jason y
  el usuario lo canto. En este formato el badge vive **dentro de la columna oscura**, arriba. Lo
  mismo vale para el `#N`, el avatar y el gancho: antes de dar una portada por buena, mirar donde
  cae cada elemento sobre el arte.
- Sin el panel oscuro el texto blanco se pierde: medido sobre el key art de GTA VI, que es playa
  rosa clara justo donde van el gancho y el `WANTED G.`.
- **Gancho de 2 palabras a 72 px; de 2 palabras largas, 64 px.** `add_text` no mide: a 72 px
  "EMPIEZA TODO" parte en dos lineas y se come el hueco del logo. Mirar la `height` que devuelve
  el elemento (una linea ≈ 76, dos ≈ 144) y recolocar el logo si hace falta.
- **El orden de capas es lo que mas cuesta.** `insert_fill` e `insert_shape` nacen delante de todo
  y `layer_element` solo acepta `front`/`back`, no un indice. No basta con mandar el arte a
  `back`: hay que subir a `front`, uno a uno y en orden, todo lo que deba quedar sobre el panel.
  Lo que se inserte despues del panel (logo, gancho) ya sale bien.
- **Nada toca la diagonal ni el borde inferior.** El panel se estrecha hacia abajo (110 px en
  720), asi que un logo que cabe arriba cruza abajo: su borde derecho tiene que quedar por
  debajo de `560 - 110*(y/720)` en **toda** su altura, con margen. La fila de `WANTED G.` y `#N`
  se para en y≈650: mas abajo pelea con el sello de duracion que YouTube pinta encima.
- Para un juego nuevo sale mas barato **copiar una portada a sangre ya hecha** que partir de la
  plantilla: la geometria ya esta y solo cambian arte, logo, gancho y `#N`.

## Mix de dos juegos a sangre: split vertical con costura diagonal

Elegido por el usuario el 2026-11-22 frente a dos franjas horizontales y a "un arte manda".
Los dos key art llenan el frame y cada juego conserva **cara y logo**.

| Elemento | Posicion | Tamaño |
|---|---|---|
| Key art juego 1 (base) | 0, 0 | 1280 x 720, personajes centrados en x≈640 |
| Key art juego 2 (panel) | 880, 0 | 400 x 720 (ratio 0.556) |
| Costura oscura | 0, 0 | `insert_shape` `M870 0 L950 0 L890 720 L810 720 Z`, `#0b0b14`, opacidad 1 |
| Filo rojo | 0, 0 | `M944 0 L954 0 L894 720 L884 720 Z`, `#f80a10` |
| Logo juego 1 | 25, 283 | 440 de ancho, alto segun su ratio **ya recortado** |
| Logo juego 2 | 25, 448 | 440 de ancho, idem |
| `MIX SESSIONS #N` | 12, 130 | ancho 466, **25 px**, una linea, alineado a la **derecha** (`text_align: end`), colgando del borde derecho del badge |
| `#N` suelto | — | **se borra**; el numero va en el subtitulo |

La costura tiene que **contener el borde recto del panel en toda su altura**: con el panel en
x=880, la banda va de 870-950 arriba a 810-890 abajo, asi que 880 queda dentro siempre. Si se
mueve el panel hay que recalcular los cuatro puntos o aparece la linea recta.

El panel se inserta despues del arte base, asi que nace delante de **todo**: hay que devolver a
`front` el avatar del canal y las dos flechas de esquina o desaparecen bajo el.

**Los dos logos van apilados en la columna oscura, no sobre su arte.** Pedido por el usuario el
2026-11-22 despues de ver la primera version con cada logo encima de su juego: en la columna se
leen mejor y los dos key art quedan limpios. El hueco util va del final del subtitulo (y≈160) a
la fila de `WANTED G.` (empieza en 581).

**Recortar el padding transparente del logo antes de subirlo: `&trim=1` en weserv.** Es lo que
mas hace crecer un logo en pantalla, mucho mas que estirar la caja. Los `logo.png` de Steam son
640x360 con el contenido flotando en medio: el de Gears E-Day queda en 590x155 (ratio 3.81) y el
de Overwatch en 640x72 (8.89). Con el padding puesto, una caja de 240x135 dejaba las letras
diminutas; recortados, los dos entran a **440 de ancho** y se leen a 320x180, que es el tamaño
real del grid de YouTube. Comprobar siempre a ese tamaño antes de dar la portada por buena.

**En el mix los logos son el elemento principal, no el texto.** Pedido por el usuario el
2026-11-22: `MIX SESSIONS` deja de ser gancho a 62 px en dos lineas y pasa a **subtitulo de una
linea** pegado bajo el badge, con el `#N` en la misma linea; el elemento del `#N` suelto de abajo
se borra. El tamaño elegido son **25 px** en una caja de 466 desde x=12. Se bajo en escalera con
el usuario: 48 → 40 → 36 → 25. La regla no es igualar el ancho del badge, es **ceder el
protagonismo a los logos**: en la duda, mas pequeño. A 25 px el rotulo deja de leerse a 320x180
(el tamaño del grid de YouTube) y funciona solo como marca de serie para quien abre el video;
decision consciente del usuario el 2026-11-22. Y va **alineado a la derecha**, colgando del
borde derecho del badge (x≈478), no centrado en la columna: centrado quedaba flotando en mitad
del hueco y no se leia como parte del bloque de cabecera.

Asi la columna queda con **tres pisos — badge, subtitulo, logos** — y `WANTED G.` solo al pie.
Los logos se centran verticalmente en lo que queda libre (centro ≈390): con 115.6 + 49.5 de alto
y 50 px de hueco entre ellos, el bloque arranca en y=283.

Con los logos recortados el limite pasa a ser **el ancho**, no la altura: el borde derecho no
puede cruzar la diagonal (`560 - 110*(y/720)`), y a y≈530 eso son 479, asi que 440 de ancho desde
x=25 es el maximo con margen. El trim de weserv respeta el borde si el contenido ya lo toca, asi
que el `2` de Overwatch no se pierde.

Ejemplo: `STREAM_2026_10_22_MIX_SESSIONS_11_OVERWATCH_GEARS_EDAY_FULLBLEED` (`DAHbp8vMb9A`).

## Resolucion: exportar tambien a 1920x1080

Pedido por el usuario el 2026-11-22: la prioridad al elegir arte es la **mayor resolucion
posible**, y el export acompaña. A 1280x720 el PNG sale de 1,5 MB y no deja margen; a 1920x1080
hay que ir en **JPG calidad 95**, que da ~600 KB y entra de sobra en el limite de 2 MB de
YouTube. Textos y formas de la plantilla ganan nitidez real; las imagenes solo interpolan, asi
que esto no arregla un arte pobre, solo deja de estropear el bueno.

## Portada de dos juegos: dos bandas horizontales

Cuando un directo mezcla dos juegos, la caja de cover se parte en dos bandas horizontales, y
cada logo va **sobre su propia banda**, arriba a la izquierda:

| Elemento | Posicion | Tamaño | Ratio |
|---|---|---|---|
| Banda superior | 72, 72 | 560.5 x 274.25 | 2.04 |
| Banda inferior | 72, 346.25 | 560.5 x 274.25 | 2.04 |
| Logo sobre banda superior | 73.5, 44.9 | 247.9 x 139.4 | 1.78 |
| Logo sobre banda inferior | 85, 358 | 247.9 x 139.4 | 1.78 |
| Texto `MIX SESSIONS #N` | 629.9, 309.4 | 623.9 x 84.8 | |

Ratio de banda **2.04**: aqui va **key art horizontal** (`library_hero_2x`, ratio 3.1), no la
cover 600x900. Una cover vertical en una banda 2.04 se recorta a una tira central inservible.

El ancho de caja de logo 247.9 x 139.4 es ratio 1.78, que es justo el de los `logo.png` de Steam
(640x360). Si el logo tiene otro ratio, hay que recalcular la caja, no reusarla.

Ojo: para cambiar el tamaño hay que **borrar** el elemento original e insertarlo de nuevo con
`insert_fill`, porque `resize_element` recorta en vez de escalar.

Ejemplo de referencia: `STREAM_2026_09_04_MIX_SESSIONS_07_MONSTER_HUNTER_WILDS_ROCKET_LEAGUE`
(`DAHUST4csqA`), copia directa de la del MIX #6. **Copiar la portada mix anterior sale mas
barato que partir de la plantilla base**: la geometria ya esta hecha y solo cambian fills y texto.

## Portada de tres juegos: bandas horizontales

Con tres juegos **no se parte en vertical**. Tres tercios de 187 x 548 tienen ratio 0.34 y el
recorte centrado de `update_fill` deja una tira inservible: un key art 2:1 se ve al 17% de su
ancho. Se parte en tres bandas horizontales:

| Banda | Posicion | Tamaño |
|---|---|---|
| Superior | 72, 72 | 560.5 x 182.85 |
| Media | 72, 254.85 | 560.5 x 182.85 |
| Inferior | 72, 437.7 | 560.5 x 182.85 |

Ratio de banda **3.07**. Los headers de Steam son 2.14 y los key art de temporada 2:1, asi que
el recorte se queda con el 65-70% central en altura y no toca los lados. Arte vertical aqui no
sirve.

**El logo va encima de su propia banda**, no en la columna derecha: pedido por el usuario el
2026-08-05. La columna derecha queda entera para el texto. Valor medido tras el ajuste manual,
logo de Overwatch sobre la banda superior:

| Elemento | Posicion | Tamaño | Ratio |
|---|---|---|---|
| Logo sobre banda superior | 63.1, 27.0 | 247.9 x 154.5 | 1.60 |

Con la columna derecha libre, el elemento del `#N` crece y se centra para el rotulo
`MIX SESSIONS #N`:

| Elemento | Posicion | Tamaño |
|---|---|---|
| Texto `MIX SESSIONS #N` | 686.3, 260.9 | 529.6 x 259.3 |

Se sale un poco de la banda por arriba y por la izquierda, y esta bien: pisa el marco y da
profundidad. Los juegos cuyo arte ya trae el titulo impreso (headers de Steam) **no llevan
logo**; solo lo necesita el que va con key art limpio, que en la practica es el juego de
servicio.
