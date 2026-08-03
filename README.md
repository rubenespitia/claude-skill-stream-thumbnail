# stream-thumbnail

Skill de Claude Code para generar portadas de directo de gaming (1280x720) con la API de Canva.

Duplica un diseño que ya existe en tu cuenta y le rellena tres cosas: la cover del juego, el
logo y el numero de episodio. Exporta el PNG listo para subir a YouTube.

No usa brand templates. Requieren Canva Enterprise, y esta skill nace justamente de que en una
cuenta normal esa ruta no existe.

## Que necesitas

- Claude Code con el MCP de Canva conectado:
  ```bash
  claude mcp add --transport http canva https://mcp.canva.com/mcp
  claude mcp login canva
  ```
- Un diseño en Canva que sirva de plantilla, con un hueco de imagen para la cover, otro para el
  logo y un texto para el numero de episodio
- Opcional: `yt-dlp` y `ffmpeg`, para identificar el juego de un directo por sus subtitulos

## Instalar

```bash
git clone <este-repo> ~/.claude/skills/stream-thumbnail
```

O copiar la carpeta a `~/.claude/skills/`. Claude la carga sola cuando pidas una portada.

## Adaptarla a tu plantilla

Los `element_id` y coordenadas de `reference/layout.md` son de una plantilla concreta. Para la
tuya:

1. Pide a Claude que abra una transaccion de edicion sobre tu diseño y liste los elementos
2. Identifica cual es la cover, cual el logo y cual el texto del numero
3. Sustituye los sufijos y las coordenadas en `reference/layout.md`

Lo demas (fuentes de arte, orden de llamadas, trampas de la API) vale igual para cualquier
plantilla.

## Contenido

| Archivo | Que hay |
|---|---|
| `SKILL.md` | El flujo y las reglas de contenido. Es lo que Claude lee siempre |
| `reference/layout.md` | Geometria de los huecos y tamaños que funcionan |
| `reference/asset-sources.md` | De donde sacar covers, logos y arte de temporada |
| `reference/canva-api.md` | Orden de llamadas, operaciones y trampas |
| `assets/games.json` | Registro de juegos ya resueltos con sus URLs |

## Lo que la API no hace

El encuadre dentro del marco. `update_fill` recorta centrado y no acepta offset, asi que la
portada sale al 90% y el ultimo ajuste de zoom es manual, unos segundos en Canva. Esta asumido
en el diseño de la skill: no intenta pelearse con eso.

## Licencia

MIT. El arte de los juegos es de sus dueños; la skill solo apunta a URLs publicas oficiales.
