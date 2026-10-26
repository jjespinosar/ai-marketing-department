# Project structure / Estructura del proyecto

> Technical reference moved from the old main README. For the case study, see [README.md](../README.md) (English) or [README.es.md](../README.es.md) (español).

🇪🇸 [Español](#español) · 🇺🇸 [English](#english)

---

## Español

### ¿Qué es este proyecto?
Un **"departamento de marketing con IA"** para **Grow Property Management**, una empresa ficticia de administración de propiedades en San Diego, California. La idea es usar Claude Code como un equipo de marketing que redacta anuncios de propiedades, publicaciones para redes sociales, correos para dueños e inquilinos y más, siempre con una voz de marca consistente.

### Estructura

| Archivo / carpeta | Para qué sirve |
|---|---|
| `CLAUDE.md` | Contexto permanente para Claude: la empresa, sus clientes, el tono, las reglas y el flujo de trabajo. |
| `referencias/marca.md` | Guía de marca: voz, vocabulario, Fair Housing, regla de distancias, estructura de anuncios y colores. |
| `referencias/vecindarios.md` | Información general de 5 vecindarios de San Diego. |
| `referencias/ejemplos.md` | 2 anuncios modelo bien escritos. |
| `.claude/agents/` | Los 4 agentes del departamento. |
| `.claude/skills/` | Skills: flujos completos que se activan con una sola petición. |
| `salidas/` | Resultados: `investigacion/`, `planes/`, `borradores/` y `revisiones/`. |

### Los agentes

| Agente | Qué hace | Guarda en |
|---|---|---|
| `investigador` | Investiga vecindarios, rentas y temas de mercado, y etiqueta cada dato según su fuente. | `salidas/investigacion/` |
| `planificador` | Crea calendarios de contenido según la temporada de San Diego. | `salidas/planes/` |
| `redactor` | Escribe anuncios, correos y posts en inglés, sin inventar datos. | `salidas/borradores/` |
| `revisor` | Revisa textos (solo lectura): da un veredicto de calidad (VERDICT), indica si está completo (PUBLISH-READY) y propone correcciones. | `salidas/revisiones/` (`[borrador]-review.md`) |

Flujo: `investigador` → `planificador` → `redactor` → `revisor`. **Todo texto del redactor debe pasar por el revisor antes de darse por terminado.**

### Skills

| Skill | Cuándo se usa | Qué produce |
|---|---|---|
| `lanzar-propiedad` | Cuando entregas los datos de una propiedad nueva y quieres sus textos de lanzamiento. | Anuncio, correo para inquilinos y post para Instagram/Facebook en `salidas/borradores/`, una revisión de cada uno en `salidas/revisiones/`, una tabla resumen y un commit `Lanzamiento: [slug]`. |

Ejemplo: *"Lanza esta propiedad: condo 1 bed / 1 bath en Mission Valley, $2,450 al mes, depósito $2,450, disponible el 1 de noviembre…"* (o escribe `/lanzar-propiedad` seguido de los datos).

### Reglas clave
- Los textos finales de marketing se escriben **en inglés**.
- Las explicaciones al usuario se dan **en español**.
- Todo anuncio debe cumplir con las leyes de **vivienda justa (Fair Housing)**.
- El proyecto usa **Git**: antes de sobrescribir un archivo de `salidas/`, Claude avisa y guarda la versión actual en un commit.

### Cómo usarlo
Abre el proyecto con Claude Code y pide, por ejemplo: *"Usa el redactor para escribir un anuncio de un 2 recámaras en Hillcrest"*. Después, el revisor revisará el texto.

---

## English

### What is this project?
An **"AI marketing department"** for **Grow Property Management**, a fictional property management company in San Diego, California. Claude Code acts as a marketing team that writes property listings, social media posts, owner and tenant emails and more — always in a consistent brand voice.

### Structure

| File / folder | Purpose |
|---|---|
| `CLAUDE.md` | Permanent context for Claude: company, audiences, tone, rules and workflow. |
| `referencias/marca.md` | Brand guide: voice, vocabulary, Fair Housing, distance rule, listing structure and colors. |
| `referencias/vecindarios.md` | General information on 5 San Diego neighborhoods. |
| `referencias/ejemplos.md` | 2 well-written model listings. |
| `.claude/agents/` | The department's 4 agents. |
| `.claude/skills/` | Skills: complete workflows triggered by a single request. |
| `salidas/` | Outputs: `investigacion/`, `planes/`, `borradores/` and `revisiones/`. |

### The agents

| Agent | What it does | Saves to |
|---|---|---|
| `investigador` | Researches neighborhoods, rents and market topics, tagging every fact by source. | `salidas/investigacion/` |
| `planificador` | Builds content calendars adapted to San Diego's seasons. | `salidas/planes/` |
| `redactor` | Writes listings, emails and posts in English without inventing data. | `salidas/borradores/` |
| `revisor` | Reviews texts (read-only): returns a quality verdict (VERDICT), a completeness status (PUBLISH-READY) and fixes. | `salidas/revisiones/` (`[draft]-review.md`) |

Flow: `investigador` → `planificador` → `redactor` → `revisor`. **Every text from the writer must pass the reviewer before it is finished.**

### Skills

| Skill | When to use | Produces |
|---|---|---|
| `lanzar-propiedad` | When you provide the details of a new property and want its launch texts. | Listing, tenant email and Instagram/Facebook post in `salidas/borradores/`, one review each in `salidas/revisiones/`, a summary table and a commit `Lanzamiento: [slug]`. |

Example: *"Launch this property: 1 bed / 1 bath condo in Mission Valley, $2,450/month, $2,450 deposit, available November 1…"* (or type `/lanzar-propiedad` followed by the details).

### Key rules
- Final marketing copy is written **in English**.
- Explanations to the user are given **in Spanish**.
- Every listing must comply with **Fair Housing** laws.
- The project uses **Git**: before overwriting a file in `salidas/`, Claude warns you and commits the current version.

### How to use it
Open the project in Claude Code and ask, for example: *"Use the redactor to write a listing for a 2-bedroom in Hillcrest"*. The reviewer then checks the text.

---

*Grow Property Management is a fictional company created for educational purposes.*
