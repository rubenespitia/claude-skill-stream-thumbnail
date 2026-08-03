# Canva MCP: orden de llamadas y trampas

## Secuencia minima

```
copy-design(design_id, page_numbers=[N])          -> nuevo design_id
start-editing-transaction(design_id)              -> transaction_id, page_id, element_ids
perform-editing-operations(transaction_id, ops)   -> borrador, devuelve thumbnail
commit-editing-transaction(transaction_id)        -> guarda
export-design(design_id, png 1280x720)            -> URL temporal
```

`cancel-editing-transaction` descarta. Sin commit **se pierde todo**, aunque las operaciones
hayan devuelto `success`.

Se puede leer el estado de un diseño abriendo una transaccion y cancelandola: es la unica forma
de ver geometria de elementos, porque `get-design-content` solo devuelve texto.

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
