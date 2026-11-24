# Publicar el VOD de un directo en Studio

Todo lo que hay que dejar puesto en la repeticion de un stream, en orden. Campos verificados en
YouTube Studio en español el 2026-11-22.

La portada **no se sube hasta que el usuario la aprueba** (ver `SKILL.md`). El resto de campos
se pueden preparar antes.

## 0. Elegir el video correcto

Cada directo genera **dos entradas**, horizontal y vertical, con el mismo titulo y 2-3 segundos
de diferencia. Se trabaja siempre sobre la **horizontal**; la vertical va a privado. No se
distinguen por el orden del listado ni por la miniatura: abrir el reproductor y leer el tamaño
real del video.

```js
const v = document.querySelector('video');
({ w: v.videoWidth, h: v.videoHeight })   // 854x480 = horizontal, 480x854 = vertical
```

Los directos nuevos pueden salir ya en 1920x1080 y sin pareja vertical; comprobar antes de
buscar un gemelo que no existe.

## 1. Titulo

- **Si la serie tiene episodios, el titulo acaba en `| JUEGO #N`.** Pedido por el usuario el
  2026-11-22. Va al final, detras del titulo que el escribio, con el mismo `#N` que la portada:
  `🌴 GTA VI y Saliendo del Rancho🕺🏎️ | GTA VI #2`. Se repite el nombre del juego aunque ya
  aparezca al principio del titulo: lo que identifica la serie es el bloque final entero.
- Empieza por el **juego**, no por el chiste (`ctr.md`). El chiste va detras.
- Mix: termina en `| MIX SESSIONS #N`, misma regla.
- El `#N` **no se saca del titulo de YouTube**, que falla: dos directos distintos llegaron a
  llamarse los dos `MIX SESSIONS #10`. Manda la cuenta de las portadas anteriores.
- Editar: hay dos `div#textbox` en la pagina; el primero es el titulo y el segundo la
  descripcion. Para cambiar solo el numero, `ctrl+End` + `Backspace` + teclear, que es mas
  seguro que reescribir un titulo con emojis.
- Al acabar en `#algo` se abre el desplegable de hashtags sugeridos: cerrar con `Escape` antes
  de guardar.
- **Enfocar el campo por `ref`, no por coordenada.** Un clic en las coordenadas del titulo puede
  no enfocar nada: lo que se teclee despues se pierde sin error y el titulo queda igual. Sacar
  el `ref` del textbox con `find` y clicar sobre el. Despues de teclear, releer el campo antes
  de guardar.

## 2. Miniatura

Formato a sangre, un juego o mix segun el directo (`layout.md`). Exportar en **JPG 1920x1080
calidad 95** (~600 KB, el limite de YouTube son 2 MB).

**No clicar el boton "Subir archivo"**: abre el selector nativo del sistema y deja la sesion
colgada. Hay que subir el archivo directamente al input oculto:

```js
// localiza el input; su id es "file-loader", accept="image/jpeg,image/png"
const walk=(r,out)=>{r.querySelectorAll('*').forEach(e=>{if(e.tagName==='INPUT'&&e.type==='file')out.push(e);if(e.shadowRoot)walk(e.shadowRoot,out);});return out;};
walk(document,[])
```

Luego `file_upload` con el ref de ese input.

Si el PNG anterior sigue abierto en un visor de Windows, `curl -o` sobre el mismo nombre falla
con **exit 23**: descargar con un nombre nuevo, no pelearse con el bloqueo.

## 3. Descripcion

Plantilla, capitulos y links fijos en `description.md`. Dos trampas que ya rompieron videos:

- **`H:MM:SS` siempre** en un directo de horas. `0:28` lo lee YouTube como 28 segundos y los
  capitulos se apelotonan al principio sin dar ningun error.
- **Si el campo esta vacio, `ctrl+a` se escribe como la letra `a`** y la descripcion queda
  empezando por `aFinal de...`. Comprobar el principio del texto antes de guardar.

## 4. Etiquetas

Bloque *Etiquetas*, dentro de **Mostrar mas** (boton "Mostrar configuración avanzada"). Input
`#text-input`, placeholder *Añade una etiqueta*, una coma detras de cada una.

El propio Studio avisa de que influyen muy poco salvo para erratas de busqueda: poner el nombre
del juego, sus variantes y poco mas. No es donde esta el partido.

## 5. Categoria y titulo del juego

Tambien en **Mostrar mas**:

| Campo | Valor |
|---|---|
| Categoria | `Videojuegos` |
| Titulo del juego (opcional) | el juego del directo |

El campo del juego es un **combobox con sugerencias**, no texto libre: hay que escribir y
**elegir una entrada de la lista** (sale con su año, p. ej. `Overwatch 2 2022`). Si no se
selecciona, se queda vacio.

**Solo admite un juego.** En un MIX SESSIONS hay que elegir el que mas minutos se jugo, o
dejarlo vacio; no hay forma de poner dos.

## 6. Listas de reproduccion

Bloque *Listas* → boton **"Seleccionar listas de reproducción"** → marcar la del **año en
curso** (`2026 STREAMING`, id en `description.md`) → Hecho.

Paso fijo, no opcional: la propia descripcion enlaza esa lista, y si el video no esta dentro, el
enlace manda al espectador a una lista donde no esta lo que acaba de ver. En enero toca lista
nueva.

## 7. Guardar y verificar

**El boton Guardar necesita dos clics si vienes de escribir en un campo.** El primero solo
retira el foco del campo y no guarda nada: el boton sigue activo y, si se intenta navegar, salta
el dialogo nativo de *"¿Salir del sitio?"* que bloquea la sesion del navegador. Dar el segundo
clic, esperar un par de segundos y comprobar que aparece el aviso:

```js
document.body.innerText.includes('Se han guardado los cambios')
```

Despues **recargar la pagina de detalles** y releer los campos:

```js
const walk=(r,out)=>{r.querySelectorAll('*').forEach(e=>{if(e.getAttribute&&e.getAttribute('contenteditable')==='true')out.push(e);if(e.shadowRoot)walk(e.shadowRoot,out);});return out;};
const e=walk(document,[]);
JSON.stringify({titulo:e[0].innerText, desc_len:(e[1]||{innerText:''}).innerText.length})
```

El grid de `/streams` **cachea la miniatura vieja** un buen rato: comprobar ahi no vale, hay que
mirar la pagina de detalles.

Y la trampa que ya pico una vez: **corregir la portada en Canva y exportar no es subirla.** Tras
cualquier arreglo hay que volver a Studio y subir el archivo nuevo.

## Descartar sin guardar

Para probar algo en el formulario y dejarlo como estaba, el boton **"Deshacer cambios"** revierte
todo lo pendiente. Mejor que recargar, que puede lanzar un dialogo nativo de "¿salir del sitio?"
y bloquear la sesion del navegador.

## 8. Archivar

Las portadas y los `.txt` que se dan por buenos se guardan en:

```
C:\Users\ruben\OneDrive\Imágenes\THUMBNAILS\STREAMS\AAAA_MM_DD\
```

La carpeta lleva la **fecha de la sesion de trabajo**, no la del directo; dentro pueden convivir
varios directos. Los archivos conservan la **fecha de su propio directo** en el nombre, con la
convencion de siempre:

```
STREAM_AAAA_MM_DD_JUEGO[_FULLBLEED].jpg|png
DESC_AAAA_MM_DD_JUEGO.txt
```

Se archiva **solo la version final**, con nombre limpio: nada de `_v9` ni `_1920`. Las
intermedias se quedan en Descargas.

## Lo que no se toca

Remixes en Shorts, comentarios y puntuaciones, publico (contenido para niños), licencia,
restriccion de edad: se quedan como estan salvo que el usuario lo pida.

## Checklist

- [ ] Es la entrada **horizontal** (la vertical, a privado)
- [ ] Titulo empieza por el juego y **acaba en `| JUEGO #N`** si la serie tiene episodios, con
      el mismo numero que la portada
- [ ] Portada a sangre **aprobada por el usuario** y subida (JPG 1920x1080)
- [ ] Descripcion con capitulos en `H:MM:SS`, minimo tres, el primero `0:00`
- [ ] Etiquetas
- [ ] Categoria `Videojuegos` + titulo del juego elegido de la lista
- [ ] Lista de reproduccion del año en curso
- [ ] Guardado y **verificado recargando** la pagina de detalles
- [ ] Portada y `.txt` archivados en `THUMBNAILS\STREAMS\AAAA_MM_DD\`
