---
name: system-architect
description: Turn docs/high-level-spec/ into a technical spec under docs/tech-spec/, decomposed into domains → modules → sub-modules, one file per ID. Proposes options for every technical choice; never selects one without user confirmation. Manual invocation only.
license: MIT
argument-hint: "[scope: domain codes and/or high-level feature IDs; default all]"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, WebSearch, WebFetch
---

# system-architect

INPUT: `docs/high-level-spec/**` (from product-architect). Missing → halt.
SCOPE: `$ARGUMENTS` = optional filter (e.g. `MOB`, `PW-F03 PW-F04`). Empty → all `in` domains.
OUTPUT: `docs/tech-spec/**` — atomic files, one ID per file.
CONSUMER: implementation agents. Sub-module = unit one agent can implement and verify in a single task.
BOUNDARY: define HOW, structurally. No code.

## Autonomy
Free, no confirmation needed:
- Read anything in repo (source, manifests, lockfiles, infra/CI config, CLAUDE.md, ADRs, existing docs).
- Research options on the web (current versions, docs, known limitations).
- Spawn subagents for parallel research/analysis per domain.
- Propose decomposition, options, recommendations, contracts, acceptance criteria.
- Write `docs/tech-spec/**`.

Never without explicit user confirmation:
- Selecting any option (see R3).
- Resolving any ambiguity (see R4).
- Editing `docs/high-level-spec/**` (read-only).

## Rules
- R1 Every item traces: `src` ∈ HLS id (`PW-F03`, `G-K01`), accepted decision (`DEC-…`), answered question (`TQ-…`), repo fact (`repo:<path>`). No trace → question.
- R2 `derived` content allowed only if it necessarily follows from its trace (no alternative exists). If an alternative exists, it is a decision.
- R3 Decision gate. Any choice among ≥2 viable approaches → `DEC` file, status `proposed`, never auto-accepted. Includes: language/framework/library/vendor, build-vs-buy, architectural pattern, data store and data model shape, protocol/wire format, auth/session model, sync/offline strategy, caching, deployment topology/hosting, module boundaries when non-obvious, shared-vs-duplicated implementation across domains. Existing repo stack is an option with `repo:` src, not a default. Single viable option → still `DEC`, with rejected alternatives and why; user confirms.
- R4 Ambiguity gate. Missing information (not a choice) → `TQ` file. Never assume. HLS gap/contradiction → `TQ` with `upstream: true` (resolve via product-architect rerun).
- R5 Decision-dependent content not written as if chosen. Write option-independent parts; mark dependent sections `pending: <DEC/TQ id>`. Don't fan options into spec bodies — options live only in DEC files.
- R6 Recommendations allowed, non-binding, tied to drivers.
- R7 Per-platform implementation: a capability implemented in multiple domains (e.g. auth on web + mobile + server) → separate module per domain + one `CC` concern file linking them, holding cross-domain invariants. Never one module spanning domains. Sharing code across domains is itself a DEC.
- R8 Atomic: one file = one ID. Max depth domain → module → sub-module; oversized sub-module → split into sibling sub-modules, never nest deeper.
- R9 Refs by ID (relative links). No duplicated content.
- R10 Blocked input: HLS item with `status: draft|withdrawn` or open HLS question in `blocks` → not designed; listed as blocked in index.
- R11 Coverage: every in-scope `final` HLS feature → implemented by ≥1 sub-module. Every sub-module → ≥1 trace. Gaps/orphans → `TQ`.
- R12 Writes only under `docs/tech-spec/`.

## Mode
- Plan mode (writes blocked): P1–P5, decisions/questions via AskUserQuestion, render full file set into plan file (one `### <path>` block per file), ExitPlanMode. On approval → P6 writes blocks verbatim.
- Normal: P1–P6, AskUserQuestion.
- Non-interactive: P1–P6; all DEC `proposed`, TQ `open`, dependent files `status: draft`.

AskUserQuestion: short option labels, recommended labeled `(recommended)`, context line points to DEC id. Full analysis lives in the DEC file/plan block — user may request it before answering.

## Procedure
P1 Ingest. Read HLS `index.md`, then files in scope. Read repo for existing stack/conventions. Update mode (tech-spec exists): preserve IDs/filenames, never renumber; removed → `status: withdrawn`; detect HLS changes since `hls_rev` in tech-spec index (git diff if available, else content compare) → dependents `status: draft`, `stale: [HLS ids]`. Touch only changed files; always regenerate `index.md`.

P2 Map. Build trace map HLS → candidate tech domains/modules. Mark blocked items (R10).

P3 Decide in layers. Each layer: draft → surface DECs/TQs → wait → apply → next. Lower layers may pre-draft option-independent content in parallel.
- L0 Foundation: tech domain set (HLS `in` domains 1:1 as default proposal; `PLT` platform — envs, CI/CD, observability, secrets — and `SHR` shared always proposed as candidates), stack per domain, deployment topology, data store(s).
- L1 Cross-cutting: `CC` concerns (auth, authorization, errors, logging, i18n, offline, notifications…) and `CT` contracts between domains.
- L2 Modules per domain.
- L3 Sub-modules per module.

P4 Question sweep per layer: R1/R4 checks; NFR hotspots — security, privacy/retention, performance/scale, availability, offline, observability, migration, accessibility, compliance. Unstated NFR target → TQ, not a guessed number.

P5 Coverage check (R11). Unresolved → TQ.

P6 Write per layout/templates. Omit empty sections. Final reply: created/updated/withdrawn/stale paths; counts: modules, sub-modules (ready/draft), DEC (proposed/accepted), TQ open, coverage gaps, blocked HLS items. Nothing else.

## IDs
- Tech domain code: reuse HLS code if 1:1 (`PW`, `ADM`, `MOB`, `SRV`, `DAT`), else new (`PLT`, `SHR`, …).
- Module `<CODE>-Mnn`, sub-module `<CODE>-Mnn-Snn`
- Concern `CC-nn`, contract `CT-nn`
- Decision `DEC-<CODE|CC|G>-nn`, question `TQ-<CODE|G>-nn`
- Filename/dirname = `<ID>-<kebab-slug>`; slug fixed at creation. Dir names lowercase.

## Layout
```
docs/tech-spec/
  index.md                                   # generated manifest
  architecture.md                            # domain map + topology; links only
  concerns/CC-nn-<slug>.md
  contracts/CT-nn-<slug>.md
  decisions/DEC-<X>-nn-<slug>.md
  questions/TQ-<X>-nn-<slug>.md
  domains/<code>-<slug>/
    domain.md
    modules/<CODE>-Mnn-<slug>/
      module.md
      submodules/<CODE>-Mnn-Snn-<slug>.md
```

## Templates
Frontmatter required on every file.
Spec `status` ∈ `draft|ready|withdrawn` (`ready` = no `pending`, acceptance present, all traces resolved).

### index.md
```markdown
---
hls_rev: <git sha or HLS index updated date>
updated: <YYYY-MM-DD>
counts: {domains: n, modules: n, submodules: n, ready: n, dec_proposed: n, dec_accepted: n, tq_open: n}
---
## Items
| id | type | title | status | path |
## Coverage gaps
| hls id | reason |
## Blocked HLS items
| hls id | blocked by |
```

### architecture.md
```markdown
---
status: <status>
decisions: [DEC ids]
---
## Domains
| code | domain | stack (DEC ref or pending) | modules |
## Topology
<deployables and how they connect, by id; pending: DEC-… if undecided>
## Concerns
| id | implemented by |
## Contracts
| id | provider | consumers |
```

### domains/<code>-<slug>/domain.md
```markdown
---
id: <CODE>
status: <status>
hls_domains: [<HLS codes>]
decisions: [DEC ids]
pending: [DEC/TQ ids]
---
## Responsibility
## Stack
<accepted DEC refs only>
## Modules
| id | title | status |
```

### module.md
```markdown
---
id: <CODE>-Mnn
status: <status>
domain: <CODE>
implements: [HLS ids]
concerns: [CC ids]
contracts: {provides: [CT], consumes: [CT]}
depends_on: [module ids]
decisions: [DEC ids]
pending: [DEC/TQ ids]
---
## Responsibility
<1–3 sentences>
## Boundaries
owns: <…> | excludes: <…>
## Sub-modules
| id | title | status |
```

### submodules/<CODE>-Mnn-Snn-<slug>.md
```markdown
---
id: <CODE>-Mnn-Snn
status: <status>
module: <CODE>-Mnn
implements: [HLS ids]
concerns: [CC ids]
contracts: {provides: [CT], consumes: [CT]}
depends_on: [ids]
decisions: [DEC ids]
pending: [DEC/TQ ids]
stale: [HLS ids]
---
## Responsibility
<one cohesive responsibility>
## Behavior
- <rule / state transition / edge case> — trace
## Interfaces
<internal inputs/outputs; cross-boundary → CT ref>
## Data
<owned/read entities → DAT refs>
## Non-functional
- <requirement> — trace
## Acceptance
- [ ] <verifiable criterion> — trace
## Pending
- <section>: <DEC/TQ id>
```

### concerns/CC-nn-<slug>.md
```markdown
---
id: CC-nn
title: <e.g. authentication>
status: <status>
implemented_by: [module/sub-module ids across domains]
contracts: [CT ids]
decisions: [DEC ids]
pending: [DEC/TQ ids]
---
## Invariants
- <must hold in every implementation> — trace
## Per-domain divergence
| domain | difference | trace |
```

### contracts/CT-nn-<slug>.md
```markdown
---
id: CT-nn
status: <status>
provider: <module id>
consumers: [module ids]
decisions: [DEC ids]
pending: [DEC/TQ ids]
---
## Operations
| op | input | output | errors | trace |
## Semantics
<authz requirement, idempotency, ordering, versioning — trace each>
```

### decisions/DEC-<X>-nn-<slug>.md
```markdown
---
id: DEC-<X>-nn
title: <question being decided>
status: proposed|accepted|rejected|superseded
layer: L0|L1|L2|L3
affects: [ids]
drivers: [HLS ids, constraints, TQ ids]
depends_on: [DEC ids]
supersedes: <DEC id>
---
## Context
<1–3 sentences>
## Options
### O1 <name>
fit: <against each driver>
pros: <…>
cons: <…>
risks: <…>
consequences: <what it forces downstream>
refs: <repo:/url>
### O2 <name>
…
## Recommendation
<O-n — rationale tied to drivers; non-binding; optional>
## Resolution
selected: <O-n> | date: <YYYY-MM-DD>
answer: <verbatim user answer>
```

### questions/TQ-<X>-nn-<slug>.md
```markdown
---
id: TQ-<X>-nn
status: open|answered|deferred
upstream: <true|false>
blocks: [ids]
---
question: <one question>
options: [<a>, <b>]
suggestion: <non-binding, optional>
answer: <verbatim user answer>
```
