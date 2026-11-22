# De donde salen covers, logos y arte de temporada

Restriccion que manda sobre todo lo demas: `upload-asset-from-url` de Canva **solo acepta URLs
ya publicas**. No se puede subir un archivo local, ni recortar una imagen antes de subirla. Asi
que solo sirve el arte que ya viene limpio y encuadrable.

## Juegos en Steam: la via buena

Por `appid`, sin API key ni registro:

```
https://cdn.cloudflare.steamstatic.com/steam/apps/<appid>/library_600x900_2x.jpg   # cover 600x900
https://cdn.cloudflare.steamstatic.com/steam/apps/<appid>/logo.png                 # logo transparente
```

**Usar siempre `_2x`.** La version sin sufijo es 300x450 y se ve blanda al escalar a 560.

El `logo.png` de Steam viene con transparencia limpia y es lo que mejor funciona en la caja
derecha. Los logos que no salen de Steam suelen traer fondo pegado o venir en version oscura.

## Juegos recien salidos: el CDN nuevo con hash

Los lanzamientos recientes ya no sirven por las rutas de arriba: `library_600x900_2x.jpg` y
`logo.png` dan **404**. Sus assets viven en rutas con hash por archivo:

```
https://shared.fastly.steamstatic.com/store_item_assets/steam/apps/<appid>/<hash>/header.jpg
```

El hash se saca del HTML de la ficha o de `appdetails`:

```bash
curl -s "https://store.steampowered.com/api/appdetails?appids=<appid>&filters=basic"
```

Ahi solo hay `header_image` (460x215), `capsule_image` (231x87) y `capsule_imagev5`. **No hay
cover vertical ni logo transparente**, asi que un juego asi obliga a bandas horizontales y a
usar arte que ya traiga el titulo impreso. Comprobado con Machine Party (appid 4108000).

Buscar el appid: `steamcommunity.com/actions/SearchApps` y `store.steampowered.com/api/storesearch`
devuelven vacio para juegos de semanas. Mas rapido buscarlo en la web, que ademas corrige el
nombre si el usuario lo dijo de memoria.

## Juegos de Xbox cuyo arte de Steam no sirve

Hay fichas de Steam donde todo el arte es el logo sobre fondo negro, sin personaje: no cumplen
la regla por mucho que las URLs respondan 200. El catalogo de la Microsoft Store tiene el key
art de verdad y es publico:

```bash
curl -s "https://www.microsoft.com/msstoreapiprod/api/autosuggest?market=en-us&sources=DCatAll-Products&filter=%2BClientType%3AStoreWeb&query=<juego>"   # da el BigCatalogId
curl -s "https://displaycatalog.mp.microsoft.com/v7.0/products?bigIds=<BIGID>&market=US&languages=en-us"
```

Las imagenes salen con `ImagePurpose`. Las que importan: `BoxArt` (cuadrada, suele ser el key
art completo con titulo), `Poster` (2:3) y `SuperHeroArt` (16:9). `store-images.s-microsoft.com`
es publico y Canva lo lee. Comprobado con Gears of War: E-Day (`9N4PT8HGCDHQ`).

## Juegos de Nintendo y otros sin Steam

`mario.wiki.gallery` (Super Mario Wiki) funciona y Canva lo lee. Se consulta por API:

```bash
curl -s "https://www.mariowiki.com/api.php?action=query&format=json&generator=search&gsrnamespace=6&gsrlimit=8&gsrsearch=intitle%3A%22SSBU%22%20box&prop=imageinfo&iiprop=url|size"
```

**`upload.wikimedia.org` NO funciona**: el fetcher de Canva recibe distinto de 200. Wikipedia
sirve para localizar el nombre del archivo, pero hay que buscar el mismo arte en otro host.

## Juegos de servicio: arte de temporada

Noticias de Steam, publica y con fecha por post, que es lo que permite elegir el arte vigente
en la fecha del directo:

```bash
curl -s "https://api.steampowered.com/ISteamNews/GetNewsForApp/v2/?appid=<appid>&count=12"
```

Las imagenes vienen como `{STEAM_CLAN_IMAGE}/...` y se resuelven con
`https://clan.akamai.steamstatic.com/images/`. El host de cloudflare devuelve 301 en esa ruta.

**No fiarse de `library_hero.jpg`**: no tiene fecha y puede llevar meses sin actualizarse. El de
Overwatch tenia arte de Halloween en pleno julio.

## Overwatch en concreto

Necesita season, no evento, y con personaje. Lo unico que cumple las tres cosas es el `og:image`
del articulo de la season en `overwatch.blizzard.com/news/`:

```
https://bnetcmsus-a.akamaihd.net/cms/blog_header/<hash>.jpg    # 880x440, limpio, personaje centrado
```

WebFetch no devuelve las imagenes de esa web: hay que abrir la pagina en el browser y leer
`document.querySelector('meta[property="og:image"]').content`.

Descartados y por que:

| Fuente | Problema |
|---|---|
| `library_hero.jpg` de Steam | Arte viejo sin fecha |
| `background_raw` del store | Season correcta pero sin personajes |
| Miniatura del trailer en YouTube | El titulo cae justo donde recorta Canva |
| Arte del livestream de la season | Lleva fecha y "DEVELOPER LIVESTREAM" impresos |

## Recortar antes de subir: images.weserv.nl

`update_fill` recorta centrado y no acepta encuadre, pero el recorte se puede hacer **antes** de
subir a Canva pasando la URL publica por un proxy de imagen:

```
https://images.weserv.nl/?url=<url sin https://>&w=1232&h=402&fit=cover&a=focal&fpx=0.5&fpy=0.40
```

`w`/`h` fijan el ratio exacto de la banda (3.07) y `fpy` sube o baja la ventana hasta que el
titulo impreso entre entero. Esto solo reencuadra; **para acercarse de verdad** hace falta
recortar una region concreta del original con `&precrop=true&cx=&cy=&cw=&ch=`. Sin
`precrop=true` el recorte se aplica **despues** del resize y lo que sale es la esquina del
thumbnail ya reducido, no la region pedida. Asi sirven los `capsule_616x353.jpg`, que llevan titulo pero son
ratio 1.75 y con el recorte centrado de Canva pierden la fila de arriba de las letras.

Las URLs de `assets.nintendo.com` son Cloudinary y aceptan sus propias transformaciones
(`c_fill,w_,h_`), pero el `c_crop` encadenado no respeta la altura pedida: sale otro ratio. Mas
fiable pasarlas tambien por weserv con `fit=cover`.

Nada de esto publica nada: solo reencuadra imagenes que ya son publicas.

## Nombrado de assets en Canva

`cover-<juego>-2x`, `logo-<juego>`, `keyart-<juego>-<evento>-<AAAA-MM>`. Con fecha en el nombre
cuando el arte caduca, para que se vea de un vistazo cual esta viejo. Canva no borra assets por
API, asi que subir de mas ensucia para siempre.
