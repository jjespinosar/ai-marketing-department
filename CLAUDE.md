# CLAUDE.md — Grow Property Management AI Marketing Department

This file gives Claude the permanent context for this project. Read it before any task.

## Who we are

**Grow Property Management** is a (fictional) residential property management company based in **San Diego, California**. We manage single-family homes, condos, townhomes and small multifamily buildings (2–20 units) across San Diego County.

What we do:
- Tenant placement (marketing, showings, screening, leases)
- Full-service management (rent collection, maintenance, inspections, accounting)
- Owner reporting and rental pricing analysis
- Move-in / move-out coordination and security deposit handling

Tagline: **"Local experts. Easy leasing."**

## Our two audiences

### 1. Property owners (our paying clients)
- Local and out-of-state owners, accidental landlords, small investors with 1–10 doors.
- What they want: reliable tenants, fewer vacant days, protected property, clear monthly reports, no headaches.
- What worries them: bad tenants, late rent, maintenance costs, California rental law (AB 1482, just-cause eviction, rent caps).
- How we talk to them: confident, data-informed, reassuring. Show expertise without jargon.

### 2. Tenants (residents and prospects)
- Students, military families, healthcare and biotech workers, remote workers, retirees.
- What they want: a well-kept home, fair price, fast responses, an easy application, a good neighborhood.
- How we talk to them: warm, clear, helpful. Highlight the home and the neighborhood lifestyle.

## Brand voice (summary)

**Professional · Approachable · Local.**
- Professional: accurate, organized, no exaggerations, no typos.
- Approachable: plain English, short sentences, friendly but never cheesy.
- Local: we know San Diego — mention real neighborhood character, beaches, parks, commute routes and local spots.

Full guide: `referencias/marca.md`.

## General rules

1. **All final marketing copy is written in English** (listings, emails, social posts, web copy, ads).
2. **Explanations to the user are in Spanish**: after each task, explain in Spanish what you did, why, and which files you created or changed.
3. **Always check the references** before writing:
   - `referencias/marca.md` — voice, vocabulary, formatting
   - `referencias/vecindarios.md` — neighborhood facts
   - `referencias/ejemplos.md` — model listings to imitate in structure and quality
4. **Fair Housing compliance is mandatory.** Describe the *property*, never the ideal *person*. Do not mention or imply preferences about race, color, religion, sex, disability, familial status, national origin, or California-protected classes (source of income, sexual orientation, gender identity, marital status, age, etc.). Avoid phrases like "perfect for families", "ideal for young professionals", "no Section 8", "adults only", "Christian neighborhood". See the full list in `marca.md`.
5. **Do not invent facts.** If a price, square footage, date or amenity is missing, use a clear placeholder like `[PRICE]` or `[SQ FT]` and tell the user.
6. **Include required details in listings**: rent, deposit, availability date, pet policy, utilities, and the company license line `Grow Property Management · CA DRE #[LICENSE]`.
7. **Neighborhood data in `vecindarios.md` is approximate** and for creative context only. Do not publish specific statistics (rents, prices, school ratings, crime) without the user verifying them.
8. Keep outputs organized: save all deliverables inside `salidas/` (see Project structure).

## Workflow

The marketing department has 4 subagents in `.claude/agents/`:

| Agent          | Job                                                    | Saves to                 |
|----------------|--------------------------------------------------------|--------------------------|
| `investigador` | Researches neighborhoods, rent trends, market topics   | `salidas/investigacion/` |
| `planificador` | Builds weekly or monthly content calendars             | `salidas/planes/`        |
| `redactor`     | Writes listings, owner/tenant emails, social posts     | `salidas/borradores/`    |
| `revisor`      | Reviews any marketing text (read-only)                 | `salidas/revisiones/`*   |

\* The `revisor` is read-only: the main conversation saves its report to `salidas/revisiones/` as `[draft-name]-review.md` (example: `anuncio-hillcrest-2bd-review.md`).

Typical flow: `investigador` → `planificador` → `redactor` → `revisor`.

**Mandatory review rule:** every text written by the `redactor` must be reviewed by the `revisor` before it is considered finished. A text is finished only when the verdict is APPROVED, or APPROVED WITH CHANGES and those changes have been applied. A REJECTED text goes back to the `redactor` and is reviewed again.

**Quality vs. completeness:** the VERDICT measures quality; the PUBLISH-READY line measures completeness. A text can be APPROVED and still PUBLISH-READY: NO while placeholders remain. Publish only when both are positive.

## Project structure

```
marketing-ia-propiedades/
├── CLAUDE.md                ← this file (project context and rules)
├── README.md                ← project overview (Spanish + English)
├── .claude/
│   └── agents/
│       ├── investigador.md  ← researcher
│       ├── planificador.md  ← content planner
│       ├── redactor.md      ← copywriter
│       └── revisor.md       ← reviewer (read-only)
├── referencias/
│   ├── marca.md             ← brand guide
│   ├── vecindarios.md       ← 5 San Diego neighborhoods
│   └── ejemplos.md          ← 2 model property listings
└── salidas/
    ├── investigacion/       ← research reports
    ├── planes/              ← content plans
    ├── borradores/          ← drafts (listings, emails, posts)
    └── revisiones/          ← reviewer reports ([draft-name]-review.md)
```
