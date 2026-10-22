# 01 · Arquitectura: del video a Free Edition

> La diapositiva del video tiene cuatro capas (ingesta, semántica, acceso y consumo) sobre un cómputo común. Abajo,
> cada caja con lo que vas a usar en Free Edition. **La columna "¿está en Free Edition?" la completás vos en la
> [etapa 00](../construir/00_verificar_free_edition.md)**: varias piezas del video (Genie One, Ontology, Unity
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
| | Lakeflow Jobs | job diario: generar → ingerir → transformar → refrescar | ⬜ | — |
| | Lakeflow SDP (Spark Declarative Pipelines) | bronze → silver → gold con *expectations* | ⬜ | notebooks con `MERGE` |
| **Semántica** | Metric views | `mv_ventas`, `mv_obras_sociales`, `mv_inventario` | ⬜ | vistas SQL + *SQL expressions* del Genie space |
| | Dominios y subdominios | dominio `andes_farma` → ventas, obras sociales, inventario | ⬜ | un esquema por dominio + tags `dominio=` en UC |
| | Pages (glosario gobernado) | *Ruta Andes*, *Venta neta*, *Cliente activo*, *Quiebre de stock*, *Débito de OS* | ⬜ | tabla `semantica.glosario` + comentarios en UC + instrucciones del space |
| | Snippets aprendidos | lo que Genie infiere del uso | ⬜ | *SQL queries* de ejemplo y *trusted assets* en el space |
| **Acceso** | Unity Catalog | grupos, grants, masks, row filters, linaje | ⬜ | — |
| | Unity Gateway | endpoint de modelo con guardrails, rate limits e inference tables | ⬜ | AI Gateway en el endpoint; si no, guardrail propio con `ai_classify` en la App |
| | MCP governance | MCP gestionado de Genie (y de funciones UC) | ⬜ | — |
| **Consumo** | Genie Agents | uno por dominio | ⬜ | — |
| | Genie One | preguntas que cruzan dominios ("ventas de Ruta Andes con débito de OS") | ⬜ | un space "Andes Farma general" con las 3 metric views, o un agente en una App que enruta |
| | Genie One MCP / 3P agents | **Claude Code** conectado por MCP | ⬜ | MCP de un Genie space puntual |
| | Agents on Apps / Agent Framework | App con chat (después del MVP) | ⬜ | — |
| | Genie Code | el asistente para escribir pipelines y notebooks | ⬜ | Databricks Assistant |
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
