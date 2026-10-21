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

## Company constants

Fixed company data. Agents, skills and the API prompts in `automatizacion/` take these values from here. They count as **given facts** in every text: never replace them with a placeholder and never change them.

| Constant | Value | Use |
|----------|-------|-----|
| CA DRE license | `00000000` (fictional) | Compliance line: `Grow Property Management · CA DRE #00000000 · Equal Housing Opportunity` |
| Phone | `(619) 555-0123` | Calls to action and contact blocks |
| Domain (example) | `growpm.example` | Web links and company email addresses (e.g. `growpm.example/rentals`, `leasing@growpm.example`) |

All three values are fictional placeholders for this project. Replace them here (and regenerate the API prompts, rule 14) before any real publication. Property-specific data (showing link, contact email) still comes from the user for each property.

## General rules

1. **All final marketing copy is written in English** (listings, emails, social posts, web copy, ads).
2. **Explanations to the user are in Spanish**: after each task, explain in Spanish what you did, why, and which files you created or changed.
3. **Always check the references** before writing:
   - `referencias/marca.md` — voice, vocabulary, formatting
   - `referencias/vecindarios.md` — neighborhood facts
   - `referencias/ejemplos.md` — model listings to imitate in structure and quality
4. **Fair Housing compliance is mandatory.** Describe the *property*, never the ideal *person*. Do not mention or imply preferences about race, color, religion, sex, disability, familial status, national origin, or California-protected classes (source of income, sexual orientation, gender identity, marital status, age, etc.). Avoid phrases like "perfect for families", "ideal for young professionals", "no Section 8", "adults only", "Christian neighborhood". See the full list in `marca.md`.
5. **Do not invent facts.** If a price, square footage, date or amenity is missing, use a clear placeholder like `[PRICE]` or `[SQ FT]` and tell the user.
6. **Include required details in listings**: rent, deposit, availability date, pet policy, utilities, and the company license line `Grow Property Management · CA DRE #00000000 · Equal Housing Opportunity` (see Company constants).
7. **Neighborhood data in `vecindarios.md` is approximate** and for creative context only. Do not publish specific statistics (rents, prices, school ratings, crime) without the user verifying them.
8. Keep outputs organized: save all deliverables inside `salidas/` (see Project structure).
9. **Never overwrite silently.** Before overwriting a file that already exists in `salidas/` (yourself or through a subagent), tell the user which file will be replaced and commit its current version first (`git add <file>` + `git commit -m "Backup: <file> before overwrite"`) so it can be recovered. The subagents cannot run git, so check for existing files *before* delegating.
   **One file per text:** never create `-v2` / `-v3` files. To correct a text, commit the current draft and its review, then overwrite the same files. Git keeps the history.
   **Drafts contain only publishable text.** Pending items (Required / Optional) go in the redactor's response and in the review report, never in the draft.
10. **Commit configuration changes.** Whenever `CLAUDE.md`, `README.md`, `referencias/`, `.claude/agents/` or `.claude/skills/` is modified, make a commit when the task is done, with a message that describes the change (example: `"marca.md: add hashtag rules"`).
11. **Delegation: pass the user's request verbatim.** When you hand a task to a subagent, send the user's request word for word. Any context or assumption you add goes in a separate section titled **"Added by orchestrator"**. Agents may mark something as "given" only if it appears in the user's original request; anything that comes only from "Added by orchestrator" is "assumed".
12. **Legal review is required — only for legal content.** A text needs review by a person with legal knowledge **only when it explains, interprets or summarizes** laws, owner or tenant rights, rent caps, or rules about fees, charges and deposits (examples: what AB 1482 allows, how just-cause eviction works, when a deposit must be returned, which fees are legal). It does **not** apply when the text only states the property's own data — rent, security deposit amount, pet deposit, pet rent or other fee amounts — or the mandatory compliance phrases (the Equal Housing Opportunity line and "(assistance animals welcome per applicable law)"). When it applies, the `revisor` lists it as the pending item **"Required: legal review"**, which blocks PUBLISH-READY until the user confirms in writing that the review was done. The `planificador` flags these items in its plan, and the `investigador` puts legal facts under "Facts to verify".
13. **Commit every finished deliverable.** When a deliverable in `salidas/` is finished (research report, plan, draft, review report), commit it with a message that says what it is (example: `"Plan: noviembre 2026 Pacific Beach"`). The subagents cannot run git, so the main conversation makes this commit. This adds to rule 9 (backup commit before overwriting); it does not replace it.
14. **Keep the API prompts in sync.** Whenever `CLAUDE.md`, anything in `referencias/`, `.claude/agents/redactor.md` or `.claude/agents/revisor.md` changes (or the field checklist of `.claude/skills/lanzar-propiedad/SKILL.md`, which defines the CSV columns), regenerate `automatizacion/prompt-redactor-api.md` and `automatizacion/prompt-revisor-api.md` (and `plantilla-propiedades.csv` if the fields changed) so they stay self-contained, commit them together with the change, and tell the user in Spanish that the prompts were regenerated and must be pasted again into the Make modules.

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

## Skills

| Skill              | Use when                                                                 | Produces |
|--------------------|--------------------------------------------------------------------------|----------|
| `lanzar-propiedad` | The user gives the details of a new property and wants its launch texts | Listing, tenant email and social post in `salidas/borradores/`, one review each in `salidas/revisiones/`, a summary table and a commit `Lanzamiento: [slug]` |

Skill files live in `.claude/skills/[name]/SKILL.md`.

**Quality vs. completeness:** the VERDICT measures quality; the PUBLISH-READY line measures completeness. A text can be APPROVED and still PUBLISH-READY: NO while placeholders or "Required" pending items remain (including "Required: legal review", see rule 12). PUBLISH-READY can be YES only when the verdict is APPROVED: with APPROVED WITH CHANGES or REJECTED it is always NO, until the fixes are applied and a new review returns APPROVED. Publish only when both are positive.

## Project structure

```
marketing-ia-propiedades/
├── CLAUDE.md                ← this file (project context and rules)
├── README.md                ← project overview (Spanish + English)
├── .gitignore               ← files Git ignores
├── .claude/
│   ├── agents/
│   │   ├── investigador.md  ← researcher
│   │   ├── planificador.md  ← content planner
│   │   ├── redactor.md      ← copywriter
│   │   └── revisor.md       ← reviewer (read-only)
│   └── skills/
│       └── lanzar-propiedad/
│           └── SKILL.md     ← new-property launch (listing + email + post)
├── automatizacion/
│   ├── README.md            ← Sheets → Make → Claude → Docs/Gmail flow (Spanish)
│   ├── plantilla-propiedades.csv ← Google Sheets header row
│   ├── prompt-redactor-api.md    ← self-contained writer system prompt (API)
│   └── prompt-revisor-api.md     ← self-contained reviewer system prompt (API)
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
