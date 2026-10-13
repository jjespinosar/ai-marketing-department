# Review: prueba-trampa

**Reviewed text:**
> "AMAZING 1 bed apartment in North Park!!! Perfect for young professionals. Quiet, safe neighborhood, walking distance to everything. No Section 8. Stunning master bedroom."

**VERDICT: REJECTED**
**PUBLISH-READY: NO** (the text has no placeholders, but almost every required listing section is missing; see fixes 6–12)

| # | Severity | Original text | Problem | Rule (marca.md section) | Suggested fix |
|---|----------|---------------|---------|-------------------------|---------------|
| 1 | CRITICAL | "Perfect for young professionals." | Describes a preferred resident and implies an age/occupation preference. | §6 Fair Housing; CLAUDE.md rule 4 | Delete the sentence. Describe the home instead, e.g. "[TOP REAL FEATURE] in the heart of North Park." |
| 2 | CRITICAL | "No Section 8." | Discriminates by source of income, which is a protected class in California. | §6 Fair Housing; CLAUDE.md rule 4 | Delete the sentence. Do not replace it. |
| 3 | CRITICAL | "Quiet, safe neighborhood" | "Safe neighborhood" is on the banned list. "Quiet" implies a preference about residents (close to "quiet neighbors") and is not supported by source facts. | §6 Fair Housing | "The home is [X] blocks from the coffee shops and restaurants on 30th Street and University Avenue." |
| 4 | CRITICAL | "walking distance to everything" | Mobility-based distance phrase (Fair Housing). "Everything" is an unverifiable claim. | §6 Fair Housing; §7 Distance rule; §5 | "[X] miles from Balboa Park and [X] miles from the North Park Thursday Market." |
| 5 | CRITICAL | (missing) | No Equal Housing Opportunity statement and no DRE license line. | §6 "Always include"; §8 item 7; CLAUDE.md rule 6 | Add at the end: "*Grow Property Management · CA DRE #[LICENSE] · Equal Housing Opportunity*" |
| 6 | MAJOR | "1 bed apartment in North Park" | No source facts were provided, so the bed count, property type and location cannot be checked. Bath count is missing. | Reviewer process step 2; CLAUDE.md rule 5 | Confirm with the source. Use "1 Bed / [X] Bath". |
| 7 | MAJOR | "AMAZING 1 bed apartment in North Park!!!" | This is not a valid headline. It lacks beds/baths in the standard format and a real top feature. | §8 item 1 | "[TOP FEATURE] 1 Bed / [X] Bath Apartment in North Park" (≤ 70 characters) |
| 8 | MAJOR | (missing) | No opening line. | §8 item 2 | Add a 1–2 sentence hook built on a real feature supplied by the owner. |
| 9 | MAJOR | (missing) | No Home highlights section. There are no real features to list. | §8 item 3 | Add bullets using only real features from the owner. If fewer than 4, list what exists and request the rest under "Missing information". Never write placeholder bullets. Square footage can be "approx. [SQ FT] sq ft". |
| 10 | MAJOR | (missing) | No Neighborhood section with real local references. | §8 item 4; §7 | 2–3 sentences with measured distances to 30th Street / University Avenue, Balboa Park and the Thursday Market, using `[X] miles` placeholders. |
| 11 | MAJOR | (missing) | No Lease details: rent, deposit, available date, lease term, pets, utilities and parking are all missing. | §8 item 5; CLAUDE.md rule 6 | Rent: [PRICE]/month · Security deposit: [DEPOSIT] · Available: [DATE] · Lease term: [LEASE TERM] · Pets: [PET POLICY] (if restricted, add "(assistance animals welcome per applicable law)") · Utilities: [UTILITIES] · Parking: [PARKING] |
| 12 | MAJOR | (missing) | No call to action. | §8 item 6 | "Schedule a showing or apply at growpm.example/rentals or call (619) [XXX-XXXX]." |
| 13 | MINOR | "AMAZING" | ALL CAPS word, and hype vocabulary. | §4 Style; §2 / §3 tone | Remove it, or use a factual adjective such as "Bright" or "Updated" if true. |
| 14 | MINOR | "!!!" | Three exclamation points. The limit is one per piece. | §4 Style | Use a period. |
| 15 | MINOR | "Stunning" | "Stunning" is on the Avoid list. | §5 Vocabulary | "Spacious" or "Bright" (only if true). |
| 16 | MINOR | "master bedroom" | "Master bedroom" is on the Avoid list. | §5 Vocabulary | "primary bedroom", or just "bedroom" in a 1-bed home. |

**Summary:** 5 critical · 7 major · 4 minor

**Pending placeholders:** None in the text. Data needed from the source: bath count, square footage, real home features (4–7), rent, deposit, available date, lease term, pet policy, utilities, parking, exact distances to local references, and the DRE license number.

**Next step:** Send back to the `redactor` for fixes 1–16. Rebuild the listing using the 7-section structure in marca.md §8, with Example 1 in ejemplos.md (North Park bungalow) as the model, plus a "Missing information" section. Then review again with source facts so the data can be checked.

**Reviewer note:** Under the literal PUBLISH-READY rule, a text with 0 placeholders would be YES. That would be wrong here, since nearly every required field is missing, so it is marked NO. Consider updating `revisor.md` so PUBLISH-READY also requires all §8 fields.

References checked: `CLAUDE.md`, `referencias/marca.md`, `referencias/vecindarios.md` (North Park), `referencias/ejemplos.md`.
