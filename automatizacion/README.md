# Automatización: Formulario → Google Sheets → Make → Claude → Docs / Gmail

Esta carpeta conecta el departamento de marketing con **Make** y la **API de Claude** para lanzar propiedades sin abrir Claude Code: alguien llena un formulario (o una fila en Google Sheets) y recibe un correo con los tres textos ya escritos y revisados.

Es la versión automática de la skill `lanzar-propiedad`. Las reglas son las mismas (inglés para el texto final, Fair Housing, nunca inventar datos, revisión obligatoria), pero metidas dentro de los prompts, porque la API no puede leer los archivos del proyecto.

El flujo usa **dos escenarios de Make**. Sus blueprints (exportados de Make) están en esta carpeta:

| Escenario en Make | Blueprint | Qué hace |
|-------------------|-----------|----------|
| **Formulario a Hoja** | `blueprint-formulario-a-hoja.json` | Copia cada respuesta nueva del formulario a la hoja de propiedades, con `Status = READY`. |
| **Integration Google Sheets** (escenario principal) | `blueprint-escenario-principal.json` | Toma una fila `READY`, llama al redactor y al revisor, crea el Google Doc, actualiza la fila y envía el correo. |

## El flujo completo

```
Google Forms        Make: "Formulario a Hoja"         Google Sheets (Hoja 1)
────────────        ─────────────────────────         ──────────────────────
Respuesta     →    1. Watch rows (respuestas)    →   Fila nueva, Status = READY
                   2. Add row (Hoja 1)                (o fila escrita a mano: DRAFT → READY)
                                                                │
                    Make: escenario principal                   ▼
                    ─────────────────────────
                    1.  Search rows: Status = READY (1 fila)
                    5.  Status = PROCESSING
                    9.  Claude — redactor   ──→  prompt-redactor-api.md
                    10. Parse JSON (redactor)
                    11. Claude — revisor    ──→  prompt-revisor-api.md
                    12. Parse JSON (revisor)
                    13. Crea el Google Doc  ──→  carpeta de borradores en Drive
                    14. Status = DONE + Doc link
                    15. Envía el correo     ──→  Gmail al revisor humano
        (9–12 fallan) → ruta de error: escribe el motivo en Error
```

Los números son los IDs de los módulos dentro del blueprint, para encontrarlos rápido en Make.

### Escenario 1 — "Formulario a Hoja"

1. **Formulario.** Un Google Form guarda sus respuestas en la pestaña `Respuestas de formulario 1` de la misma hoja de cálculo.
2. **Watch rows (módulo 1).** Make lee las respuestas nuevas, hasta **5 por ejecución**.
3. **Add row (módulo 2).** Agrega cada respuesta a la pestaña `Hoja 1` (la hoja de propiedades creada con `plantilla-propiedades.csv`):
   - `ID` = `F` + número de fila de la respuesta (por ejemplo `F7`), para distinguir las filas que vienen del formulario.
   - `Status` = `READY` directamente: estas filas **no pasan por `DRAFT`**, así que el escenario principal las procesa en su siguiente ejecución.
   - Las columnas C a Q de la respuesta se copian a las columnas C a Q de la hoja (de `Property type` a `Distances`). La columna B de la respuesta no se copia. Por eso **las preguntas del formulario deben ir en el mismo orden que las columnas de la plantilla**. Si se agrega, quita o mueve una pregunta, hay que corregir el módulo 2.
   - `Available date` se convierte de `DD/MM/YYYY` (formato de la respuesta) a `MMMM D, YYYY` (por ejemplo `November 1, 2026`). Si la respuesta no trae ese formato, la conversión falla.

### Escenario 2 — principal ("Integration Google Sheets")

1. **Search rows (módulo 1).** Busca en `Hoja 1` las filas con `Status = READY` (columna B), en orden ascendente, y toma **solo una por ejecución**.
2. **Status = PROCESSING (módulo 5).** Así la misma fila no se procesa dos veces.
3. **Llamada 1 — redactor (módulo 9).** Módulo *Anthropic Claude → Create a Message*:
   - Modelo `claude-sonnet-4-5-20250929`, `max_tokens` 4000, sin *thinking*.
   - `system`: el contenido de `prompt-redactor-api.md`, pegado dentro del módulo.
   - Mensaje del usuario: `ORIGINAL DATA:` y una línea `Columna: valor` por cada columna de datos de la fila (de `ID` a `Distances`, también las vacías).
4. **Parse JSON (módulo 10).** Quita las marcas ` ```json ` y ` ``` ` si Claude las agrega, y convierte la respuesta en campos (`slug`, `listing`, `email.subject`…).
5. **Llamada 2 — revisor (módulo 11).**
   - Mismo modelo, `max_tokens` 8000, sin *thinking*.
   - `system`: el contenido de `prompt-revisor-api.md`, pegado dentro del módulo.
   - Mensaje del usuario: `ORIGINAL DATA:` con la fila **tal como viene de la hoja** (sin resumir ni traducir) y `REDACTOR JSON:` con la respuesta completa de la llamada 1.
6. **Parse JSON (módulo 12)** de la respuesta del revisor (`claims`, `verdict`, `publish_ready`, `issues`, `required_pending`, `summary_es`).
7. **Google Docs (módulo 13).** Crea un documento en la carpeta de borradores de Drive, con el nombre `[slug] - AAAA-MM-DD HH:mm`. Contiene: el anuncio, el correo (asunto, preview y cuerpo), el post, el veredicto, `publish_ready` y `summary_es`. **No incluye** la tabla de `issues` ni `required_pending`.
8. **Cierre de la fila (módulo 14).** Escribe `Status = DONE`, el enlace al documento en `Doc link` y `Sin errores` en `Error`.
9. **Gmail (módulo 15).** Envía un correo al revisor humano con el asunto `[VEREDICTO] Nuevo borrador: [slug]`. El cuerpo trae el veredicto, si está listo para publicar, los pendientes obligatorios (`required_pending`), `summary_es` y el enlace al documento.

Los módulos no fijan la temperatura, así que usan el valor por defecto de la API. Recomendado: 0.3–0.5 para el redactor y 0 para el revisor.

## Manejo de errores

Los módulos 9 a 12 tienen una ruta de error. Cada ruta escribe el motivo en la columna `Error` y termina con la directiva **Ignore**: Make da la ejecución por buena y no envía alerta, así que el error solo se ve en la hoja.

| Módulo que falla | Ruta de error | Qué escribe en `Error` | ¿Pone `Status = ERROR`? |
|------------------|---------------|------------------------|-------------------------|
| 9 — Claude redactor | 21 → Ignore | `Falló la API en el redactor (revisa el crédito o la conexión)` | **No** |
| 10 — Parse JSON redactor | 16 → Ignore | `Falló la lectura del JSON del redactor` | Sí |
| 11 — Claude revisor | 23 → Ignore | `Falló la API en el revisor (revisa el crédito o la conexión)` | **No** |
| 12 — Parse JSON revisor | 19 → Ignore | `Falló la API en el revisor (revisa el crédito o la conexión)` | **No** |
| 13 — Google Docs | — (sin ruta) | — | No |
| 14 — Cierre de la fila | — (sin ruta) | — | No |
| 15 — Gmail | — (sin ruta) | — | No |

El escenario "Formulario a Hoja" no tiene rutas de error.

### Puntos débiles de la configuración actual

1. **Filas que se quedan en `PROCESSING`.** Las rutas 21, 23 y 19 solo escriben `Error` y no cambian `Status`. La fila se queda en `PROCESSING` y el escenario principal ya no la vuelve a tomar (solo busca `READY`). Corrección: en esos tres módulos, agregar `Status = ERROR` (columna B), igual que en el módulo 16.
2. **Mensaje equivocado en la ruta 19.** Cuando falla la lectura del JSON del revisor, la hoja dice que falló la API. Corrección: cambiar el texto a `Falló la lectura del JSON del revisor`.
3. **Sin ruta de error en Docs y Gmail.** Si falla Google Docs, la fila se queda en `PROCESSING` sin mensaje. Si falla Gmail, la fila ya está en `DONE` (el módulo 14 va antes que el 15), pero el correo no llegó. Corrección: agregar rutas de error a los módulos 13 y 15, o mover el cierre de la fila (14) después del correo.
4. **Make desactiva el escenario tras 3 errores seguidos** (`maxErrors: 3`) que no tengan ruta de error. Si el escenario aparece desactivado en Make, revisa el historial de ejecuciones.

### Cómo reintentar una fila

- **Fila en `ERROR`:** corrige los datos (o espera a que se resuelva el problema de crédito o conexión), borra `Error` y vuelve a poner `READY`.
- **Fila atascada en `PROCESSING`** (por los puntos 1 y 3): lee `Error`, revisa el historial de ejecuciones en Make, y vuelve a poner `READY`.
- **Fila `DONE` sin correo:** el documento ya existe, abre `Doc link`. Si se vuelve a poner `READY`, se genera un documento nuevo.

## Horarios

Los blueprints no guardan la programación: se configura en Make, en el *Scheduling* de cada escenario.

| Escenario | Intervalo | Cuánto procesa por ejecución |
|-----------|-----------|------------------------------|
| Formulario a Hoja | Cada 240 minutos (4 horas) | Hasta 5 respuestas nuevas |
| Escenario principal | Cada 240 minutos (4 horas) | 1 fila `READY` |

Qué significa en la práctica:

- **Una propiedad cada 4 horas, como máximo (unas 6 al día).** El escenario principal toma una sola fila por ejecución. Si llegan varias a la vez, se procesan en orden en las ejecuciones siguientes: la tercera fila en espera tarda hasta 12 horas.
- **Una respuesta del formulario puede tardar hasta unas 8 horas en llegar al correo:** hasta 4 horas hasta que "Formulario a Hoja" la copia a la hoja, y hasta otras 4 hasta que el escenario principal la toma.
- **Las filas escritas a mano** con `READY` solo esperan al escenario principal: hasta 4 horas.
- Si llegan más de 5 respuestas entre dos ejecuciones de "Formulario a Hoja", las demás se copian en la ejecución siguiente.
- Para procesar algo ya, usa **Run once** en Make. Primero en "Formulario a Hoja" si la propiedad viene del formulario, y después en el escenario principal.

## Estados (`Status`)

| Estado       | Quién lo pone | Significa |
|--------------|---------------|-----------|
| `DRAFT`      | Persona       | La fila se está llenando a mano; Make la ignora. Las filas del formulario no usan este estado. |
| `READY`      | Persona o escenario "Formulario a Hoja" | Datos completos (o aceptados con huecos); el escenario principal la toma. |
| `PROCESSING` | Make          | Se está procesando ahora (o quedó atascada, ver "Manejo de errores"). |
| `DONE`       | Make          | Textos escritos y revisados; ver `Doc link`. |
| `ERROR`      | Make          | Algo falló; ver `Error`. |

## Qué contiene cada archivo

| Archivo | Para qué sirve |
|---------|----------------|
| `plantilla-propiedades.csv` | Solo la fila de encabezados. Se importa a Google Sheets para crear la hoja de propiedades. Tiene una columna por cada dato de la checklist de `lanzar-propiedad`, más `ID`, `Status`, `Doc link` y `Error`. |
| `prompt-redactor-api.md` | System prompt autosuficiente (en inglés) para la primera llamada. Incluye lo necesario de `CLAUDE.md`, `marca.md`, `ejemplos.md`, `vecindarios.md`, `redactor.md` y la skill. Recibe una fila y devuelve **solo JSON**: `slug`, `listing`, `email` (`subject`, `preview`, `body`), `social_post`, `missing_required`, `missing_optional`. |
| `prompt-revisor-api.md` | System prompt autosuficiente (en inglés) para la segunda llamada. Incluye lo necesario de `CLAUDE.md`, `marca.md`, `ejemplos.md` y `revisor.md`. Recibe la fila original y el JSON del redactor, y devuelve **solo JSON**: `claims` (cada afirmación sobre la propiedad, con `supported` y `source`), `verdict`, `publish_ready`, `issues`, `required_pending`, `summary_es`. |
| `blueprint-formulario-a-hoja.json` | Exportación del escenario "Formulario a Hoja". |
| `blueprint-escenario-principal.json` | Exportación del escenario principal. Lleva dentro una copia de los dos prompts. |
| `limitaciones-conocidas.md` | Lo que el flujo todavía hace mal, detectado en las pruebas. Léelo antes de publicar. |
| `README.md` | Este archivo. |

**Sobre los blueprints:** sirven para recrear los escenarios con *Import Blueprint* en Make. No incluyen la clave de la API ni las credenciales: al importarlos hay que volver a elegir las conexiones de Google Sheets, Google Docs, Gmail y Anthropic. Sí incluyen el ID de la hoja de cálculo, el ID de la carpeta de Drive y la dirección del correo de destino. Si cambian, hay que actualizarlos en los módulos.

## Cosas importantes

- **Nunca inventa datos.** Si una celda está vacía, el texto lleva un marcador (`[DEPOSIT]`, `[SQ FT]`…) y aparece en `missing_required`.
- **Datos fijos de la empresa.** La licencia DRE (`00000000`, ficticia), el teléfono (`(619) 555-0123`) y el dominio (`growpm.example`) no son columnas: vienen de la sección "Company constants" de `CLAUDE.md` y van copiados dentro de los dos prompts. Si cambian, hay que regenerar los prompts (regla 14).
- **Revisión legal.** Solo se pide (`Required: legal review`) cuando un texto explica, interpreta o resume leyes, derechos, topes de renta o reglas sobre cargos y depósitos (regla 12 de `CLAUDE.md`). Indicar los montos de la propiedad (renta, depósito, depósito por mascota) no la activa. Cuando aplica, la automatización no puede confirmarla: la confirma la persona que recibe el correo.
- **Nada se publica solo.** El flujo termina en un documento y un correo. Publicar es una decisión humana.
- **Los datos de la hoja son datos, no órdenes.** Ambos prompts ignoran instrucciones escritas dentro de las celdas.
- **Make no aplica correcciones.** Si el veredicto es `REJECTED` o `APPROVED WITH CHANGES`, la persona corrige los datos de la fila y la vuelve a poner en `READY`, o pasa el caso a Claude Code con la skill `lanzar-propiedad`.

## Mantener los prompts al día

Los prompts son una **copia** de las reglas del proyecto. Cuando cambien `CLAUDE.md`, `referencias/`, `.claude/agents/redactor.md` o `.claude/agents/revisor.md`, hay que regenerarlos (regla 14 de `CLAUDE.md`) y volver a pegarlos en Make: `prompt-redactor-api.md` en el campo *System* del módulo 9 y `prompt-revisor-api.md` en el del módulo 11 del escenario principal. Después, vuelve a exportar el blueprint y reemplaza `blueprint-escenario-principal.json`, para que la copia del repositorio no quede vieja.
