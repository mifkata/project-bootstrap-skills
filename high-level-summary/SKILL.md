---
name: high-level-summary
description: Generate a client/stakeholder-facing, human-readable summary of docs/high-level-spec/, organized around user journeys, including scope, constraints, audit safeguards and accepted risks, and improvements. Read-only on the spec. Manual invocation only.
license: MIT
argument-hint: "[lang:en|bg] [length:brief|short|full] [pdf]"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep
---

# high-level-summary

INPUT: `docs/high-level-spec/**` including `audit/` and `improvements/` when present. Missing → halt.
ARGS: `lang:` (missing → ask), `length:` (default `short`), `pdf` (optional export).
OUTPUT: `docs/high-level-summary.<lang>.md` (+ `.pdf` if requested).
AUDIENCE: non-technical clients/stakeholders. The skill file is agent-facing; the OUTPUT is human-facing prose.

## Rules
- R1 Read-only on the spec. Never edit HLS.
- R2 No invention. Every sentence traces to HLS content. No claims, numbers, benefits or timelines absent from the spec. Tone may persuade; content may not exceed the spec.
- R3 Traceability hidden from readers: trace IDs go in HTML comments after each paragraph/list item (`<!-- PW-F03, A-02 -->`). No IDs in visible text.
- R4 Plain language. No technical terms, no internal jargon (domain codes, "capability", "workflow", "actor"). Name actors by role as the client would ("customers", "store staff"). Define product-specific terms on first use.
- R5 Content filter:
  - include `final` items
  - omit `draft` and `withdrawn` items and open questions; report omitted counts to the user only, not in the doc
  - open `blocker`/`major` audit findings → ask before writing: include as "under review" / omit / halt
- R6 Language `bg`: write native Bulgarian, not translated English. Keep product/brand names untranslated. Use consistent terms throughout; the glossary is authoritative.
- R7 Regeneration: the file carries a generation stamp. If the file changed since the last stamp (manual edits) → ask: overwrite / write as `.new.md` / halt.

## Structure (fixed order; omit empty sections)
1. **Title + one-line proposition**: from the product summary.
2. **In brief**: what the product is, who it's for, what problem it solves (product.md). ≤1 paragraph.
3. **Who uses it**: one line per actor, primary end users first, then operators/admin.
4. **Journeys** (core, ~60% of length):
   - Group by actor (same order as 3). Within an actor, order by lifecycle: discover → join/onboard → core use → manage/adjust → leave/end.
   - Build each journey from workflows and cross-domain flows. Merge the features a journey touches into one story.
   - Per journey:
     - heading = the goal in the user's words ("Placing an order")
     - 2–6 sentence narrative, start to finish
     - "What the product takes care of": ≤4 bullets
     - outcome: 1 sentence
   - Include safeguards from fixed audit findings inline, where they occur in the journey, as benefits ("Customers can change their mind within 14 days").
   - Features not covered by any journey → section "Also included", one line each.
5. **What's included / not included**: scope in/out lists, plain language.
6. **Ground rules**: constraints (global + domain) rephrased as commitments/limits. Group legal obligations under "Compliance and privacy".
7. **Risks we've considered**:
   - `fixed` blocker/major findings → one line each, "risk → how it's handled"
   - `accepted-risk` → risk + the stated rationale, neutral tone
   - `not-applicable` and minor/note → omitted
8. **Improvements**:
   - applied improvements: "Added after market research", one line each with the user benefit
   - parked improvements: "Future opportunities", one line each
   - rejected: omitted
9. **Glossary**: product-specific terms from data concepts/actors used in the doc. Only terms a client wouldn't know.

## Length budgets
- `brief`: 1 page. Sections 1–2, top 3 journeys (primary actor first, 2 sentences each, no bullets), 5 as one line each, 7 limited to blocker-level items. Omit 6, 8, 9.
- `short`: 3–5 pages. All sections; journeys with bullets; "Also included" capped at 8 lines.
- `full`: no cap. Every journey and feature.
Over budget → merge journeys that share an actor and goal before cutting content.

## Mode
- Plan mode: P1–P3, render outline + full document into the plan file, ExitPlanMode. On approval → P4.
- Normal: P1–P4 with the outline checkpoint.
- Non-interactive: requires the `lang:` arg (else halt); open blocker/major findings → omit them and report.

## Procedure
P1 Ingest. Read HLS index, product.md, actors, domains, features, flows, constraints, `audit/` (findings, facts), `improvements/`. Apply R5. Detect manual edits (R7).

P2 Outline. Build: actor order, journey list per actor (title + covered feature IDs), "Also included" list, section inclusion per length budget.

P3 Checkpoint. Ask via AskUserQuestion:
- language (if not given)
- outline approval: approve / reorder journeys / rename journeys / drop sections
- handling of open blocker/major findings (if any)

P4 Write the document per structure, R2–R6. Self-check before saving:
- every visible paragraph/bullet has a trace comment
- no IDs, domain codes or technical terms in visible text
- length within budget
- glossary covers every product-specific term used

`pdf` arg → export via pandoc if available (ask to run); unavailable → skip and report.

Final reply, nothing else:
- output path(s)
- omitted counts (draft, withdrawn, open questions, open findings)
- self-check result

## Output skeleton
```markdown
<!-- generated: {date: <YYYY-MM-DD>, hls_rev: <sha/date>, lang: <lang>, length: <length>, hash: <sha256 of body>} -->
# <Product name>
<one-line proposition>

## In brief
<paragraph> <!-- ids -->

## Who uses it
- **<Role>**: <what they need from the product> <!-- A-nn -->

## <Role>: journeys
### <Goal in user's words>
<narrative> <!-- ids -->

**What the product takes care of**
- <benefit> <!-- ids -->

**Outcome:** <sentence> <!-- ids -->

## Also included
- <feature in one line> <!-- id -->

## What's included and what's not
**Included:** …
**Not included:** …

## Ground rules
### Compliance and privacy
- <commitment> <!-- G-Knn, AUD-nn -->
### Other commitments
- …

## Risks we've considered
- **<risk>**: <how it's handled | why it's accepted> <!-- AUD-nn -->

## Improvements
### Added after market research
- <benefit> <!-- IMP-nn -->
### Future opportunities
- <opportunity> <!-- IMP-nn -->

## Glossary
- **<term>**: <plain definition> <!-- DAT-Dnn -->
```
Headings are translated when `lang:bg`.
