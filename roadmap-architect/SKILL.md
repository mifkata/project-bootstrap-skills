---
name: roadmap-architect
description: Sequence docs/high-level-spec/ and docs/tech-spec/ into an implementation roadmap under docs/roadmap/ — dependency graph, gates, milestones. Interactive only; proposes every ordering/grouping choice, never selects alone. Manual invocation only.
argument-hint: "[scope: tech domain codes, module IDs or HLS feature IDs; default all]"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep
---

# roadmap-architect

INPUT: `docs/high-level-spec/**` (product-architect), `docs/tech-spec/**` (system-architect). Either missing → halt.
SCOPE: `$ARGUMENTS` = optional filter. Empty → all.
OUTPUT: `docs/roadmap/**` — atomic files, one ID per file.
CONSUMER: user + implementation agents picking next work.
UNIT: tech-spec sub-module (`<CODE>-Mnn-Snn`). Module ID = all its sub-modules.

## Interaction
- Interactive at all times via AskUserQuestion. No user channel → halt, reply with pending proposals list; write nothing.
- Plan mode: P1–P6 interactively, render full file set into plan file (one `### <path>` block per file), ExitPlanMode. On approval → P7 writes verbatim.
- Options: short labels, recommended labeled `(recommended)`, context line points to RDEC id. Full analysis in RDEC file/plan block on request.

## Rules
- R1 Read-only on HLS and tech-spec. Gap/contradiction/cycle there → `RQ` with `upstream: hls|tech`.
- R2 Derived (no confirmation): hard edges from tech-spec traces, graph, critical path, readiness, validation results. Each hard edge cites its source (`depends_on`, `CT` provider→consumer, `CC` foundation, `DEC.depends_on`).
- R3 Decision gate → `RDEC`, status `proposed`, never auto-accepted: sequencing strategy, milestone boundaries/composition, order among independent items, soft edges (any "better before" not forced by spec), parallel tracks, spikes, deferral/cut of scope, release grouping, gate placement.
- R4 Ambiguity (missing info, e.g. deadlines, capacity, external lead times) → `RQ`. Never assume.
- R5 No dates, durations or effort unless user supplies capacity and asks. Then relative only, as RDEC.
- R6 Readiness: item with `pending` DEC/TQ or `stale` may be placed only behind a decision gate naming those ids. Never presented as unblocked.
- R7 Never defer, drop, or reorder accepted content silently. Spec change breaking an accepted milestone → flag violation + propose fixes as RDEC; don't move items.
- R8 Every in-scope item in exactly one milestone or explicitly deferred (RDEC). Milestone order must respect all hard and accepted soft edges.
- R9 Atomic: one file = one ID. Refs by ID. Generated files never source of truth.
- R10 Writes only under `docs/roadmap/`.

## Procedure
P1 Ingest. Read HLS and tech-spec indexes, then in-scope files. Collect readiness (`ready|draft|stale`, pending ids). Update mode (roadmap exists): preserve IDs/filenames, never renumber; removed → `withdrawn`; re-derive graph; list violations of accepted milestones (R7). Touch only changed files; always regenerate `index.md`, `graph.md`, `roadmap.md`.

P2 Graph. Nodes: sub-modules, CT, CC. Hard edges (R2). Detect cycles → RQ upstream. Compute critical path and parallelizable sets.

P3 Gates. Identify:
- `convergence`: item needs a set of items done (all-of). E.g. payment integration waits for catalog + cart + order flow; end-to-end analytics waits for all event-emitting features.
- `external`: prerequisite outside the codebase — vendor account/sandbox/API keys, contracts, app store accounts, legal/compliance sign-off, data migration windows. Owner = user.
- `decision`: unresolved DEC/TQ (R6).
Convergence gates from hard edges = derived; any proposed gate = RDEC. External gates → RQ for lead time/status.

P4 Criteria. Ask user (first batch): priority driver (value-first / risk-first / dependency-first / fixed-date demo), hard deadlines or release targets, parallelism available (people/agents), must-first features, external dates. Unanswered → RQ.

P5 Strategy. Propose 2–3 sequencing strategies as one RDEC, each with milestone sketch, critical path impact, risk profile. Common options: walking skeleton (thin end-to-end slice first), foundation-first (L0/L1 concerns + contracts, then features), risk-first (spikes on high-risk DECs/integrations early), value-first (highest-value HLS features end-to-end). Wait for selection.

P6 Milestones. Under selected strategy propose milestones in batches: composition, order, entry (milestones/gates), outcome (HLS features/workflows delivered end-to-end), soft edges, spikes (`SPK-nn`, roadmap-only, trace to DEC/risk), deferrals, tracks. Each → RDEC; wait. Validate R8 after each batch.

P7 Write per layout/templates. Omit empty sections. Final reply: created/updated/withdrawn paths; counts: milestones, gates (open), RDEC (proposed/accepted), RQ open, unscheduled items, violations. Nothing else.

## IDs
Milestone `MS-nn`, gate `GT-nn`, spike `SPK-nn`, track `TR-nn`, decision `RDEC-nn`, question `RQ-nn`. Filename = `<ID>-<kebab-slug>.md`; slug fixed at creation.

## Layout
```
docs/roadmap/
  index.md                    # generated manifest + validation
  roadmap.md                  # generated: ordered milestones by track
  graph.md                    # generated: nodes, edges, critical path, cycles
  milestones/MS-nn-<slug>.md
  gates/GT-nn-<slug>.md
  spikes/SPK-nn-<slug>.md
  tracks/TR-nn-<slug>.md
  decisions/RDEC-nn-<slug>.md
  questions/RQ-nn-<slug>.md
```

## Templates
Frontmatter required. Roadmap item `status` ∈ `proposed|accepted|superseded|withdrawn`.

### index.md
```markdown
---
hls_rev: <sha/date>
tech_rev: <sha/date>
updated: <YYYY-MM-DD>
counts: {milestones: n, gates_open: n, rdec_proposed: n, rdec_accepted: n, rq_open: n, unscheduled: n, violations: n}
---
## Items
| id | type | title | status | path |
## Unscheduled
| item | reason |
## Violations
| milestone | edge/gate broken | proposed fix (RDEC) |
```

### roadmap.md
```markdown
---
strategy: <RDEC id → selected option>
updated: <YYYY-MM-DD>
---
| order | milestone | track | entry | delivers (HLS) | status |
```

### graph.md
```markdown
---
updated: <YYYY-MM-DD>
---
## Nodes
| id | type | readiness | pending |
## Edges
| from | to | kind: hard|soft | source | status |
## Critical path
<ordered ids>
## Cycles
<id chains → RQ ids>
```

### milestones/MS-nn-<slug>.md
```markdown
---
id: MS-nn
title: <outcome name>
status: <status>
order: <n>
track: <TR id>
entry: [MS/GT ids]
delivers: [HLS feature/workflow ids]
items: [sub-module/SPK ids, ordered]
decisions: [RDEC ids]
pending: [RDEC/RQ/GT ids]
---
## Outcome
<what is usable end-to-end after this milestone>
## Sequence
| # | item | after | edge kind | source |
## Exit criteria
- [ ] <HLS feature/workflow usable; tech-spec acceptance refs>
## Deferred
- <item> — RDEC id
```

### gates/GT-nn-<slug>.md
```markdown
---
id: GT-nn
kind: convergence|external|decision
status: open|satisfied|withdrawn
waits_for: [ids | external description]
unblocks: [ids]
owner: <user|agent>
source: <derived edge refs | RDEC id>
pending: [RQ ids]
---
<one-line condition>
```

### spikes/SPK-nn-<slug>.md
```markdown
---
id: SPK-nn
status: <status>
reduces_risk_of: [DEC/item ids]
unblocks: [ids]
decisions: [RDEC ids]
---
question: <what the spike must answer>
exit: <evidence required to resolve target DEC>
```

### tracks/TR-nn-<slug>.md
```markdown
---
id: TR-nn
status: <status>
milestones: [MS ids ordered]
decisions: [RDEC ids]
---
<one-line purpose, e.g. mobile client, platform>
```

### decisions/RDEC-nn-<slug>.md
```markdown
---
id: RDEC-nn
title: <what is being decided>
status: proposed|accepted|rejected|superseded
affects: [ids]
drivers: [RQ answers, HLS/tech ids, critical path]
depends_on: [RDEC ids]
supersedes: <RDEC id>
---
## Context
<1–3 sentences>
## Options
### O1 <name>
effect: <ordering/milestones produced>
pros: <…>
cons: <…>
risks: <…>
critical_path: <impact>
### O2 <name>
…
## Recommendation
<O-n — rationale tied to drivers; non-binding; optional>
## Resolution
selected: <O-n> | date: <YYYY-MM-DD>
answer: <verbatim user answer>
```

### questions/RQ-nn-<slug>.md
```markdown
---
id: RQ-nn
status: open|answered|deferred
upstream: <none|hls|tech>
blocks: [ids]
---
question: <one question>
options: [<a>, <b>]
suggestion: <non-binding, optional>
answer: <verbatim user answer>
```
