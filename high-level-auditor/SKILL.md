---
name: high-level-auditor
description: Audit docs/high-level-spec/ for legal/compliance exposure and logic holes (edge cases, contradictions, unreachable or abusable flows) before anything is built, and modify the spec in place. Legal obligations are written directly as constraints; feature/workflow/scope changes are proposed and applied on confirmation. Manual invocation only.
argument-hint: "[scope: HLS domain codes or feature IDs; default all]"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, WebSearch, WebFetch
---

# high-level-auditor

INPUT: `docs/high-level-spec/**` (product-architect). Missing → halt.
SCOPE: `$ARGUMENTS` = optional filter. Empty → all.
OUTPUT: in-place edits under `docs/high-level-spec/**` + findings in `docs/high-level-spec/audit/`.
STANCE: adversarial. Assume every workflow fails, every actor misbehaves, every regulation applies until shown otherwise. Report nothing that isn't traced to a spec ID.
NOT LEGAL ADVICE: legal findings are risk flags. `blocker`/`major` legal findings carry `counsel: recommended`.

## Rules
- R1 Every finding traces to spec IDs (`PW-F03`, `A-02`, `G-K01`, workflow step `PW-F03-W1.4`). No trace → not a finding.
- R2 Legal claims cite the instrument and article/section. Cite only what was verified via WebSearch/WebFetch this run (`ref: <url>`). Unverified → `verified: false`, severity capped at `major`.
- R3 Jurisdiction never assumed. Read it from spec (markets, users, operator location). Absent or partial → question first; legal checks for an unconfirmed jurisdiction are `conditional`.
- R4 Applicability never assumed. An obligation depends on facts the spec doesn't state (volume, user age, data category, business model) → question, not obligation.
- R5 Edit authority:
  - Direct (no confirmation): create finding files; add obligation constraint files (`G-Knn` / `<CODE>-Knn`) for obligations whose jurisdiction AND applicability are confirmed; link `findings` in affected files' frontmatter; set affected items `status: draft` while any blocker/major finding is open; create HLS question files; regenerate `index.md`.
  - Confirmation required: any change to product summary, scope in/out, actors, domains, features, workflows, rules, data concepts; any withdrawal.
- R6 No feature-design decisions. Fixes are options with a `(recommended)` marker. The user picks; the choice is applied with `src: [AUD-nn, Q-…]`.
- R7 Keep HLS invariants intact: product-architect templates, atomic one-ID-per-file, IDs/filenames never renumbered or renamed, removed → `status: withdrawn`, no technical content (no tech, schemas, APIs, UI).
- R8 Risk acceptance allowed. The user may reject a fix: finding `status: accepted-risk`, verbatim rationale recorded, affected items return to their prior status.
- R9 Writes only under `docs/high-level-spec/`. `docs/tech-spec/` exists → final reply notes that downstream items for edited IDs will be marked stale by system-architect.

## Mode
- Plan mode: P1–P5 interactively, render findings + full edit set (one `### <path>` block per file) into plan file, ExitPlanMode. On approval → P6.
- Normal: P1–P6, AskUserQuestion.
- Non-interactive: P1–P4, then direct edits only (R5), all fixes `proposed`.

AskUserQuestion: short labels, `(recommended)`, context line = AUD id + severity. Batch by domain, blockers first.

## Checklists
Derive applicability from spec signals. Lists are non-exhaustive; extend them from the spec's domain.

### Legal / compliance
- Personal data: lawful basis per purpose, consent capture/withdrawal, data subject rights (access, erasure, portability, objection), retention, special categories, profiling, cross-border transfers, processors/third parties, impact-assessment triggers, breach notification.
- Tracking/marketing: cookies/device storage, email/SMS opt-in, unsubscribe.
- Consumer: pre-contract information, price transparency, withdrawal/cancellation rights, subscription renewal and cancellation, unfair terms, dark patterns, reviews authenticity.
- Minors: age determination, parental consent, age-restricted goods/content.
- Accessibility obligations for the product type and markets.
- Payments/finance: payer authentication, refunds/chargebacks, KYC/AML triggers, invoicing/tax records, holding customer funds.
- Platforms/UGC: notice-and-action, moderation transparency, illegal content, IP/licensing of uploads, trader identification.
- AI: risk classification, disclosure when users interact with AI or see generated content, automated decisions with significant effect.
- Sector triggers: health, employment/gig work, gambling, location/biometric data, critical services security, sanctions/export.
- Terms/policies the spec implies but never lists (ToS, privacy notice, cookie notice, imprint).

### Logic / edge cases
- Lifecycle: every entity has create, change, end (delete, expire, cancel, archive) and, where relevant, restore. Missing ends = hole.
- State: every workflow defines failure, abandonment, retry, timeout, partial completion.
- Concurrency: two actors on one item; the same actor on two devices; action on an item deleted or changed mid-flow.
- Authority: role removed mid-action, owner leaves/dies/is banned, ownership transfer, last admin, self-escalation.
- Cardinality/boundaries: zero, one, many, limits, empty states, first-run, duplicates.
- Time: time zones, deadlines crossing midnight/DST, expiry during use, backdating.
- Money: partial refunds, price change during checkout, currency, free vs paid transitions, failed renewal.
- Identity: duplicate accounts, account merge, email change/loss, impersonation, shared accounts.
- Abuse: spam, fraud, scraping, harassment, fake accounts, resource exhaustion by a user, gaming of incentives.
- External dependency: a named third party unavailable, slow, or returning wrong data (behavior only, no mechanism).
- Consistency: contradictions between features/rules/constraints; actors referenced but not defined; workflows unreachable from any entry point; data concepts with no owner or no creator; capabilities with no actor.

## Procedure
P1 Ingest. Read HLS index, then in-scope files and existing `audit/` (update mode: preserve AUD IDs, re-check open findings, mark `resolved` where the spec now satisfies them — cite the change).

P2 Jurisdiction and facts. Extract markets, user types, data categories, monetization, operator location. Missing → first question batch.

P3 Sweep. Run both checklists per domain/feature/workflow step. Verify legal claims on the web (R2). Each hit → finding with kind, severity, trace, evidence, 2–3 fix options, affected IDs.

P4 Triage. Dedupe; merge findings with the same root cause; order by severity.
- `blocker`: unlawful as specified, or a core workflow cannot complete.
- `major`: obligation missing, or a likely edge case with user/business harm.
- `minor`: gap with limited harm.
- `note`: worth recording, no action required.

P5 Resolve. Ask the user per finding (blockers first): fix option / accepted-risk / defer / not-applicable (with reason → updates facts, may retire other findings).

P6 Write. Apply direct edits (R5), then confirmed fixes via product-architect templates. Continue ID numbering. New/changed items: `src` includes AUD + Q ids. Regenerate `index.md`. Final reply: finding counts by severity × status, edited/created/withdrawn paths, open questions, `counsel: recommended` count, downstream-stale note. Nothing else.

## IDs
Finding `AUD-nn`. Questions and constraints use HLS formats (`Q-<CODE>-nn`, `G-Knn`, `<CODE>-Knn`) so downstream skills treat them natively (open questions block via `blocks`).

## Layout additions
```
docs/high-level-spec/
  audit/
    AUD-nn-<slug>.md
    facts.md          # jurisdiction + applicability facts, each with src
```

## Templates

### audit/facts.md
```markdown
---
updated: <YYYY-MM-DD>
---
| fact | value | src |
|---|---|---|
| jurisdictions | <…> | <HLS id / Q id> |
| user types incl. minors | <…> | … |
| data categories | <…> | … |
| monetization | <…> | … |
```

### audit/AUD-nn-<slug>.md
```markdown
---
id: AUD-nn
kind: legal|compliance|logic|edge-case|contradiction|abuse
severity: blocker|major|minor|note
status: open|fixed|accepted-risk|deferred|not-applicable|resolved
affects: [HLS ids]
jurisdiction: [<codes>|n/a]
verified: <true|false|n/a>
counsel: <recommended|n/a>
questions: [Q ids]
applied: [paths changed]
---
## Finding
<1–3 sentences: what breaks or what obligation is unmet>
## Trace
- <HLS id / workflow step>: <what the spec says or omits>
## Evidence
- <instrument, article> — ref: <url>
## Scenario
<concrete sequence showing the failure; logic findings only>
## Options
- O1 <spec-level fix> (recommended)
- O2 <alternative>
- O3 accept risk
## Resolution
selected: <O-n|accepted-risk|…> | date: <YYYY-MM-DD>
answer: <verbatim user answer>
```

### Affected HLS file frontmatter additions
```yaml
findings: [AUD-nn]
```
