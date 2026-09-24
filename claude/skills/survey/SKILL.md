---
name: survey
description: Read-only reconnaissance of a repo - go look and bring back the map. Use when the human wants to know what's here, what's drifted, what's latent, or what's owed, without anything being changed. Also the right first move on an inherited or unfamiliar codebase. Keywords - survey, map this, what's here, look around, whole-scope map, reconcile, drift check, inherited codebase, before we touch anything.
---

# survey — go look, bring back the map

Read-only, `mapping` mode, any repo, any point in its life. `discuss` steps out to think;
**survey steps out to look.** The deliverable is the honest whole-scope map — the lead's job made
a word — never a fix to whatever was noticed along the way.

## The four questions

1. **What's here** — structure and what actually works, derived from code and running behavior,
   never from docs claiming it. Right-size the depth to the repo; this is a map, not an audit.
2. **What's drifted** — code vs STATE doc vs registers. Report the drift; don't repair it here.
   (This is the reconciler role, triggered.)
3. **What's latent** — capabilities that exist but nothing surfaces, inventoried with costs
   attached, unsorted by my opinion of what matters.
4. **What's owed** — unverified claims, assumptions that aged unconfirmed, open loops going stale.

## Rules

- **Strictly no edits.** No files changed, no queue items filed, nothing "quickly fixed" on the way
  through. A survey that patches has become a fixing session wearing the wrong name.
- **Findings carry their grain** — but they land in the report, not the registers. On exit, same
  move as `discuss`: **"promote anything?"** The human routes findings to the anchor, the slices
  register, the backlog, or the sky. Silence promotes nothing.
- **Findings over verdicts.** `n=` on anything counted; "deviation, no baseline" where there is no
  comparison class; the shape of a problem, never its magnitude on feel.
- Delegate depth where it earns it (read-only subagents for big trees); the map is still one report.

## Report shape

The investigation may be broad; the first human reply is not. It must fit on one screen and be easy to
understand when read aloud:

1. `Survey complete. I changed nothing.`
2. **Fix first** — the most consequential verified finding, in one or two short sentences.
3. **Decide next** — the one genuine owner decision, only when one exists.
4. **Later** — the most important non-urgent drift or debt.
5. One recommendation, then `Promote that into the project plan?`

Use at most **three findings total** in the first reply. Mark each **Verified**, **Likely**, or **Not tested**
when its certainty is not obvious. Use plain words, short lines, whitespace and concrete consequences. Do not
repeat the four-question framework as four long sections merely because the investigation used it.

Evidence stays behind the summary. Give at most one useful `file:line` pointer per finding; provide the full
map only when the human asks for detail. Never expose raw commands, shell errors, worker/subagent handbacks,
rule IDs, counts with no consequence, or duplicated summaries. A worker report is input to the lead's
synthesis, not another report pasted before it.

Security or data-loss findings outrank adapter hygiene and documentation drift. Describe only what the
evidence proves: an exposed path is not a successfully reproduced exploit; callable is not verified working;
missing provisioning is a fresh-install risk until a clean install proves failure.

## The exit — name what the map makes possible

One line, after the promotion question: the next move the map itself points at — `groundwork` if there is
no anchor, `spinup` if the adapter isn't wired, a team shape if the scopes are separable enough to own
concurrently. A capability nobody is told about does not exist; every code word ends by naming the
next one that just became available.
