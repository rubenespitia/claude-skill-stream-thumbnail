# CTR: que rinde y que no

Medido en YouTube Studio del canal WantedGull el 2026-11-06, rango 1 ene – 5 nov 2026.
Canal entero: 3.054 vistas, 92.663 impresiones, **1,0% de CTR**.

## El dato

CTR ponderado por impresiones, solo videos con **>= 1.000 impresiones** (por debajo de eso el
porcentaje son cuatro o cinco clics y no significa nada):

| Grupo | n | Impresiones | CTR |
|---|---|---|---|
| Videos editados (reviews, ensayos) | 6 | 26.002 | **2,18%** |
| Directos (VOD) | 14 | 47.984 | **0,35%** |

Factor 6. Y los directos acumulan **mas impresiones que los editados**: YouTube si los esta
repartiendo, no se convierten.

Sin el filtro de 1.000 impresiones: editados 2,18% (n=7), directo de un juego 0,50% (n=25),
MIX SESSIONS 0,34% (n=9).

## Lo que el dato NO dice

- **El `#N` no es el problema.** MIX (0,22%) contra un juego numerado (0,39%) en el tramo
  >= 1k, pero ahi MIX son solo 2 videos. No es base para quitar la numeracion. La brecha que
  importa es directo contra editado, y la cruzan las dos familias por igual.
- **No separa portada de titulo.** Los directos llevan titulos que son chistes internos sin
  juego ni gancho ("Compes De Suuusto", "Lone Ranger o algo asi", "Truco o Trabuco") y duran
  tres horas. Los editados llevan titulo descriptivo. Fallan juntos y no se sabe en que
  proporcion.
- **Halloween no movio nada.** Los tres con impresiones altas dan 0,2-0,3%, igual o peor.

## Trampa al leer la tabla de Studio

Los CTR mas altos del listado son los mas enga�osos: 13,5%, 6,8% y 5,9% salen de videos con
37-90 impresiones. **Filtrar siempre por impresiones antes de comparar CTR.** Un video recien
subido tampoco cuenta: a las 24 h tenia 54 impresiones.

## La regla, para los directos siguientes

Portada y titulo se dise�an **como pareja**: la portada abre la curiosidad, el titulo la cierra.
Nunca repetir lo mismo en los dos.

1. **Titulo que empiece por el juego**, no por el chiste. El chiste va detras si cabe.
   `ZELDA OCARINA OF TIME | ...` antes que `DALE ZELDA DALE!!! | ZELDA...`
2. **2-3 palabras de gancho en la portada**, en la columna derecha debajo del logo. Lo
   publicado da ~6,8% de CTR con 0-2 palabras contra 4,3% con 6 o mas: el limite es por arriba,
   no por abajo.
3. **El `#N` se queda** donde esta. No hay dato que justifique quitarlo.
4. La plantilla fija no se toca: el reconocimiento de serie es lo unico que funciona hoy.

Para aislar el efecto de la portada sola, YouTube tiene *Probar y comparar* (tres miniaturas
por video). El titulo no entra en ese test.

## Como volver a medirlo

Extension Claude in Chrome, con Chrome reiniciado despues de instalarla. Despues:

1. `studio.youtube.com` → Estadisticas → **Modo Avanzado**
2. En el selector de fecha de la izquierda, elegir el a�o (no "ultimos 28 dias")
3. Las metricas *Impresiones de la miniatura* y *Porcentaje de clics de las miniaturas* ya
   vienen en la vista de Modo Avanzado; no hay que a�adirlas
4. Leer la tabla del DOM, que es mas barato que exportar el CSV:

```js
[...document.querySelectorAll('yta-explore-table-row')].map(r => {
  const c = r.innerText.split('\n').filter(s => s.trim());
  return [c[1], c[2], c[10], c[11]].join('\t');   // titulo, vistas, impresiones, CTR
});
```

El `javascript_tool` trunca la respuesta cerca de los 1.200 caracteres: guardar el array en
`window.__d` y leerlo en tandas de ~18 filas.

**No construir la URL de `/explore` a mano.** Quitarle el `entity_id` deja la pagina en negro.
Entrar pulsando *Modo Avanzado* y cambiar el rango desde el desplegable.
