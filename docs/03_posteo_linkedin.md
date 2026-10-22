# 03 · El posteo de LinkedIn (preguntas guía)

> Se responden **al final**, con números reales. Mientras construís, anotá respuestas sueltas en la
> [bitácora](04_bitacora.md): una respuesta equivocada de Genie, una decisión que cambiaste, un límite de Free Edition.

## Las preguntas centrales

1. **¿Qué proyecto hice?** Una cadena de farmacias argentina simulada, Andes Farma, donde un agente responde
   preguntas de negocio sobre datos gobernados en Databricks Free Edition: Lakeflow, metric views, Genie, MCP y
   guardrails, siguiendo la arquitectura de *agentic analytics* que presentó Databricks.
2. **¿Por qué lo hice?** Porque tener tablas gold no alcanza: quería ver qué hace falta para que un agente responda
   **bien**, y exprimir Free Edition con eso.
3. **¿Cómo usé la IA?** Claude Code como acompañante, y también como **cliente** del proyecto (se conecta al Genie por
   MCP). Qué decidí yo, qué le pedí, qué corregí y qué aprendí de sus explicaciones. No "la IA lo hizo".
4. **¿Qué aprendí de Databricks?**

## Para darle más valor

5. ¿Por qué una **farmacia** y no otra industria? (datos de salud sensibles, obras sociales que debitan, inflación,
   vencimientos, canales con nombre propio)
6. ¿Cuál fue el **problema inyectado más difícil** de resolver y cómo lo resolví? (tickets reenviados, devoluciones
   negativas, códigos de plan que cambian, débitos)
7. ¿Qué diferencia hubo entre el agente **sin semántica y con semántica**? (mismo Genie, mismas tablas: X% → Y%)
8. ¿Qué significa **"venta"** en Andes Farma y cuántas definiciones distintas encontré? (bruta, neta, cobrada, a pesos
   constantes)
9. ¿Hasta dónde llegué con **Free Edition** y qué piezas del video no estaban? (la matriz de la etapa 00 y sus planes B)
10. ¿Cuánto me costó? (**USD 0**, y cuánto de la cuota diaria de cómputo consumió una corrida)
11. ¿Cómo protegí los **datos de salud** aunque fueran sintéticos? (masks, row filters, el agente que no ve silver)
12. ¿Qué pregunta **bloquea** el guardrail y por qué? ("top 10 farmacéuticos", "qué compra el cliente X")
13. ¿Qué le puede preguntar un **gerente** a Genie? ¿Y un **farmacéutico de sucursal**? ¿Y un **data engineer** con
    Genie Code?
14. ¿Dónde se **equivocó Genie** y cómo lo arreglé? (casi siempre era el modelo de datos o una definición faltante, no
    las instrucciones)
15. ¿Cómo **medí** que responde bien? (benchmarks, % de aciertos por dominio)
16. ¿Qué cambió al consultar el mismo Genie desde **Claude Code por MCP**? (misma respuesta, mismo gobierno)
17. ¿Qué **decisión de arquitectura** cambié a mitad de camino y por qué?
18. ¿Qué **número** muestra la escala? (filas, sucursales, días simulados, preguntas del benchmark, tiempo de corrida)
19. ¿Qué haría **distinto en una farmacia real**? (ANMAT, receta electrónica, trazabilidad de medicamentos)
20. ¿Qué parte **no** le delegaría nunca a una IA? (las definiciones de negocio: qué es "venta" lo decide el negocio)
21. ¿Cómo lo puede **reproducir** otra persona? (repo y guía `construir/`)
22. ¿Qué **sigue**? (compras, forecast de demanda, alertas de vencimiento, una App con chat)

## Material para juntar mientras construís

- Captura de la **misma pregunta** antes y después de la semántica (etapas 09 y 12).
- Captura de Genie mostrando que usó `mv_ventas` y la definición de Ruta Andes.
- Captura de Claude Code respondiendo por MCP.
- Captura del guardrail bloqueando "top 10 farmacéuticos".
- El diagrama de arquitectura (la diapositiva del video, con tus piezas).
- La tabla "problema inyectado → dónde se resuelve".
- La matriz de Free Edition (etapa 00).
