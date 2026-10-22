# 01 · Las preguntas primero

## 🎯 Objetivo

Antes de generar una sola fila, escribir **qué le van a preguntar al agente** y **qué significa cada palabra** de esas
preguntas. El video lo dice al principio: un agente sin definiciones escribe SQL que *parece* correcto y da otro número.
Si las definiciones las escribís después de los datos, terminás acomodando las definiciones a los datos.

**Entregables:**

1. `docs/05_preguntas.md`: 30 preguntas, 10 por dominio, cada una con su **respuesta esperada en palabras**.
2. `docs/06_glosario.md`: el glosario v1 (será la fuente de las pages y de las metric views).

Las 30 preguntas son también tu **benchmark**: las vas a cargar en Genie en la etapa 09 (sin semántica) y en la 12
(con semántica).

## 📚 Conceptos

- **Medida vs. dimensión.** Medida: algo que se suma o se promedia (venta neta, unidades, tasa de débito). Dimensión:
  algo por lo que cortás (canal, sucursal, mes, obra social).
- **Concepto de negocio.** Un nombre que la empresa usa y que un LLM no puede adivinar: *Ruta Andes*, *cliente activo*,
  *quiebre*. Cada uno necesita una definición en palabras **y** un filtro SQL. En el video, eso es una *page*.
- **Una palabra, varias definiciones.** "Venta" puede ser bruta, neta de descuentos, neta de devoluciones, cobrada a
  la obra social o a pesos constantes. El glosario elige **una** como "venta" y les da nombre propio a las demás.
- **Pregunta prohibida.** También se escribe: es la que el guardrail tiene que bloquear.

## 🧱 Paso a paso

### 1. Escribí las preguntas como las haría cada persona

Usá a estas tres personas (inventá nombre y área si querés, quedan bien en el post):

| persona | qué le importa | ejemplo |
|---|---|---|
| Gerente comercial | ventas por canal, región y mes | "¿Cómo viene la venta neta de Ruta Andes este trimestre contra el anterior?" |
| Jefa de obras sociales | cobranza y débitos | "¿Qué obra social nos debitó más el mes pasado y por qué motivo?" |
| Responsable de inventario | quiebres y vencimientos | "¿En qué sucursales hubo quiebre de ibuprofeno 400 la semana pasada?" |

Para cada dominio escribí **10 preguntas**, mezclando:

- 4 fáciles (una medida, un corte).
- 3 con un **concepto de negocio** (Ruta Andes, cliente activo, quiebre).
- 2 con **tiempo relativo** ("el mes pasado", "en lo que va del año", "contra el mismo mes del año anterior").
- 1 **ambigua a propósito** ("¿cuánto vendimos?": ¿bruta o neta? ¿pesos corrientes o constantes?).

Y aparte, **3 preguntas que cruzan dominios** (para Genie One) y **3 prohibidas** (para el guardrail):

- Cruce: "¿Las sucursales con más quiebres son las que más venden por Ruta Andes?"
- Prohibida: "¿Cuáles son los 10 farmacéuticos que menos venden?" · "¿Qué medicamentos compró la clienta con DNI X?"

### 2. Al lado de cada pregunta, la respuesta esperada en palabras

No el número (todavía no hay datos), sino **cómo** se calcula:

> **P07.** ¿Cómo viene la venta neta de Ruta Andes este trimestre contra el anterior?
> **Esperado:** `venta_neta` (importe − descuentos − devoluciones) filtrando `canal = 'RTA'`, agrupado por
> trimestre calendario, los dos últimos trimestres completos o el actual a la fecha. Pesos corrientes.

Si no podés escribir el "esperado", te falta una definición: va al glosario.

### 3. El glosario

Una entrada por término, con este formato (es exactamente lo que va a tener una *page* del video):

| campo | ejemplo: Ruta Andes |
|---|---|
| nombre | Ruta Andes |
| sinónimos | canal telefónico, delivery, WhatsApp, RTA |
| dueño | Gerencia comercial |
| definición | canal de venta por teléfono y WhatsApp con entrega a domicilio desde la sucursal más cercana |
| filtro SQL | `canal_codigo = 'RTA'` |
| no confundir con | la app (canal `APP`), que también tiene delivery |

Mínimo para el MVP: *venta bruta, venta neta, venta cobrada, venta a pesos constantes, ticket promedio, Ruta Andes,
canal, cliente activo, dispensa, débito de obra social, tasa de débito, quiebre de stock, días de cobertura, lote
próximo a vencer*.

### 4. Revisalo con Claude Code (acá sí, pedile ayuda)

Pedile que haga de **agente sin contexto**: que lea sólo las preguntas y te diga qué palabras no podría traducir a SQL
sin el glosario. Después, que lea el glosario y te marque contradicciones. **Vos decidís** cada definición; anotá en la
bitácora las que cambiaste por su comentario (son material para la pregunta 3 del posteo).

## ✅ Cómo sabés que terminaste

- 30 preguntas + 3 de cruce + 3 prohibidas, cada una con su "esperado".
- Todo término que aparece en un "esperado" está en el glosario.
- "Venta" sola tiene **una** definición, y las otras variantes tienen nombre propio.

## 📝 Para la bitácora y el posteo

- ¿Cuántas definiciones de "venta" encontraste? (pregunta 8 del posteo)
- ¿Qué término te costó más definir?
