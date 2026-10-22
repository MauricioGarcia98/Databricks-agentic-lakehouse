# 01 · Arquitectura: del video a Free Edition

> La diapositiva del video tiene cuatro capas (ingesta, semántica, acceso y consumo) sobre un cómputo común. Abajo,
> cada caja con lo que vas a usar en Free Edition. **La columna "¿está en Free Edition?" salió de la
> [etapa 00](../construir/00_verificar_free_edition.md)** (probado por API el 2026-10-22; lo marcado ⬜ sólo se ve en la UI): varias piezas del video (Genie One, Ontology, Unity
> Gateway) son recientes y su disponibilidad en Free Edition cambia. Por eso cada una tiene un plan B.

## Las tres preguntas que guían todo (los tres riesgos del video)

| riesgo | la pregunta | qué lo resuelve en este proyecto |
|---|---|---|
| **Exactitud** | ¿el agente sabe que "venta" es venta neta y que Ruta Andes es el canal `RTA`? | metric views + pages + comentarios en UC + benchmarks |
| **Acceso** | ¿el agente se niega a mostrar qué compra un cliente o cuánto vende un farmacéutico? | column masks, row filters, guardrails del gateway, MCP con identidad del usuario |
| **Escala** | ¿aguanta muchas preguntas sin romper la cuota diaria? | serverless SQL warehouse, tablas gold chicas y agregadas, rate limits |

## Capa por capa

| capa del video | pieza | en Andes Farma | ¿está en Free Edition? | plan B |
|---|---|---|---|---|
| **Ingesta** | Lakeflow Connect | — (no hay SaaS que conectar) | — | Auto Loader sobre Volumes |
| | Lakeflow Jobs | job diario: generar → ingerir → transformar → refrescar | ✅ serverless | — |
| | Lakeflow SDP (Spark Declarative Pipelines) | bronze → silver → gold con *expectations* | ✅ serverless | — |
| **Semántica** | Metric views | `mv_ventas`, `mv_obras_sociales`, `mv_inventario` | ✅ YAML `version: 1.1` | vistas SQL + *SQL expressions* del Genie space |
| | Dominios y subdominios | dominio `andes_farma` → ventas, obras sociales, inventario | ⬜ verificar en la UI | un esquema por dominio + tags `dominio=` en UC |
| | Pages (glosario gobernado) | *Ruta Andes*, *Venta neta*, *Cliente activo*, *Quiebre de stock*, *Débito de OS* | ⬜ verificar en la UI | tabla `semantica.glosario` + comentarios en UC + instrucciones del space |
| | Snippets aprendidos | lo que Genie infiere del uso | ⬜ verificar en la UI | *SQL queries* de ejemplo y *trusted assets* en el space |
| **Acceso** | Unity Catalog | grupos, grants, masks, row filters, linaje | ✅ catálogo propio, tags, column masks | — |
| | Unity Gateway | endpoint de modelo con guardrails, rate limits e inference tables | ⚠️ AI Gateway: usage tracking, rate limits, PII y safety ✅ · temas permitidos ❌ · inference tables ❌ · sin modelos Claude | guardrail de temas con `ai_classify` (✅ probado) en una función UC o en la App |
| | MCP governance | MCP gestionado de Genie (y de funciones UC) | ✅ `/api/2.0/mcp/genie/{space_id}` | — |
| **Consumo** | Genie Agents | uno por dominio | ✅ (creados por API) | — |
| | Genie One | preguntas que cruzan dominios ("ventas de Ruta Andes con débito de OS") | ⬜ verificar en la UI | un space "Andes Farma general" con las 3 metric views, o un agente en una App que enruta |
| | Genie One MCP / 3P agents | **Claude Code** conectado por MCP | ⚠️ el MCP de un space ✅; el de Genie One, verificar | MCP de un Genie space puntual |
| | Agents on Apps / Agent Framework | App con chat (después del MVP) | ✅ Apps | — |
| | Genie Code | el asistente para escribir pipelines y notebooks | ⬜ verificar en la UI | Databricks Assistant |
| **Cómputo** | Lakehouse | serverless (notebooks, jobs, SQL warehouse) | ✅ | — |

## El catálogo

```
andes_farma                 (o workspace.<esquema> si Free Edition no te deja crear catálogos)
├── raw         Volumes con los archivos que deja el generador
├── bronze      tal cual llegó + metadatos de ingesta
├── silver      limpio, deduplicado, tipado, con PII enmascarada
├── gold        modelo estrella: fact_ventas, fact_dispensas_os, fact_stock_diario, dim_*
├── semantica   metric views, glosario, funciones UC para el agente
└── ops         calidad, corridas, benchmarks, logs del gateway
```

## Quién ve qué (grupos)

| grupo | ve | no ve |
|---|---|---|
| `gerencia` | todo gold y semántica, agregado | DNI, receta, diagnóstico; ventas por farmacéutico |
| `region_cuyo` (ejemplo) | sus sucursales (row filter) | el resto |
| `data_eng` | bronze → gold | PII en claro (sólo enmascarada) |
| `agentes` (service principal del MCP) | semántica + gold | silver, bronze, PII |

> En Free Edition sos el único usuario. Igual creás los grupos y probás los filtros con `is_account_group_member`
> sumándote y sacándote del grupo: alcanza para demostrarlo.

## Lo que encontró la etapa 00 (2026-10-22)

- **Internet saliente ✅** desde serverless: Georef y BCRA responden. La API de estadísticas del BCRA vigente es la
  **v4.0** (`/estadisticas/v4.0/monetarias`); la v3.0 devuelve **410 Gone**. Cambiarias: `/estadisticascambiarias/v1.0`.
- **Modelos:** 12 endpoints fundacionales pay-per-token (Llama 3.x/4, GPT-OSS, Qwen, Gemma, DeepSeek y embeddings).
  **No hay Claude.** El "UG Claude" del video no se puede replicar tal cual: Claude Code entra por **MCP** al Genie,
  y el modelo detrás del gateway es uno de los de Databricks.
- **Guardrail de PII ⚠️:** bloquea, pero no reconoce el **DNI argentino** (`30.123.456` pasó sin anonimizar; el
  teléfono sí lo detectó). En silver el DNI se enmascara con regex propia, no con el guardrail.
- **Guardrail de temas ❌:** la API acepta `valid_topics` e `invalid_keywords` pero **los descarta sin avisar**. El
  bloqueo de "top 10 farmacéuticos" se hace con `ai_classify` (lo clasificó bien como `performance_individual`).
- **Inference tables ❌** para endpoints fundacionales en Free Edition. Plan B: usage tracking + `system.serving`.
- **Genie** usó la metric view sola y tradujo "Ruta Andes" a `canal = 'RTA'` **sólo por el comentario de la
  dimensión**: la semántica en UC ya cambia la respuesta.
