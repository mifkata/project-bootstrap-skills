---
name: beads-manager
description: Materialize docs/high-level-spec, docs/tech-spec and docs/roadmap into a beads (bd) work graph — domain epic → feature epic → sub-feature epic → tasks/stories/bugs/chores/spikes — serially chained so exactly one sub-feature epic is executable at a time behind a review gate. Interactive; proposes, never decides. Manual invocation only.
license: MIT
argument-hint: "[scope: milestone IDs, tech domain codes or module IDs; default all accepted milestones]"
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash(bd version:*), Bash(bd where:*), Bash(bd info:*), Bash(bd list:*), Bash(bd show:*), Bash(bd ready:*), Bash(bd gate list:*), Bash(bd epic status:*), Bash(bd lint:*), Bash(bd types:*)
---

# beads-manager

INPUT: `docs/high-level-spec/**`, `docs/tech-spec/**`, `docs/roadmap/**`. Any missing → halt.
SCOPE: `$ARGUMENTS` = optional filter. Empty → all accepted roadmap milestones.
OUTPUT: bd graph (source of truth) + run log `docs/beads/runs/<YYYYMMDD-HHMM>.md`.
CONSUMER: agentic factory loop: `bd ready --exclude-type epic,milestone --json` → claim → implement → close → `bd epic close-eligible` → reviewer runs `bd gate resolve` → next sub-feature epic unblocks.

## Hierarchy (derived, fixed depth 4)
| level | bd type | source | external-ref | spec-id |
|---|---|---|---|---|
| domain epic | epic | tech-spec domain | `ts:<CODE>` | `…/domain.md` |
| feature epic | epic | tech-spec module | `ts:<CODE>-Mnn` | `…/module.md` |
| sub-feature epic (SFE) | epic | tech-spec sub-module / roadmap spike | `ts:<CODE>-Mnn-Snn` / `rm:SPK-nn` | sub-module / spike file |
| work item | task, story, bug, chore, spike | SFE decomposition | `<SFE ref>#Tnn` | SFE spec file |
| milestone | milestone | roadmap milestone | `rm:MS-nn` | milestone file |
| review gate | gate (human) | one per SFE | `review:<SFE ref>` | — |

Spike SFE parent = feature epic of the module it de-risks.

Work item type:
- `story`: user-visible behavior tracing to an HLS workflow.
- `task`: internal implementation.
- `chore`: setup, config, scaffolding, verification.
- `spike`: timeboxed investigation.
- `bug`: only for a stated defect in existing code, or (update mode) closed work contradicting an updated spec. Never invent.

## Invariants
- I1 Parent chain exact: work item → SFE → feature epic → domain epic. No work item under a feature or domain epic.
- I2 Serial chain. All in-scope SFEs form one total order. SFE[n+1] is blocked by SFE[n] and by Review gate[n]. The last SFE's gate blocks its milestone. Result: `bd ready --exclude-type epic,milestone` returns items of exactly one SFE.
- I3 Cross-SFE ordering only via the chain. A dependency pointing to a later SFE = violation → question with `upstream: roadmap`. Intra-SFE dependencies allowed.
- I4 Reviewable SFE: one coherent change set, ~≤8 work items. Oversized → question `upstream: tech` (split the sub-module via system-architect); a local split `<SFE ref>/pN` only if the user accepts it.
- I5 Verification: if accepted as a convention, the last work item of every SFE is a `chore` "Verify <SFE ref>" (label `verify`), blocked by all its siblings, checking the spec's Acceptance.
- I6 Every bead has title, `-d`, `--acceptance`, `--spec-id`, `--external-ref`. Work items also have `--design` with spec ID refs (sections, CT, CC, DEC). `bd lint` must pass.
- I7 Idempotent. Look up by external-ref (`bd list --external-ref <ref> --status open,in_progress,blocked,deferred,closed --json`, review gates via `bd gate list --all --json`) before any create. Never duplicate. Never `bd delete`.
- I8 Coverage. Every Acceptance item of a materialized sub-module is covered by ≥1 work item's acceptance.

## Input readiness
- Materialize only sub-modules with tech-spec `status: ready` inside accepted roadmap milestones. Draft, pending or stale → skip and report.
- Roadmap decision gates → their items are not materialized.
- Open roadmap external gates → human gate `External: <GT title>` blocking the first SFE that needs it.
- Convergence gates → must be satisfied by chain order; else violation.
- More than one roadmap track → interleaving into the single chain is a decision.

## Decision gate
Propose, never decide:
- work item decomposition and types per SFE
- intra-SFE order not forced by the spec
- track interleaving
- SFE splits
- placement of new SFEs in an existing chain
- priorities (default: unset)
- changes to in_progress or closed beads
- closing beads for withdrawn specs
- `bd init`
- conventions

Derived, no confirmation: hierarchy mapping, chain order from a single accepted track, labels, spec links, acceptance text copied from the spec.

Ambiguity → question `BQ-nn`, never assume.

Interaction via AskUserQuestion: short labels, `(recommended)` marker, batches per feature epic. "Approve whole batch" is always an option.

## Mode
- Plan mode: P0–P5 (read-only `bd` allowed), render run log + full command script into the plan file, ExitPlanMode. On approval → P6.
- Normal: P0–P5, write run log `status: planned`, ask approval of the whole plan, then P6.
- No user channel → halt after P1 with report. Execute nothing.

## Procedure
P0 Preflight.
- `bd version` fails → halt, report install (`brew install beads` / `npm i -g @beads/bd`).
- `bd where` shows no database → question: `bd init` vs `bd init --skip-agents` (default init edits AGENTS.md and installs hooks). Never run it unconfirmed.

P1 Ingest.
- Read the three index files, then in-scope files; apply readiness.
- Load existing beads by external-ref prefixes `ts:`, `rm:`, `review:`.
- Classify each spec unit: create / update (open and untouched, spec changed) / unchanged / locked (in_progress or closed) / orphan (spec withdrawn).

P2 Conventions (first run, or when changed):
- verification chore yes/no
- review gate type `human` | `gh:pr`
- label scheme
- priority policy
- store the factory protocol via `bd remember` yes/no (text below)

P3 Chain.
- Order SFEs from roadmap milestone order → item order.
- Validate every tech-spec and roadmap hard edge points backward in the chain (I3).
- Update mode: new SFEs may be inserted only among not-started SFEs; the rewiring goes into the plan.

P4 Decompose each SFE from its sub-module:
- Behavior → stories/tasks
- provided CT operations → tasks
- Data → tasks
- NFR → tasks, or acceptance on existing items
- setup → chores
- Acceptance → distribute (I8)

Each item traces to spec IDs/sections. Locked beads: propose a follow-up task/bug in a new SFE. Orphans: propose close with reason or `bd supersede`.

P5 Plan. Build the ordered op list and command script; validate I1–I8 on the plan. Get approval.

P6 Execute top-down:
1. domain → feature → SFE → work items (capture IDs with `--silent`)
2. intra-SFE deps
3. chain deps
4. review gates
5. external gates
6. milestones and their deps

On first error: stop, record, report. Never retry or roll back writes without asking.

Post-checks:
- `bd lint`
- assert `bd ready --exclude-type epic,milestone --json` ⊆ chain-head SFE descendants (`--parent <head>`)

Update the run log with bead IDs and check results.

P7 Final reply: counts (created/updated/unchanged/locked/skipped/orphan), chain head, lint result, assertion result, open BQs. Nothing else.

## Command patterns (bd ≥1.3)
```bash
D=$(bd create "<domain>" -t epic --external-ref "ts:PW" --spec-id "<path>" -d "<responsibility>" --acceptance "<success criteria>" -l "dom:pw" --silent)
F=$(bd create "<module>" -t epic --parent "$D" --external-ref "ts:PW-M01" --spec-id "<path>" -d "…" --acceptance "…" -l "mod:pw-m01" --silent)
S=$(bd create "<sub-module>" -t epic --parent "$F" --external-ref "ts:PW-M01-S01" --spec-id "<path>" -d "…" --acceptance "<sub-module Acceptance>" -l "sfe,ms:ms-01" --silent)
T=$(bd create "<item>" -t task --parent "$S" --external-ref "ts:PW-M01-S01#T01" --spec-id "<path>" -d "…" --design "refs: PW-M01-S01 §Behavior.2; CT-03; DEC-SRV-02" --acceptance "…" --silent)
bd dep add "$T2" "$T1"                    # intra-SFE: T2 after T1
bd dep add "$S_NEXT" "$S_PREV"            # chain
G=$(bd gate create --type=human --blocks "$S_NEXT" --title "Review: ts:PW-M01-S01" --reason "Review before next SFE" --json | jq -r .id)
bd update "$G" --external-ref "review:ts:PW-M01-S01"
M=$(bd create "<milestone>" -t milestone --external-ref "rm:MS-01" --spec-id "<path>" -d "…" --acceptance "<exit criteria>" --silent)
bd dep add "$M" "$S"                      # milestone waits for each of its SFEs
```
Labels inherit from parent. Set each label only at the level that introduces it.

## Factory protocol (for `bd remember`, if accepted)
`Work only items from: bd ready --exclude-type epic,milestone --json. Items outside the current SFE never appear; do not work around blockers. After last item: bd epic close-eligible. Stop at review gate; never resolve gates.`

## Run log template
```markdown
---
run: <YYYYMMDD-HHMM>
status: planned|applied|failed
revs: {hls: <sha/date>, tech: <sha/date>, roadmap: <sha/date>}
scope: <args or all>
---
## Questions
| id | question | answer | upstream |
## Plan
| # | op | ref | type | parent ref | deps | title |
## Commands
<full script>
## Result
| ref | bead id | op | status |
## Checks
lint: <pass|fail + ids> | single-SFE ready: <pass|fail> | coverage: <gaps>
## Skipped
| ref | reason |
```
