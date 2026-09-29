---
name: product-architect
description: Decompose a product description document into non-technical, high-level requirements under docs/high-level-spec/, one file per domain/feature/scope item, for a downstream System Architect agent. Manual invocation only.
license: MIT
argument-hint: <path-to-product-doc>
disable-model-invocation: true
allowed-tools: Read, Glob, Grep
---

# product-architect

INPUT: `$ARGUMENTS` = path to product description doc. Empty/unreadable → ask for path, halt.
OUTPUT: `docs/high-level-spec/**` — atomic files, one ID per file.
CONSUMER: System Architect agent (proposes technical solution options). Optimize for machine consumption and per-file revision: stable IDs, frontmatter, ID refs, no prose padding.
BOUNDARY: define WHAT and WHO. Never HOW.

## Rules
- R1 Non-technical. Forbidden: technologies, frameworks, protocols, vendors (unless source names them as a business constraint), schemas/types/tables, endpoints/APIs, architecture/deployment patterns, UI layouts/screens/components, estimates.
- R2 Every item carries `src`: `§<section>` / short anchor from source, or `Q-<id>` (user answer). No src → question, not item.
- R3 No feature-design decisions. Suggestions encouraged, live only in question files as `suggestion`; promoted only when accepted (src = Q-id).
- R4 Domain set adaptive. No domain included or excluded without src.
- R5 Writes only under `docs/high-level-spec/`.
- R6 Never default an unstated behavior. Hotspots commonly left implicit — always check: roles/permissions, account lifecycle, ownership/multi-tenancy, pricing, data retention/deletion, offline use, locales, notification channels, moderation, audit.
- R7 Atomicity: one file = one ID = one independently revisable unit. Never aggregate multiple features, actors, flows, concepts, constraints, or questions in one file. Feature with >1 independent user outcome → split into separate features.
- R8 Cross-reference by ID only (relative link). Never duplicate another file's content.

## Mode
- Plan mode (writes blocked): P1–P4, questions via AskUserQuestion, render full file set into plan file (one `### <path>` block per file), ExitPlanMode. On approval → P5 writes blocks verbatim.
- Normal: P1–P5, questions via AskUserQuestion.
- Non-interactive (no user): P1–P5; unresolved → question files `status: open`; blocked items not written; affected domain `status: draft`.

## Procedure
P1 Ingest. Read source. If `docs/high-level-spec/` exists → update mode: read only files relevant to changes, preserve IDs and filenames (never rename/renumber even if title changes), carry answered questions, set removed items `status: withdrawn` (don't delete). Touch only files whose content changes; always regenerate `index.md`.

P2 Domains.
- Baseline: `public-web` PW, `admin` ADM, `mobile` MOB, `server` SRV, `data` DAT.
- Discover extras from source signals. Examples (non-exhaustive — derive others):

| signal in source | candidate | code |
|---|---|---|
| pricing, plans, checkout, invoices, refunds | billing | BIL |
| alerts, reminders, digests, "notify" | notifications | NOT |
| named external systems/partners | integrations | INT |
| dashboards, KPIs, exports | reporting | REP |
| health/finance/minors/PII, audit trail | compliance | CMP |
| third parties consuming product data | partner-api | PAPI |
| multiple languages/regions/currencies | localization | L10N |
| staff-managed editorial content | content | CNT |
| tickets, disputes, account recovery by staff | support-ops | SUP |

- Classify each: `in` (src), `out` (src explicitly excludes), `candidate` (signal, unconfirmed) → question, `unknown` (baseline, no signal) → question.

P3 Extract per `in` domain: features (with their workflows), data concepts, cross-domain flows, actors, constraints. Assign IDs + src each. Apply R7.

P4 Questions.
- Test every P2 classification and P3 item against R2; sweep R6 hotspots per domain.
- Scope-generic phrases ("manage users", "handle payments") → question on concrete scope.
- Workflow requiring an unnamed actor → question.
- Domain/feature boundary overlap → question.
- First batch = domain set confirmation. Then batch by domain, small batches, options mutually exclusive, suggestion as an option labeled `(suggested)`.
- Loop until no new questions or user defers. Deferred → `open`.

P5 Write files per layout/templates. Omit empty sections. Final reply: created/updated/withdrawn paths, counts (domains, features, open questions). Nothing else.

## IDs
- Actor `A-nn`, global constraint `G-Knn`, cross-domain flow `X-nn`
- Feature `<CODE>-Fnn`, its workflows `<CODE>-Fnn-Wn` (inline, not separate files)
- Data concept `DAT-Dnn` (DAT `out` → `<CODE>-Dnn` under owning domain)
- Domain constraint `<CODE>-Knn`, question `Q-<CODE>-nn` (global: `Q-G-nn`)
- Filename = `<ID>-<kebab-slug>.md`. Slug fixed at creation.

## Layout
```
docs/high-level-spec/
  index.md                          # generated manifest, never source of truth
  product.md                        # summary + product-level scope in/out
  actors/A-nn-<slug>.md
  constraints/G-Knn-<slug>.md
  flows/X-nn-<slug>.md
  domains/<code>-<slug>/
    domain.md                       # purpose, scope in/out, feature index
    features/<CODE>-Fnn-<slug>.md
    concepts/<CODE>-Dnn-<slug>.md   # DAT domain (or owning domain if DAT out)
    constraints/<CODE>-Knn-<slug>.md
  questions/Q-<CODE>-nn.md
```
Lowercase code in dir names (e.g. `domains/pw-public-web/`).

## Templates
Frontmatter required on every file. `status` ∈ `draft|final|withdrawn`.

### index.md
```markdown
---
product: <name>
source: <path>
updated: <YYYY-MM-DD>
counts: {domains: n, features: n, concepts: n, flows: n, open_questions: n}
---
| id | type | title | status | path |
```

### product.md
```markdown
---
id: PRODUCT
status: <status>
src: [<src>]
---
## Summary
<≤5 sentences, source-only>
## In scope
- <item> — src
## Out of scope
- <item> — src
```

### domains/<code>-<slug>/domain.md
```markdown
---
id: <CODE>
domain: <slug>
classification: in
status: <status>
depends_on: [<CODE>]
actors: [A-nn]
open_questions: [Q-..]
src: [<src>]
---
## Purpose
<1–2 sentences>
## Scope
in: <bullets, src each>
out: <bullets, src each>
## Features
<list of links to feature files, id — title only>
```

### features/<CODE>-Fnn-<slug>.md
```markdown
---
id: <CODE>-Fnn
domain: <CODE>
title: <title>
status: <status>
actors: [A-nn]
concepts: [DAT-Dnn]
depends_on: [<feature/flow ids>]
constraints: [<K ids>]
open_questions: [Q-..]
src: [<src>]
---
## Outcome
<one sentence: what the actor achieves>
## Workflows
### <CODE>-Fnn-W1 <name>
1. <A-nn>: <user-visible step>
outcome: <result>
exceptions: <stated only>
## Rules
- <stated business rule> — src
```

### concepts/<CODE>-Dnn-<slug>.md
```markdown
---
id: <CODE>-Dnn
title: <noun>
status: <status>
owner_domain: <CODE>
used_by: [<feature ids>]
related: [<concept ids>]
src: [<src>]
---
## Meaning
<1–2 sentences>
## Relations
- <plain-language relation to concept id>
## Lifecycle / sensitivity
<stated only>
```

### actors/A-nn-<slug>.md
```markdown
---
id: A-nn
title: <role>
status: <status>
domains: [<CODE>]
src: [<src>]
---
<1–2 sentences: who, what they need>
```

### flows/X-nn-<slug>.md
```markdown
---
id: X-nn
title: <name>
status: <status>
domains: [<CODE> ordered]
features: [<feature ids>]
src: [<src>]
---
1. <CODE>/<feature id>: <step>
```

### constraints/<ID>-<slug>.md (global or domain)
```markdown
---
id: <G-Knn|CODE-Knn>
status: <status>
applies_to: [<ids>]
src: [<src>]
---
<stated constraint, verbatim meaning>
```

### questions/Q-<CODE>-nn.md
```markdown
---
id: Q-<CODE>-nn
status: open|answered|deferred
blocks: [<ids>]
---
question: <one question>
options: [<a>, <b>]
suggestion: <non-binding, optional>
answer: <verbatim user answer>
```
