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

## Nombrado de assets en Canva

`cover-<juego>-2x`, `logo-<juego>`, `keyart-<juego>-<evento>-<AAAA-MM>`. Con fecha en el nombre
cuando el arte caduca, para que se vea de un vistazo cual esta viejo. Canva no borra assets por
API, asi que subir de mas ensucia para siempre.
