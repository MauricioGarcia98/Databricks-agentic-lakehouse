# 00 · Definición del proyecto

> **Estado:** propuesta. Cuando la apruebes, cualquier cambio se anota en la [bitácora](04_bitacora.md).

## En una frase

**Andes Farma** es una cadena de farmacias argentina ficticia cuyo lakehouse en **Databricks Free Edition** convierte
ventas, dispensas a obras sociales e inventario en una **capa semántica gobernada** (metric views, dominios, pages),
para que un **agente** (Genie, Genie One o Claude Code por MCP) responda preguntas de negocio **con la misma
definición que usaría el gerente**, y se niegue a responder lo que no corresponde.

## Por qué existe

1. **Reproducir el video de Databricks con mis manos**, en una industria distinta a la del demo (e-commerce) y con
   las herramientas que da gratis Free Edition.
2. **Pasar de "tengo tablas gold" a "un agente responde bien"**: la parte que casi ningún tutorial muestra.
3. **Medirlo**: benchmarks de Genie con porcentaje de aciertos antes y después de agregar semántica.
4. **Documentar los límites de Free Edition** para la arquitectura del video: qué está, qué no, y el plan B.
5. **Contarlo** en LinkedIn ([03](03_posteo_linkedin.md)) con una guía para reproducirlo (`construir/`).

## ¿Por qué una farmacia?

Es el equivalente de "Brickcommerce" del video, pero con problemas que un e-commerce no tiene:

| lo que trae una farmacia | por qué sirve para *agentic analytics* |
|---|---|
| **Datos de salud** (qué medicamento compró quién, con qué receta) | son **datos sensibles** para la Ley 25.326: obliga a enmascarar y a que el agente se niegue a ciertas preguntas |
| **Obras sociales** que pagan una parte y después **debitan** | conciliación: "venta" tiene tres definiciones (bruta, neta de descuento, cobrada) → el riesgo de *exactitud* del video |
| **Inflación**: listas de precios que cambian cada semana | "¿vendimos más?" en pesos o en unidades: el agente tiene que saber cuál usar (fuente real: BCRA) |
| **Lotes y vencimientos** | inventario con reglas de negocio no obvias (FEFO, quiebre de stock) |
| **Canales con nombre propio** (Ruta Andes) | el ejemplo perfecto de *Page*: un nombre que un LLM no puede adivinar |
| **Performance individual** (ventas por farmacéutico) | el ejemplo del guardrail del video: "¿quiénes son los 10 peores farmacéuticos?" se bloquea |

**Alternativas que descarté** (si preferís una, se cambia acá y en la bitácora):

- *Logística de última milla*: buena para streaming, pero pobre en semántica.
- *Club de fútbol / ticketing*: divertido para el posteo, pero sin datos sensibles reales ni conciliación.

## Los dominios (subdominios del video)

| dominio | pregunta típica | tabla gold principal | metric view |
|---|---|---|---|
| **Ventas** | ¿cómo evolucionó la venta neta real de Ruta Andes? | `fact_ventas` | `mv_ventas` |
| **Obras sociales** | ¿qué obra social nos debita más y por qué? | `fact_dispensas_os` | `mv_obras_sociales` |
| **Inventario** | ¿en qué sucursales hay quiebre de los 20 productos top? | `fact_stock_diario` | `mv_inventario` |
| **Compras** *(después del MVP)* | ¿qué droguería nos entrega tarde? | `fact_ordenes_compra` | `mv_compras` |

## Las fuentes

| fuente | real o sintética | cómo llega |
|---|---|---|
| Tickets del punto de venta (POS) de 40 sucursales | sintética, con problemas inyectados | archivos JSON en un Volume → Auto Loader |
| Pedidos de la app y de Ruta Andes | sintética | archivos JSON en un Volume → Auto Loader |
| Dispensas y liquidaciones de obras sociales | sintética, con débitos | CSV mensual en un Volume |
| Stock diario por sucursal y lote | sintética | snapshot diario (Parquet) |
| Inflación (CER/IPC) y tipo de cambio | **real**: API pública del BCRA | Python → bronze |
| Provincias, departamentos y localidades | **real**: API Georef (datos.gob.ar) | Python → `dim_sucursal` |

## Problemas inyectados (lo que el pipeline y el agente tienen que resistir)

| problema | dónde aparece | dónde se resuelve |
|---|---|---|
| El POS reenvía el mismo ticket al reconectarse | bronze ventas | silver: dedupe por `ticket_id` + hash |
| Devoluciones como cantidad negativa en el mismo ticket | bronze ventas | silver: separar `es_devolucion`; la métrica decide |
| EAN con dígito verificador inválido | bronze ventas/stock | silver: expectation + cuarentena |
| Una obra social cambia el código de un plan a mitad de año | bronze OS | silver: mapeo de códigos (SCD2) |
| La obra social paga menos de lo facturado (débito) | liquidaciones | gold: `importe_debitado`; la métrica "venta cobrada" |
| Una sucursal cambia de región comercial | maestro sucursales | `dim_sucursal` SCD2 |
| Notas libres con DNI o diagnóstico | pedidos Ruta Andes | silver: `ai_mask` / regex + column mask |
| Venta de un lote vencido | stock/ventas | gold: flag + alerta |
| Precios nominales con inflación del 3% mensual | todo lo que está en pesos | metric view: medida "a pesos constantes" con CER |

## Alcance

**MVP:** Ventas + Obras sociales + Inventario · 3 metric views · 3 Genie Agents · Genie One (o su plan B) ·
Claude Code conectado por MCP · un guardrail que bloquea preguntas de performance individual y de salud de un cliente ·
benchmarks.

**Después:** Compras · Databricks App con chat · forecast de demanda con `ai_forecast` · alertas de vencimiento.

**Fuera de alcance:** datos reales de personas (todo es sintético, con semilla fija) · alta disponibilidad · ML
productivo.

## Decisiones tomadas (propuestas)

| tema | decisión | por qué |
|---|---|---|
| Industria | farmacia (Andes Farma) | ver arriba |
| Plataforma | Databricks Free Edition, **verificada** (acceso saliente a internet) | lo aprendiste en Andes Bank: sin verificación no llegás a las APIs |
| Nube externa | **ninguna** | este proyecto no es de ingesta; el costo cero es parte del mensaje |
| Estructura | `docs/` + `construir/` por etapas + `genie/` + bundle | la misma de los proyectos anteriores |
| Forma de trabajo | **lo construís vos**; Claude Code explica, revisa y propone; vos decidís y lo anotás en la bitácora | el posteo cuenta qué decidiste vos |
| Idioma | castellano rioplatense en docs; nombres de tablas en castellano sin tildes | Genie responde mejor si la tabla se llama como la pregunta |
