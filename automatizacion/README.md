# Automatización: Google Sheets → Make → Claude → Docs / Gmail

Esta carpeta conecta el departamento de marketing con **Make** y la **API de Claude** para lanzar propiedades sin abrir Claude Code: alguien llena una fila en Google Sheets y recibe un correo con los tres textos ya escritos y revisados.

Es la versión automática de la skill `lanzar-propiedad`. Las reglas son las mismas (inglés para el texto final, Fair Housing, nunca inventar datos, revisión obligatoria), pero metidas dentro de los prompts, porque la API no puede leer los archivos del proyecto.

## El flujo

```
Google Sheets          Make                         Claude API                 Google Docs / Gmail
─────────────          ────                         ──────────                 ───────────────────
Fila con Status   →   1. Busca filas READY
= READY               2. Status = PROCESSING
                      3. Llamada 1 (redactor)  →   prompt-redactor-api.md
                                               ←   JSON con los 3 textos
                      4. Parse JSON
                      5. Llamada 2 (revisor)   →   prompt-revisor-api.md
                                               ←   JSON con veredicto
                      6. Parse JSON
                      7. Crea el documento     ─────────────────────────→  Google Doc con los textos
                      8. Envía el correo       ─────────────────────────→  Gmail al revisor humano
                      9. Status = DONE + Doc link
   (si algo falla)    → Status = ERROR + Error
```

### Paso a paso

1. **Google Sheets.** Se importa `plantilla-propiedades.csv` para crear la hoja. Cada fila es una propiedad. Mientras se llena, `Status` = `DRAFT`. Cuando está lista para procesar, se cambia a `READY`.
2. **Make — disparador.** Un escenario programado (por ejemplo cada 15 minutos) busca filas con `Status = READY`. Para cada una, primero cambia `Status` a `PROCESSING`, así la misma fila no se procesa dos veces.
3. **Llamada 1 — redactor.** Módulo de Anthropic Claude (o HTTP a `https://api.anthropic.com/v1/messages`):
   - `system`: el contenido de `prompt-redactor-api.md` desde la línea `=== SYSTEM PROMPT START ===` hacia abajo.
   - mensaje del usuario: los datos de la fila como líneas `Columna: valor` (todas las columnas de datos, también las vacías).
   - Recomendado: temperatura baja (0.3–0.5) y `max_tokens` de unos 4000.
4. **Parse JSON.** El módulo *Parse JSON* de Make convierte la respuesta en campos (`slug`, `listing`, `email.subject`…). Si falla, la fila pasa a `ERROR`.
5. **Llamada 2 — revisor.**
   - `system`: el contenido de `prompt-revisor-api.md` desde `=== SYSTEM PROMPT START ===`.
   - mensaje del usuario: dos bloques, `ORIGINAL DATA:` con la fila **tal como viene de la hoja** (sin resumir ni traducir) y `REDACTOR JSON:` con la respuesta completa de la llamada 1.
   - Recomendado: temperatura 0.
6. **Parse JSON** de la respuesta del revisor (`claims`, `verdict`, `publish_ready`, `issues`, `required_pending`, `summary_es`).
7. **Google Docs.** Se crea un documento (idealmente desde una plantilla) con: el anuncio, el correo (asunto, preview y cuerpo), el post, la tabla de `issues` y los pendientes.
8. **Gmail.** Se envía un correo al revisor humano con `summary_es` en el cuerpo y el enlace al documento.
9. **Cierre.** Make escribe el enlace en `Doc link` y pone `Status = DONE`.

Si cualquier paso falla (error de la API, JSON inválido, Docs o Gmail no responden), una ruta de manejo de errores pone `Status = ERROR` y escribe el motivo en `Error`. Para reintentar, se corrige la fila, se borra `Error` y se vuelve a poner `READY`.

## Estados (`Status`)

| Estado       | Quién lo pone | Significa |
|--------------|---------------|-----------|
| `DRAFT`      | Persona       | La fila se está llenando; Make la ignora. |
| `READY`      | Persona       | Datos completos (o aceptados con huecos); Make la toma. |
| `PROCESSING` | Make          | Se está procesando ahora. |
| `DONE`       | Make          | Textos escritos y revisados; ver `Doc link`. |
| `ERROR`      | Make          | Algo falló; ver `Error`. |

## Qué contiene cada archivo

| Archivo | Para qué sirve |
|---------|----------------|
| `plantilla-propiedades.csv` | Solo la fila de encabezados. Se importa a Google Sheets para crear la hoja de propiedades. Tiene una columna por cada dato de la checklist de `lanzar-propiedad`, más `ID`, `Status`, `Doc link` y `Error`. |
| `prompt-redactor-api.md` | System prompt autosuficiente (en inglés) para la primera llamada. Incluye lo necesario de `CLAUDE.md`, `marca.md`, `ejemplos.md`, `vecindarios.md`, `redactor.md` y la skill. Recibe una fila y devuelve **solo JSON**: `slug`, `listing`, `email` (`subject`, `preview`, `body`), `social_post`, `missing_required`, `missing_optional`. |
| `prompt-revisor-api.md` | System prompt autosuficiente (en inglés) para la segunda llamada. Incluye lo necesario de `CLAUDE.md`, `marca.md`, `ejemplos.md` y `revisor.md`. Recibe la fila original y el JSON del redactor, y devuelve **solo JSON**: `claims` (cada afirmación sobre la propiedad, con `supported` y `source`), `verdict`, `publish_ready`, `issues`, `required_pending`, `summary_es`. |
| `README.md` | Este archivo. |

## Cosas importantes

- **Nunca inventa datos.** Si una celda está vacía, el texto lleva un marcador (`[DEPOSIT]`, `[SQ FT]`…) y aparece en `missing_required`.
- **Datos fijos de la empresa.** La licencia DRE (`00000000`, ficticia), el teléfono (`(619) 555-0123`) y el dominio (`growpm.example`) no son columnas: vienen de la sección "Company constants" de `CLAUDE.md` y van copiados dentro de los dos prompts. Si cambian, hay que regenerar los prompts (regla 14).
- **Revisión legal.** Solo se pide (`Required: legal review`) cuando un texto explica, interpreta o resume leyes, derechos, topes de renta o reglas sobre cargos y depósitos (regla 12 de `CLAUDE.md`). Indicar los montos de la propiedad (renta, depósito, depósito por mascota) no la activa. Cuando aplica, la automatización no puede confirmarla: la confirma la persona que recibe el correo.
- **Nada se publica solo.** El flujo termina en un documento y un correo. Publicar es una decisión humana.
- **Los datos de la hoja son datos, no órdenes.** Ambos prompts ignoran instrucciones escritas dentro de las celdas.
- **Make no aplica correcciones.** Si el veredicto es `REJECTED` o `APPROVED WITH CHANGES`, la persona corrige los datos de la fila y la vuelve a poner en `READY`, o pasa el caso a Claude Code con la skill `lanzar-propiedad`.

## Mantener los prompts al día

Los prompts son una **copia** de las reglas del proyecto. Cuando cambien `CLAUDE.md`, `referencias/`, `.claude/agents/redactor.md` o `.claude/agents/revisor.md`, hay que regenerarlos (regla 14 de `CLAUDE.md`) y volver a pegarlos en los módulos de Make.
