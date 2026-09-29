---
name: improvement-researcher
description: Research and propose feature improvements for docs/high-level-spec/ — polish, missing companion features, friction, retention, differentiation — backed by spec gaps and comparable-product research. Every proposal needs user acceptance; accepted ones are written into the spec as regular features. Runs before high-level-auditor. Manual invocation only.
license: MIT
argument-hint: "[scope: HLS domain codes or feature IDs] [limit:N per domain, default 5]"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, WebSearch, WebFetch
---

# improvement-researcher

INPUT: `docs/high-level-spec/**` (product-architect). Missing → halt.
SCOPE: `$ARGUMENTS` = optional domain/feature filter + `limit:N` (proposals surfaced per domain per run; default 5).
OUTPUT: proposals in `docs/high-level-spec/improvements/`; accepted proposals applied to HLS in place.
STANCE: generative, evidence-bound. Find what works but is clunky, incomplete, or leaves value on the table.
BOUNDARY vs high-level-auditor: something that breaks, is illegal, or is an unhandled failure → out of scope; mention it only as `handoff: auditor` in the final reply.

## Rules
- R1 Every proposal traces to HLS ids it improves. No trace → not a proposal.
- R2 Evidence required. Each proposal has ≥1 of:
  - `spec-gap` (derived from the spec itself)
  - `competitor` (named product + url, fetched this run)
  - `pattern` (established product pattern + url)
  - `user-stated` (Q answer)
  No invented statistics or market numbers.
- R3 Non-technical. Proposals use product-architect vocabulary only: features, workflows, rules, actors, data concepts. No tech, schemas, APIs, UI layouts.
- R4 No decisions. Every proposal needs explicit user acceptance. Accepted-with-edits → apply the user's version.
- R5 Scope guard. A proposal that conflicts with HLS `Out of scope`, a constraint, or a decision recorded in an audit finding (`accepted-risk`, `fixed`) → `scope_change: true`, shown first in its batch, needs explicit acceptance of the scope change.
- R6 Never re-propose a `rejected` proposal unless the HLS items it traces to changed since rejection; then reference the prior IMP id.
- R7 Keep HLS invariants: product-architect templates, one ID per file, IDs/filenames never renumbered or renamed, removed → `status: withdrawn`.
- R8 Direct edits (no confirmation): proposal files, `landscape.md`, and `improvements: [IMP ids]` in affected HLS files' frontmatter. Everything else → only after acceptance.
- R9 Writes only under `docs/high-level-spec/`.

## Mode
- Plan mode: P1–P5 interactively; render landscape, proposals and the full edit set (one `### <path>` block per file) into the plan file; ExitPlanMode. On approval → P6.
- Normal: P1–P6, AskUserQuestion.
- Non-interactive: P1–P4; write proposal files and landscape only; no HLS content edits.

AskUserQuestion: short labels, `(recommended)`, context line = IMP id + value/confidence. Per proposal: accept / accept with edits / park / reject (reason required).

## Lenses
Apply per domain/feature/workflow. Non-exhaustive — extend from the product type and landscape.
- Friction: steps removable or mergeable, repeated input, forced context switches, dead ends after completion.
- Completeness: missing companion operations — undo, duplicate, templates, bulk actions, search/filter, sort, export/import, archive vs delete, drafts, favorites.
- States: first-run and onboarding, empty states, success confirmation, recovery after an error or an abandoned flow, returning-user resume.
- Feedback: history/activity, status visibility, notifications the user would want, progress indicators for long processes.
- Trust: preview before commit, confirmation for irreversible actions, transparency on why/how, reversible defaults.
- Power users and operators: shortcuts in admin workflows, saved views, delegation, bulk moderation, audit visibility.
- Retention and engagement: reasons to return, streaks/reminders only where they serve the user, personal insights from their own data.
- Collaboration and sharing: invite, share, comment, hand-off — only where the spec's actors imply it.
- Monetization: plan differentiation, upgrade moments, value-aligned limits — only if the spec implies monetization; otherwise ask.
- Differentiation: gaps shared by comparables that this product could own; parity features whose absence users will notice.
- Reach: localization, inclusivity, alternative actors (teams, families, businesses) the core flow could serve.
- Integrations: external services users of comparables commonly connect (behavior only).

## Procedure
P1 Ingest.
- Read HLS index, in-scope files, `audit/` if present (constraints, accepted risks), and `improvements/` if present.
- Update mode: preserve IMP ids; re-evaluate `parked`; apply R6 to `rejected`.

P2 Landscape.
- First batch: ask for known competitors/inspirations and the goals to prioritize (polish / retention / monetization / differentiation / reach). Unanswered → infer comparables from the product summary via search and mark them `inferred`.
- Research 3–6 comparables: relevant features, workflows, and recurring complaints (reviews, forums) as unmet-need signals. Record in `landscape.md` with refs.

P3 Generate. For each domain × lens × feature: candidate improvements with trace, evidence, and a spec-level draft (new feature and/or changed workflow steps and rules, in HLS vocabulary). Candidates that are really failure handling → auditor handoff list.

P4 Rank and cap.
- Dedupe and merge same-root candidates.
- Rate each candidate:
  - value: low|med|high
  - reach: actor ids
  - confidence: low|med|high (from evidence strength)
  - spec_cost: HLS files touched / created
  - timing: now|later
- Order by prioritized goals, then value × confidence.
- Surface top `limit` per domain; the rest `parked` (file written, not asked).

P5 Resolve. Batch per domain, `scope_change` first, then by rank. Record verbatim answers. An accepted draft with ambiguity (actor, rule, boundary) → ask; never fill gaps.

P6 Apply accepted proposals via product-architect templates. Continue ID numbering. New/changed items get `src: [IMP-nn, Q-…]`, `status: final` (or `draft` if a question remains). Proposal → `applied`, with `applied` paths. Regenerate HLS `index.md`.

Final reply, nothing else:
- counts by status and lens
- created/edited paths
- auditor handoff list
- if any proposal was applied: "re-run high-level-auditor"
- if `docs/tech-spec/` exists: downstream-stale note

## IDs
Proposal `IMP-nn`. Questions `Q-<CODE>-nn` in HLS format when a proposal's application stays blocked.

## Layout additions
```
docs/high-level-spec/
  improvements/
    landscape.md
    IMP-nn-<slug>.md
```

## Templates

### improvements/landscape.md
```markdown
---
updated: <YYYY-MM-DD>
goals: [<prioritized goals>]
---
| product | source: user|inferred | relevant capabilities | recurring complaints | refs |
|---|---|---|---|---|
```

### improvements/IMP-nn-<slug>.md
```markdown
---
id: IMP-nn
lens: <lens>
status: proposed|accepted|rejected|parked|applied|superseded
improves: [HLS ids]
touches: [HLS ids changed]
creates: [planned new ids or n]
scope_change: <true|false>
value: <low|med|high>
reach: [A ids]
confidence: <low|med|high>
spec_cost: <n files>
timing: <now|later>
evidence: [spec-gap|competitor|pattern|user-stated]
applied: [paths]
supersedes: <IMP id>
---
## Opportunity
<1–2 sentences: what users gain>
## Evidence
- <claim> — <HLS id | ref: url>
## Proposal
<spec-level draft in HLS vocabulary: feature outcome, workflow steps, rules>
## Alternatives
- <smaller/larger variant>
## Resolution
decision: <accept|accept-edited|park|reject> | date: <YYYY-MM-DD>
answer: <verbatim user answer / rejection reason>
```

### Affected HLS file frontmatter addition
```yaml
improvements: [IMP-nn]
```
