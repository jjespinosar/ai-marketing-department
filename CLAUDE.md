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
9. **Never overwrite silently.** Before overwriting a file that already exists in `salidas/` (yourself or through a subagent), tell the user which file will be replaced and commit its current version first (`git add <file>` + `git commit -m "Backup: <file> before overwrite"`) so it can be recovered. The subagents cannot run git, so check for existing files *before* delegating.
   **One file per text:** never create `-v2` / `-v3` files. To correct a text, commit the current draft and its review, then overwrite the same files. Git keeps the history.
   **Drafts contain only publishable text.** Pending items (Required / Optional) go in the redactor's response and in the review report, never in the draft.
10. **Commit configuration changes.** Whenever `CLAUDE.md`, `README.md`, `referencias/`, `.claude/agents/` or `.claude/skills/` is modified, make a commit when the task is done, with a message that describes the change (example: `"marca.md: add hashtag rules"`).
11. **Delegation: pass the user's request verbatim.** When you hand a task to a subagent, send the user's request word for word. Any context or assumption you add goes in a separate section titled **"Added by orchestrator"**. Agents may mark something as "given" only if it appears in the user's original request; anything that comes only from "Added by orchestrator" is "assumed".
12. **Legal review is required.** Any text that talks about laws, security deposits, rent caps, fees or charges, or owner and tenant rights needs review by a person with legal knowledge. The `revisor` lists it as the pending item **"Required: legal review"**, which blocks PUBLISH-READY until the user confirms in writing that the review was done. The `planificador` flags these items in its plan, and the `investigador` puts legal facts under "Facts to verify".

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

**Quality vs. completeness:** the VERDICT measures quality; the PUBLISH-READY line measures completeness. A text can be APPROVED and still PUBLISH-READY: NO while placeholders or "Required" pending items remain (including "Required: legal review", see rule 12). Publish only when both are positive.

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
