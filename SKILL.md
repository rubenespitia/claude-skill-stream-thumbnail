---
name: stream-thumbnail
description: Genera portadas de directo de gaming 1280x720 con la API de Canva, duplicando una plantilla y rellenando cover del juego, logo y numero de episodio. Usar cuando el usuario pida "portada", "miniatura", "thumbnail" de un directo o stream, o cuando haya que numerar episodios cruzando el historial del canal de YouTube.
---

# Portadas de directo

Duplicar un diseño de Canva y rellenarlo por API. **No usa brand templates**: requieren
Canva Enterprise y la cuenta tipica no los tiene. `search-brand-templates` devolviendo
`{"items":[]}` no es un error, es lo normal.

## Ciclo por portada

1. `copy-design` de la plantilla, indicando la pagina que sirve de base
2. `start-editing-transaction` sobre la copia → devuelve `page_id` y los `element_id`
3. `perform-editing-operations`: `update_fill` de la cover, `update_fill` del logo,
   `replace_text` del `#N`, `update_title`
4. `commit-editing-transaction` (sin esto se pierde todo)
5. `export-design` a PNG 1280x720, `lossless: true`, `export_quality: "pro"`
6. Descargar la URL del export; caduca en horas

Los sufijos de `element_id` son identicos en toda copia de la misma pagina; solo cambia el
prefijo, que es el `page_id`. Asi que basta con leerlos una vez y componer:
`{page_id}-{sufijo}`.

## Reglas de contenido

- **Cover oficial del juego** en la caja grande, no un frame del video. El frame no aporta y
  ademas no se puede subir: `upload-asset-from-url` solo acepta URLs ya publicas.
- **Juego de servicio** (GTA Online, Overwatch, Marvel Rivals, Fortnite…): arte de la
  temporada o evento vigente **en la fecha del directo**, no la cover perenne.
- **Overwatch**: solo season, nunca evento, y con personaje visible. Es una excepcion pedida
  explicitamente.
- **Juego cerrado** (Pokemon, Smash, Barony, R.E.P.O.…): cover perenne, no cambia.
- Recortar el titulo del juego al encuadrar no importa: el logo ya va en su propia caja.

## Numeracion de episodios

El `#N` sale de cruzar dos fuentes:

- historial de directos del canal (`youtube.com/@CANAL/streams`, leer `ytInitialData` en el
  browser; los items son `lockupViewModel`, no `videoRenderer`)
- los `#N` que ya usan las portadas anteriores

Se numera por juego y en orden cronologico. Si un juego encadena temporadas propias
(Overwatch S1 #01…#06, S2 #07), el siguiente continua la cuenta, no reinicia.

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

## Referencias

- `reference/layout.md` — geometria exacta de cada hueco y los tamaños que funcionan
- `reference/asset-sources.md` — de donde sacar covers, logos y arte de temporada
- `reference/canva-api.md` — orden de llamadas, errores y limites
- `assets/games.json` — registro de juegos ya resueltos, con sus URLs
