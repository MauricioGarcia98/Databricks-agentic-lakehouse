# 02 · Plan de construcción

Cada etapa es un archivo en `construir/`, termina con un **entregable que se puede ver** y con un commit + tag
`etapa-NN`. No pasás a la siguiente sin el entregable.

| etapa | qué hacés | entregable | capa del video |
|---|---|---|---|
| **00** | Verificar qué trae tu Free Edition | la matriz de [01](01_arquitectura.md) completa | — |
| **01** | Las preguntas primero: qué le van a preguntar al agente y qué significa cada palabra | 30 preguntas + glosario v1 | semántica |
| **02** | Repo, bundle, catálogo, esquemas y Volumes | `databricks bundle deploy` funciona | cómputo |
| **03** | Generador sintético con problemas inyectados | archivos en `raw` con semilla fija | productores |
| **04** | Fuentes reales: BCRA y Georef | `bronze.bcra_series`, `bronze.georef_localidades` | productores |
| **05** | Bronze con Auto Loader | tablas bronze idempotentes | ingesta |
| **06** | Silver con SDP y *expectations* | cuarentena con los problemas inyectados | ingesta |
| **07** | Gold: modelo estrella | facts y dims documentadas | ingesta |
| **08** | Gobierno: comentarios, tags, PII, masks, row filters | una consulta que muestra el mask funcionando | acceso |
| **09** | **Genie "sin semántica"** + benchmark base | % de aciertos **antes** | consumo |
| **10** | Metric views | 3 metric views certificadas | semántica |
| **11** | Dominios y pages (o glosario) | Ruta Andes definida en un lugar | semántica |
| **12** | Genie Agents por dominio + benchmark | % de aciertos **después** | consumo |
| **13** | Genie One (cruce de dominios) | "ventas de Ruta Andes con débitos de OS" responde bien | consumo |
| **14** | MCP: Claude Code consulta el Genie | la misma respuesta desde la terminal | acceso / 3P agents |
| **15** | Gateway y guardrails | "top 10 farmacéuticos" bloqueado y logueado | acceso |
| **16** | Job diario, costos y observabilidad | job verde + consulta a `system.billing.usage` | cómputo |
| **17** | Posteo | el post + capturas | — |

**La idea fuerza del posteo** sale de las etapas 09 y 12: *el mismo Genie, las mismas tablas, X% → Y% de aciertos
sólo agregando semántica*. Por eso el benchmark base va **antes** de las metric views.
