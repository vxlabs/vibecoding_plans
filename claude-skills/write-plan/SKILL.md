---
name: write-plan
description: Write a new numbered module plan in docs/plans/NN-<slug>/ — README (goal, decisions, status, dependency table, acceptance) plus self-contained slice files that /run-plan can execute — and register it in docs/PLAN.md. Use when the user says "/write-plan", "write plan NN", "plan <feature>" or asks to create slices for a feature.
argument-hint: <feature description or plan number + goal>
---

# Write a plan for: $ARGUMENTS

You're writing for two readers:
- the user, who approves the decisions;
- `slice-worker` agents that start cold and read **only their slice file**.

Everything a slice needs must be in that slice file or be extractable from it with one command. Your output tokens are the most expensive part of this workflow, so don't write the same text twice. Run this on Opus: the slice boundaries decide whether `/run-plan` can run slices in parallel safely.

## 1. Understand the request

- Next plan number: `ls docs/plans | sort -n | tail -3`. Slug: short kebab-case.
- Read the `## Module execution order` table in `docs/PLAN.md` and the READMEs of the plans this one builds on (just the Goal and Decisions sections).
- Find the code you'll extend with grep and `ls`, not by reading whole modules. You need the module names, the existing public-api tokens, the schema files and the screens.
  - If the plan touches more than two modules, delegate the survey to one `Explore` agent (`model: sonnet`). Ask it for a brief of at most 40 lines covering paths, tokens and extension points, not file dumps. Read source yourself only where a decision depends on it.
- **A short request is normal.** Users often write one sentence ("we need Azure DevOps, we only have GitHub"). After the survey, draft the brief yourself: Goal, Users and what each role can do, In scope, **Out of scope**, and Builds on. Make the scope as small as is still useful, and put the rest under out of scope as "later". Show this brief in the same question round as the questions below, so the user corrects a draft instead of writing one.
- **Ask the user only about product decisions that change the design**: who can do what, one-way vs two-way, what's out of scope. Use AskUserQuestion, put your recommended option first, and keep it to at most one round. Decide engineering details yourself and record them as decisions.

## 2. Write `docs/plans/NN-<slug>/00-README.md`

Follow the existing plans (`docs/plans/33-azure-devops/00-README.md` is the reference):

```
# NN — <Title>

Master plan: [`../../PLAN.md`](../../PLAN.md) · Builds on: <links> · Used by: <links or omit>

## Goal
<who it's for and the outcomes, as short bullets>

## Decisions (already made; do not revisit)
| # | Decision |
| --- | --- |
| <L>1 | **<Name>.** <schema, rules, permissions, endpoints, jobs; concrete names, limits, enums, status codes> |

## Status
- [ ] 01 — <slice title>
...

## Dependencies and parallel tracks
| Slice | Prerequisites | Parallel with | Model | Files touched |

**Rules for every slice:** <only rules specific to this plan; the generic ones live in .claude/agents/slice-worker.md>

## Acceptance checklist
- [ ] <observable behaviour a person or test can check>
```

- Number the decisions with a plan-specific letter prefix that isn't used yet (`grep -ho '^| [A-Z]1 ' docs/plans/*/00-README.md | sort -u`).
- Every permission, limit, status code and name that a slice depends on must appear in a decision.
- The architecture rules in `AGENTS.md` apply: layers, Clerk-only humans, paise, UTC/date-only, encrypted write-only credentials, AI output as drafts only.
- Keep each decision on **one table row** (use `<br>` for structure), so slices can extract it with grep.

**Checkpoint:** stop after the README and ask the user to approve the decisions and the slice list in one message. Show the decision IDs with one-line summaries, the slice table, and any points you're unsure of. Write slice files only after approval. Changing a decision later means rewriting every slice that uses it.

## 3. Cut the slices

**Usual order:** schema → domain → infrastructure/gateways → application and API → runner (if any) → web → acceptance. Merge layers when a slice would be tiny; split when a slice would touch more than ~15 files or two modules' internals.

**Parallel safety** is the point of the dependency table:
- Two slices may be "Parallel with" each other only if their **Files touched** globs don't overlap. Check this pair by pair.
- Schema/migration slices never run in parallel. Client regeneration is not a reason to serialise: workers don't regenerate `packages/api-client`, `/run-plan` does after each gate, so don't list it in Files touched.
- A web slice depends only on the API slices whose endpoints it calls, not on the last backend slice. Look for that track pair by pair; it is usually the biggest parallel win.
- A web slice that changes user-visible behaviour also updates its Help topic (`apps/web/src/features/help/content/`) under the plan 35 J8 content rules: 300–900 words, for screen users, shipped behaviour only, no plan numbers, KI ids or env vars.
- When two slices must touch one file (a worker, a module file), give ownership to one and describe the other's change as a small, named, isolated edit. Plan 33's 06/08 split of `azdo-events.worker.ts` is the model.

**Model per slice:**
- **Haiku:** mechanical work that follows an existing pattern (DTO fields, a mapping table, docs-only edits).
- **Sonnet:** the default for real code.
- **Opus:** cross-module refactors, provider abstractions, security-sensitive auth or credential flows.

**Fewer, fuller slices.** Each slice pays a fixed cost: a cold-start agent, and a root typecheck and lint that takes minutes. Merge adjacent slices that would run one after the other anyway and together touch fewer than ~15 files. Split only to gain parallelism, a different model, or a module boundary — or when a slice would need more than ~4 existing files as read-only context; agent context, not file count, is what makes slices expensive (workers that read widely pass 150k tokens). For a typical plan, aim for 5–9 slices including acceptance.

The last slice is always acceptance (Opus + Haiku: runbook, known issues via the Sentinel MCP tools, the feature's Help topic (`apps/web/src/features/help/content/`) and checklist evidence).
## 4. Write each slice file `MM-<slug>.md`

`/run-plan` reads each slice only up to `## Tasks` when it schedules, so the header and Scope go first:

```
# NN/MM — <Title>

Plan folder: [`00-README.md`](00-README.md) · Model: <Sonnet> · Prerequisites: <NN/MM or none> · Parallel with: <NN/MM (+ the shared-file note) or none>

## Decisions you need
<IDs plus a one-line note on which part applies, then the command that extracts them, e.g.>
Z6 (webhook auth), Z7 (dedupe), Z8 (polling fallback) — run `grep -E '^\| (Z6|Z7|Z8) ' 00-README.md`.
<Quote text only for excerpts from OTHER plans (a few lines, never a whole section). Don't copy this plan's own decisions into the slice; the grep gives the agent the exact rows.>

## Read first
<the 3–8 specific existing files to read or extend, with paths. Never "read plan 11", only a named section if it's truly needed.>

## Scope (files you own)
<explicit paths/globs, including specs and e2e files>

## Tasks
1. **<Part>.** <precise: names, signatures, routes, status codes, edge cases>

## Done when
<named tests and observable outcomes; which existing suites must stay green; "Root typecheck and lint pass. Append `## Result`.">
```

- Quote the requirement text the slice needs (trimmed to this slice) instead of pointing at README rows like `L3`; agents grepping long README rows pull 20k+ characters each time.
- Under `Read first`, name the exact symbols or line ranges to read, not whole files.
- Write tasks precisely enough that the agent doesn't have to make product decisions: name the routes, DTO fields, error codes and test cases.
- Name existing extension points (`OUTBOX_SUBSCRIBERS`, `application/public-api.ts`, ports) so the agent reuses them instead of inventing new ones.
- Note known traps that apply, for example: the tsvector DROPs in `migrate diff` output, the e2e file filter.

## 5. Register and cross-link

- Add the row to the `## Module execution order` table in `docs/PLAN.md` (`| NN | [Title](plans/NN-slug/00-README.md) | summary; requires … |`).
- Update other text in `PLAN.md` that the plan changes (boundaries, unauthenticated routes, roles).
- If this plan amends an earlier plan, add a one-line "Amended by plan NN" note there. `AGENTS.md` requires the master and affected plans to change together.
- If the plan adds credentials, token types or cross-system boundaries, propose the matching `AGENTS.md` architecture-rule line to the user instead of editing it silently.

## 6. Check before handing over

Run this, and fix anything it prints:

```bash
d=docs/plans/NN-slug
for f in $d/[0-9][0-9]-*.md; do
  case $f in *00-README.md) continue;; esac
  for s in '^## Decisions you need' '^## Scope' '^## Tasks' '^## Done when' 'Model:' 'Prerequisites:'; do
    grep -q "$s" "$f" || echo "$f missing $s"
  done
done
grep -c '^- \[ \] [0-9]' $d/00-README.md   # status lines == number of slice files
```

Then check these yourself:
- every acceptance row is covered by some slice's Done when;
- every "Parallel with" pair has disjoint Files touched;
- no slice says "see plan X" for something it must implement.

Reply to the user in at most 12 lines:
- the plan path
- the slices, with models and the parallel tracks
- decisions worth a second look
- open questions

Say it's ready for `/run-plan NN` in a fresh session. Don't commit unless asked.
