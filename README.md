# 💊 Andes Farma · Agentic analytics E2E en Databricks Free Edition

> **Estado: definición.** Todavía no hay código. Primero acordamos qué se construye; después lo construís vos,
> etapa por etapa, siguiendo `construir/`.

Andes Farma es una **cadena de farmacias argentina ficticia** (40 sucursales, venta en mostrador, app y un canal
telefónico con delivery llamado **Ruta Andes**). Este repo construye su **fundación de *agentic analytics*** de punta
a punta en **Databricks Free Edition**, siguiendo la arquitectura del video
[*Agentic Analytics Foundation*](https://youtu.be/ZuLAAdIjeeE) de Databricks:

```
PRODUCTORES ─► INGESTA ─────────► SEMÁNTICA ─────────► ACCESO ─────────────► CONSUMO
  datos        Lakeflow            Metric views          Unity Catalog          Genie Agents (por dominio)
  sintéticos   (Jobs + SDP)        Dominios y Pages      Gateway (guardrails)   Genie One (cruza dominios)
  + APIs                           Snippets aprendidos   MCP gobernado          Claude Code vía MCP
  públicas                                                                      Genie Code (DE/DS)
                         └──────────── cómputo: Lakehouse serverless ────────────┘
```

Es el tercer proyecto de la familia, después de
[Andes Viento Austral](https://github.com/MauricioGarcia98/andes-viento-austral) y
[Andes Bank](https://github.com/MauricioGarcia98/andes-bankin-datbricks). La diferencia: **el foco no es la ingesta,
es que un agente responda bien**. Los tres riesgos del video (exactitud, acceso y escala) son la vara con la que se
mide el proyecto.

## Empezá por acá

| doc | de qué trata |
|---|---|
| [00 · Definición del proyecto](docs/00_definicion_del_proyecto.md) | qué es, por qué una farmacia, alcance y decisiones |
| [01 · Arquitectura](docs/01_arquitectura.md) | cada caja del video → qué se usa en Free Edition y el plan B si no está |
| [02 · Plan de construcción](docs/02_plan_de_construccion.md) | las etapas de `construir/`, con su entregable |
| [03 · Posteo de LinkedIn](docs/03_posteo_linkedin.md) | las preguntas para contar el proyecto |
| [04 · Bitácora](docs/04_bitacora.md) | decisiones, errores y correcciones (la materia prima del posteo) |
| [construir/](construir/README.md) | la guía paso a paso |

## Tope de costo

**USD 0.** Todo corre dentro de Databricks Free Edition; las fuentes reales son APIs públicas gratuitas (BCRA y
Georef). Sin Azure.
