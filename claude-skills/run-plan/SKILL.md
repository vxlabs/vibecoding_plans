---
name: run-plan
description: Orchestrate a numbered plan in docs/plans/NN-*/ — dispatch its slices to slice-worker agents (in parallel where safe), gate each with tools/verify-slice.sh, tick the README, then run acceptance with slice-checker. Use when the user says "/run-plan NN", "run plan NN" or "orchestrate plan NN".
argument-hint: <plan number> [first slice]
---

# Run plan $ARGUMENTS

You are the orchestrator. You decide sequencing and blockers yourself and report the decision in one line; don't ask the user to choose. Stop and report only when a slice fails its gate, an agent reports BLOCKED, or a step needs a human (credentials, live third-party accounts, manual UI walkthroughs).

**You never write slice code yourself, and you never tick a row without script output you have read.** If something unexpected happens, stop and report it instead of patching around it.

## 1. Load the plan (keep it lean)

- `ls docs/plans/ | grep '^<NN>-'` locates the plan folder. Read `00-README.md`: the status list, the "Dependencies and parallel tracks" table and the acceptance checklist.
- Don't read the slice files in full. `sed -n '1,3p;/^## Scope/,/^## Tasks/p'` on each unticked slice gives you its header line and Scope, which is enough to schedule.
- If a prerequisite slice from another plan is unticked, run it first (it's in the same format), then continue.
- Run `git status --short` and note uncommitted files: another session may own them. Don't dispatch a slice whose Scope overlaps them; wait, or message that session with SendMessage.

## 2. Schedule (rolling, not batches)

- A slice is **ready** when all its prerequisites are ticked (an early tick from §4 counts).
- Keep up to 3 slices in flight. Whenever a slot frees or a tick makes slices ready, dispatch them right away; never wait for the rest of a "batch".
- Two slices may run together when their "Files touched" don't overlap. The README's "Parallel with" column and "Shared files" note are authoritative: files declared shared there (lockfile, marked module blocks) don't count as overlap.
- Always run schema/migration slices alone.
- Workers don't regenerate the API client; you do (§4 step 5), so client regeneration is no reason to serialise. If an older slice file still says "regenerate the client", tell the worker in the dispatch note that you'll do it.
- Use the model in the slice table: Sonnet → `slice-worker` (default), Haiku → `slice-worker` with `model: haiku`. Opus → `slice-worker` with `model: opus`; Opus slices are often refactors across modules, so run them alone unless their scope is a disjoint new folder.
- Put all Agent calls of one dispatch in **one message**, each with `run_in_background: true`. Don't poll; you'll be notified when each finishes.

## 3. Dispatch prompt (keep it this short)

```
Slice: docs/plans/<plan>/<MM>-<name>.md
Running in parallel: <other slice ids and their scope globs, or "none">. Don't touch their files.
<only notes not already in the slice file, e.g. a Result fact from a previous slice that changes this one>
```

Everything else (rules, env, verify, Result format, reply format) is in the agent definition. Don't repeat it.

## 4. Gate each finished slice (immediately)

Gate a slice as soon as its agent replies, even while others are running:
1. STATUS isn't DONE → stop dispatching and report it to the user, along with what else is still running.
2. Run `git diff --stat` and check that the changed files match the slice's Scope. Ask about any extra file; revert nothing yourself.
3. Run `tools/verify-slice.sh <the slice's spec files>` yourself. Don't trust the agent's PASS.
   - Typecheck/lint errors only in files owned by a slice that is **still in flight** don't block: the slice passes if its own files and specs are clean.
   - A failure in a file outside every slice's scope: rerun that test alone. An open known issue (`list_known_issues`, `q` = the failing test or file) describing the failure is baseline; without MCP every failure counts.
   - A real failure in this slice that is small and inside its Scope (a missing spec, one failing test): dispatch a **fresh** `slice-worker` with `model: haiku` and a 3-line prompt naming the fix. Don't resume the original agent, which carries its whole context; resume with SendMessage only an agent that ran fewer than ~25 turns. Anything bigger → stop and report.
4. Tick the slice in `00-README.md`. Add "gated while NN/MM in flight" when step 3 relied on the in-flight exception, and a one-line note if it's partial.
5. If the slice changed controllers or DTOs, run `npx pnpm@9.15.9 generate:client` (one at a time, never while another regen runs) before dispatching any web slice that depends on it.
6. Dispatch whatever became ready (§2).

**Full gate.** Whenever nothing is in flight, run `tools/verify-slice.sh` once over every spec ticked since the last full gate. This catches cross-slice breaks that the in-flight exception let through; a failure here → stop and report, naming the slices involved.

## 5. Acceptance (last slice)

- Dispatch `slice-checker` with the acceptance slice file, the README checklist and an evidence directory under your scratchpad.
- **Read the evidence files yourself** before ticking:
  - cited test names must exist (the grep is in the file);
  - the commands must show the claimed effect;
  - rows needing real browsers, keyboards or third-party accounts stay MANUAL.
- Tick only PASS rows you confirmed. List MANUAL rows for the user as numbered steps.

## 6. Close out

- Record known issues through the Sentinel MCP tools:
  1. `list_known_issue_boards` finds the project.
  2. `create_known_issue` records each new open item (MANUAL rows, follow-ups, deferred work, new baseline failures) with kind, area, plan, severity, detail and source, the same categories as before.
  3. `resolve_known_issue` with a note naming the plan and slice marks an item this plan fixed.
  4. When the `sentinel` MCP server is unavailable, list the items under "Known issues to record" in the final report and do not claim they were recorded.
- Report to the user in at most 15 lines:
  - slices done
  - checks run, with their PASS/FAIL lines
  - what's left for a human
  - known-issue IDs added

  Don't commit unless the user asks.
