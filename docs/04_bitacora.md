# 04 · Bitácora

> Una fila por decisión, error o corrección. Es la materia prima de la pregunta 3 del posteo ("qué decidí yo, qué
> pedí, qué corregí"). Anotalo **en el momento**: una semana después ya no te acordás.

| # | fecha | etapa | tipo | qué pasó | qué hice / qué decidí |
|---|---|---|---|---|---|
| B01 | | 00 | decisión | elegí la industria | farmacia, por datos sensibles + obras sociales + inflación |
| B02 | 2026-10-22 | 00 | decisión | Free Edition deja crear catálogos | el catálogo es `andes_farma` (no `workspace`) |
| B03 | 2026-10-22 | 00 | límite | BCRA v3.0 responde 410 (deprecada) | se usa la v4.0 |
| B04 | 2026-10-22 | 00 | límite | no hay modelos Claude en los endpoints de Free Edition | Claude Code se conecta al Genie por MCP; el gateway gobierna un modelo de Databricks |
| B05 | 2026-10-22 | 00 | límite | el guardrail de PII no detecta el DNI argentino | máscara propia por regex en silver + column mask |
| B06 | 2026-10-22 | 00 | límite | `valid_topics` / `invalid_keywords` se descartan sin error | guardrail de temas con `ai_classify` |
| B07 | 2026-10-22 | 00 | límite | inference tables no soportadas en endpoints fundacionales | usage tracking + tablas de sistema |
| B08 | 2026-10-22 | 00 | aprendizaje | Genie tradujo "Ruta Andes" a `RTA` sólo por el comentario de la metric view | los comentarios son semántica: se escriben en serio |
| B09 | 2026-10-22 | 00 | pedido | le pedí a Claude Code que corriera las pruebas por API | las corrió y borró todo; yo verifico en la UI lo que no tiene API |

**Tipos:** `decisión` (la tomé yo) · `pedido` (se lo pedí a la IA) · `corrección` (la IA propuso algo y lo cambié) ·
`aprendizaje` (algo que entendí) · `límite` (Free Edition no me dejó) · `genie` (una respuesta mala y su arreglo).
