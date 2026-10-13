---
name: lanzar-propiedad
description: Use when the user provides the details of a new property and wants its launch texts (listing, email and social post). Checks the data, writes the three texts with the redactor agent, reviews each one with the revisor agent, saves everything to salidas/ and commits it.
---

# Launch a property (lanzar-propiedad)

Turns the details of a new rental property into three launch texts — a listing, an email to interested tenants and one social post for Instagram and Facebook — each reviewed and saved, and committed to Git.

Follow `CLAUDE.md` at all times: final copy in English, explanations to the user in Spanish, Fair Housing, never invent data.

## Step 1 — Check the data

Compare what the user gave you against this checklist:

| # | Field | Example |
|---|-------|---------|
| 1 | Property type | condo, single-family home, townhome, apartment |
| 2 | Neighborhood | Mission Valley |
| 3 | Bed / bath | 1 bed / 1 bath |
| 4 | Rent | $2,450/month |
| 5 | Security deposit | $2,450 |
| 6 | Available date | November 1, 2026 |
| 7 | Lease term | 12 months |
| 8 | Pet policy | cats only, $50/month pet rent |
| 9 | Utilities | resident pays SDG&E; water/trash included |
| 10 | Parking | 1 assigned covered space |
| 11 | Square footage | 720 sq ft |
| 12 | Features | **at least 4** real features (laundry, A/C, kitchen, balcony, storage…) |

## Step 2 — Ask once for everything missing

- If anything is missing, ask for **all** missing fields **in one single list** (in Spanish). Do not ask field by field.
- Wait for the answer. If the user replies "continúa" (or clearly says to go on without the data), continue with placeholders (`[DEPOSIT]`, `[AVAILABLE DATE]`, `[SQ FT]`…).
- **Never invent data.** Missing features are never turned into placeholder bullets: the listing uses only the real features, and the redactor lists the rest under "Missing information".
- If everything is present, go straight to Step 3.

## Step 3 — Create the slug

Build a short slug from neighborhood + bedrooms: lowercase, words joined by hyphens, bedrooms as `[n]bd` (`studio` for a studio).

Examples: `mission-valley-1bd`, `north-park-2bd`, `pacific-beach-studio`.

The three files will be:

- `salidas/borradores/[slug]-anuncio.md`
- `salidas/borradores/[slug]-email.md`
- `salidas/borradores/[slug]-post.md`

**Before writing**, check whether any of these files (or their reviews) already exist. If they do, follow rule 9 of `CLAUDE.md`: tell the user which files will be replaced and commit their current version first. Subagents cannot run Git, so do this before delegating.

## Step 4 — Write the three texts with the `redactor`

Call the `redactor` agent three times, once per text (the calls are independent and can run in parallel). Give each call the full property data, the content type, the audience (Tenants) and **the exact file path** from Step 3.

**a) Listing** → `[slug]-anuncio.md`
- Follow exactly the "Standard listing structure" section of `referencias/marca.md`.

**b) Email to interested tenants** → `[slug]-email.md`
- `Subject:` max **50 characters**.
- `Preview text:` max **90 characters**.
- Short body: the home's strongest benefits, key lease facts (rent, available date), and one clear next step.
- One call-to-action button, written as `[Button: Schedule a showing]` (or similar short label).
- **No emojis.**
- End with the compliance line: `Grow Property Management · CA DRE #[LICENSE] · Equal Housing Opportunity`.

**c) Social post for Instagram and Facebook** → `[slug]-post.md`
- **One single text** that works on both platforms.
- Max **2–3 emojis**.
- **3–5 local hashtags** (e.g. `#SanDiegoRentals`, `#MissionValley`, `#SanDiegoLiving`).
- Include rent, bed/bath and a call to action.

## Step 5 — Save the drafts

The `redactor` saves each text to the path you gave it. Confirm the three files exist in `salidas/borradores/`.

## Step 6 — Review each text with the `revisor`

Call the `revisor` agent **separately for each text**, passing the file path and the property data the user gave (the source facts). The `revisor` is read-only: save each report yourself to `salidas/revisiones/` as `[file-name]-review.md`:

- `[slug]-anuncio-review.md`
- `[slug]-email-review.md`
- `[slug]-post-review.md`

## Step 7 — Handle the verdicts

- **APPROVED** → done.
- **APPROVED WITH CHANGES** → per `CLAUDE.md`, the text is finished only when the fixes are applied. Send the fixes to the `redactor` (it saves `[file-name]-v2.md`), then review the new version once.
- **REJECTED** → send the text and the review back to the `redactor` **once** (it saves `[file-name]-v2.md`), and review it again (`[file-name]-v2-review.md`).
- If a text is REJECTED a **second time**: **stop**. Do not make the commit. Tell the user in Spanish which text failed and why, and show the table from Step 8 with what you have.

## Step 8 — Summary table (in Spanish)

End with this table, one row per text (use the latest version of each):

| Texto | Veredicto | Publish-ready | Pendientes |
|-------|-----------|---------------|------------|
| Anuncio (`[slug]-anuncio.md`) | APPROVED | NO | [DEPOSIT], [SQ FT] |
| Correo (`[slug]-email.md`) | … | … | … |
| Post (`[slug]-post.md`) | … | … | … |

"Pendientes" lists the remaining placeholders and missing information, or "Ninguno".

## Step 9 — Commit

Commit only the files of this launch:

```bash
git add salidas/borradores/[slug]-* salidas/revisiones/[slug]-*
git commit -m "Lanzamiento: [slug]"
```

Then explain to the user, in Spanish, what you did, which files you created, and what is still missing before the texts can be published.
