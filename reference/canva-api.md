# Canva MCP: orden de llamadas y trampas

## Secuencia minima

```
copy-design(design_id, page_numbers=[N])                      -> nuevo design_id
read-design(design_id, open_transaction=true)                 -> transaction_id, page_id, elements
edit-design(transaction_id, page_index, ops, "keep_open")     -> borrador, devuelve thumbnail
edit-design(transaction_id, "commit")                         -> guarda
export-design(design_id, png 1280x720)                        -> URL temporal
```

`finalize: "cancel"` descarta. Sin commit **se pierde todo**, aunque las operaciones hayan
devuelto `applied_unverified`.

`commit` y `cancel` no admiten operaciones en la misma llamada: hay que mandarlas antes con
`keep_open`.

**La transaccion caduca.** Con ~25 min entre el `read-design` y el `commit` la llamada falla con
`Editing transaction <id> not found` y se pierde todo el trabajo. Abrir y cerrar de una tacada:
nada de muestrear frames, buscar arte o descargar entre medias. Si hay que investigar, antes.

Para ver geometria de elementos hace falta `open_transaction: true`; un `read-design` normal
solo devuelve el texto. No pasar `filter.element_ids: []`, que filtra todo y deja la lista de
elementos vacia; con un sufijo de verdad (`["LBRNkqfYCvW8xyd5"]`) si filtra bien, y es la forma
barata de sacar el `page_id` de una copia sin tragarse el documento entero.

Los sufijos de `element_id` son los mismos en toda copia de la misma pagina, asi que basta con
componer `{page_id}-{sufijo}` y no hace falta releer los elementos de cada copia.

## Operaciones utiles

| Operacion | Para que | Ojo |
|---|---|---|
| `update_fill` | cambiar la imagen de un hueco | recorta **centrado**, sin control de encuadre |
| `replace_text` | cambiar el `#N` | el texto original suele traer un espacio inicial |
| `update_title` | renombrar el diseño | |
| `delete_element` | quitar un elemento | |
| `insert_fill` | meter imagen nueva con posicion y tamaño | **si escala** |
| `position_element` / `resize_element` | mover / redimensionar | `resize_element` **recorta**, no escala |

La distincion entre `resize_element` e `insert_fill` es la trampa principal. Para que una imagen
se vea entera a otro tamaño: borrar e insertar.

Lo que entra con `insert_fill` o `insert_shape` **aparece delante de todo**. Un fondo a pagina
completa necesita siempre un `layer_element` con `position: "back"` despues, y como el
locator_id solo llega en la respuesta, son dos llamadas.

`update_stroke_properties` sobre los elementos `line` de la plantilla devuelve
`internal_error`. Para cambiarles el color funciona `recolor_element`, que tambien sirve en los
rect con mapa de `recoloring` (flechas, panel de rayas diagonales).

`update_fill` **hereda el recorte que el humano dejo puesto en ese hueco**. Si la portada de la
que se copia fue reencuadrada a mano, el asset nuevo entra con ese mismo zoom y sale cortado
aunque el ratio de la caja coincida con el del PNG. Se vio con el logo de Overwatch, que quedo
como "VERWATCH" sin el "2". En logos, `delete_element` + `insert_fill` en vez de `update_fill`.

## Brand templates

`search-brand-templates` devuelve `{"items":[]}` en cuentas que no son Enterprise. No es un
fallo de permisos que se pueda arreglar. Los IDs de brand template empiezan por `BTM`; los de
diseño por `DAG`/`DAH`. Son espacios distintos: un design_id nunca aparecera en esa lista.

## Assets

- `upload-asset-from-url`: solo URLs publicas. Nunca publicar un archivo privado en un hosting
  para sortear esto.
- `get-assets`: da dimensiones reales, necesarias para calcular la caja del logo con el ratio
  correcto.
- No hay borrado de assets por API.

## Organizacion

`move-item-to-folder` acepta diseños e imagenes. Conviene separar:

- una carpeta para **portadas terminadas**
- otra para **materia prima** (covers, logos, key arts)

Convencion de titulo: `STREAM_AAAA_MM_DD_JUEGO`.

## Exports

`export-design` requiere que el formato este soportado; comprobarlo con `get-export-formats`
antes si hay duda. Para portada de YouTube: `png`, `width: 1280`, `height: 720`,
`lossless: true`, `export_quality: "pro"`. Las URLs caducan en horas, asi que descargar en el
momento.
