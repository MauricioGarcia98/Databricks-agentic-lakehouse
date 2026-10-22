# 00 · Verificar qué trae tu Free Edition

## 🎯 Objetivo

Completar la columna **"¿está en Free Edition?"** de la [arquitectura](../docs/01_arquitectura.md) con lo que ves
**en tu workspace hoy**, no con lo que dice un video o un blog. El video muestra Databricks con todo prendido; Free
Edition tiene cuotas y algunas piezas pueden no estar, o estar en *preview*.

**Entregable:** la matriz completa (✅ está · ⚠️ está con límites · ❌ no está) y una fila en la bitácora por cada ❌
con el plan B elegido.

## 📚 Conceptos

- **Free Edition** es un workspace serverless gratuito con cuota diaria de cómputo. No elegís clusters: todo es
  serverless (notebooks, jobs, pipelines y un SQL warehouse chico).
- **Verificar la cuenta** (lo aprendiste en Andes Bank) habilita el acceso saliente a internet. Sin eso, no llegás a
  las APIs del BCRA ni de Georef.
- **Preview ≠ no existe.** Muchas piezas aparecen en *Settings → Previews* y hay que prenderlas.
- **Probalo, no lo leas.** Para cada pieza, la prueba es crear algo mínimo y borrarlo.

## 🧱 Paso a paso

### 1. Cuenta y previews

1. Entrá a tu workspace de Free Edition (o creá uno nuevo para este proyecto).
2. Revisá que la cuenta esté verificada.
3. Abrí tu usuario → **Settings → Previews** y anotá en la bitácora qué previews ves relacionados con: Genie,
   metric views, dominios / ontology / pages, AI Gateway / Unity Gateway, MCP, Apps, Genie Code. Prendé los que
   vas a usar.

### 2. Catálogo (cómputo + Unity Catalog)

En un notebook o en el SQL editor:

```sql
CREATE CATALOG IF NOT EXISTS andes_farma;
```

- Si funciona → ✅, el catálogo se llama `andes_farma`.
- Si falla → ⚠️, usás el catálogo `workspace` con esquemas prefijados. Anotalo: cambia todos los nombres de la guía.

Después, en el catálogo que te quedó:

```sql
CREATE SCHEMA IF NOT EXISTS prueba;
CREATE VOLUME IF NOT EXISTS prueba.raw;
```

### 3. Internet saliente (para las fuentes reales)

En un notebook serverless:

```python
import requests
r = requests.get("https://apis.datos.gob.ar/georef/api/provincias", timeout=20)
print(r.status_code, r.json()["cantidad"])
```

Y probá la API de estadísticas del BCRA: `https://api.bcra.gob.ar/estadisticas/v4.0/monetarias` (la v3.0 ya
devuelve **410**: si alguna guía vieja te la pasa, es por eso).

- 200 en las dos → ✅.
- Error de conexión → la cuenta no está verificada o el dominio está bloqueado. Plan B: bajás los JSON en tu máquina
  y los subís al Volume.

### 4. Ingesta: Jobs y pipelines (SDP)

- **Jobs & Pipelines → Create → Job**: creá un job con una tarea de notebook vacío y corrélo. ✅ si termina verde.
- **Create → ETL pipeline**: creá un pipeline con un archivo SQL que tenga
  `CREATE MATERIALIZED VIEW prueba_mv AS SELECT 1 AS x;` y corrélo.

### 5. Semántica: metric views

```sql
CREATE TABLE prueba.t AS SELECT * FROM VALUES ('RTA', 100.0), ('MOS', 50.0) AS t(canal, importe);

CREATE OR REPLACE VIEW prueba.mv_prueba
WITH METRICS
LANGUAGE YAML
AS $$
version: 1.1
source: prueba.t
dimensions:
  - name: canal
    expr: canal
measures:
  - name: venta_total
    expr: SUM(importe)
$$;

SELECT canal, MEASURE(venta_total) FROM prueba.mv_prueba GROUP BY canal;
```

`version: 1.1` funciona (probado el 2026-10-22).

### 6. Semántica: dominios, pages y ontology

Buscá en la barra lateral y en **Catalog**: *Discover*, *Domains*, *Pages*, *Ontology*. Si existen, creá un dominio
`prueba` y una page `Prueba`. Si no existen, ❌ y plan B: glosario en tabla + comentarios en UC + instrucciones del
Genie space.

### 7. Consumo: Genie, Genie One y Genie Code

- **Genie → New**: creá un space sobre `prueba.mv_prueba` y preguntá "¿cuánto vendió cada canal?".
- Buscá **Genie One** (un chat que no pide elegir space). Anotá si está.
- Abrí el asistente en un notebook (**Genie Code / Databricks Assistant**) y pedile "explicame esta celda".
- Buscá en el space la pestaña de **Benchmarks**.

### 8. Acceso: Gateway y MCP

- **Serving**: anotá qué endpoints de modelos fundacionales ves (pay-per-token) y si podés abrir la configuración de
  **AI Gateway / Unity Gateway** (guardrails, rate limits, inference tables, usage tracking). Probá activar un
  guardrail; anotá exactamente qué te deja y qué no.
- **MCP**: buscá en la sección de *Agents* o *MCP servers* la URL del **MCP gestionado de Genie** para tu space.
  Anotala; la usás en la etapa 14 con Claude Code.

### 9. Apps y costos

- **Compute → Apps**: ¿podés crear una app? ¿Cuántas?
- `SELECT * FROM system.billing.usage LIMIT 10;` → si responde, vas a poder medir la cuota que usa cada corrida.

### 10. Limpiá

```sql
DROP SCHEMA prueba CASCADE;
```

Borrá el job, el pipeline y el Genie space de prueba.

## 🧪 Resultado de la corrida por API (2026-10-22)

Claude Code corrió los pasos 2 a 9 contra tu workspace con el SDK y después borró todo (queda sólo el catálogo
`andes_farma`, vacío). El resultado está en la matriz de [01 · Arquitectura](../docs/01_arquitectura.md#lo-que-encontró-la-etapa-00-2026-10-22).
**Te falta mirar en la UI** lo que no tiene API pública: dominios, pages, snippets/ontology, Genie One, Genie Code,
la pestaña Benchmarks de un space y *Settings → Previews*.

## ✅ Cómo sabés que terminaste

- No queda ningún ⬜ en la matriz de [01 · Arquitectura](../docs/01_arquitectura.md).
- Cada ❌ tiene su plan B elegido en la bitácora.
- Sabés el nombre del catálogo, la URL del BCRA que funciona y la URL del MCP de Genie.

## 📝 Para la bitácora y el posteo

- Esta matriz **es** la respuesta a la pregunta 9 del posteo ("¿hasta dónde llegué con Free Edition?"). Sacale
  captura.
- Anotá la fecha: lo que no está hoy puede aparecer en un mes, y está bueno decirlo en el post.
