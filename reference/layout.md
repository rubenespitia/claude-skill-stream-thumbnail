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

Lo que se deduce:

- El logo vive en una banda de **470-570 px de ancho**, centrada cerca de x=940
- **El ratio de la caja sigue al del PNG**, no al reves. Poner un logo 2.5:1 en una caja 16:9
  lo recorta por los lados y se come letras
- Los logos generados por API salieron **pequeños**; el ajuste humano los agrando ~1.7x
- Si el logo queda alto, el `#N` sube: en R.E.P.O. paso de y=500.5 a y=443.4

## Cover: el encuadre lo pone el humano

Las cajas de cover casi no se movieron (±10 px), pero **el encuadre interno si**. En todas las
portadas revisadas el ajuste fue acercar la imagen para llenar el marco y sacar de cuadro el
titulo impreso en la propia cover.

Ese encuadre interno **no se puede tocar por API**: `update_fill` recorta centrado y no acepta
offset. Conclusion practica: la API deja la portada al 90% y el reencuadre es un gesto humano
de diez segundos. No pelearse con esto.

## Portada de dos juegos

Cuando un directo mezcla dos juegos, la caja de cover se parte en dos mitades:

| Elemento | Posicion | Tamaño |
|---|---|---|
| Cover izquierda | 72, 72 | 280.3 x 548.5 |
| Cover derecha | 352.3, 72 | 280.3 x 548.5 |

Y los dos logos se apilan en la banda derecha. Ojo: hay que **borrar** los elementos originales
e insertarlos de nuevo con `insert_fill`, porque `resize_element` recorta en vez de escalar.
