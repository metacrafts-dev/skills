# Model registry

Who fills the orchestrate lanes — as of 2026-11. Volatile by design — models, caps, quotas and
drain drills change; edit this file, never the skill. Loaded once at `/orchestrate`
invocation and re-read on any quota event. A repo-local `orch-models.md` at the repo root
replaces this file wholesale for that repo.

## What models, what role

| Lane | Model | Cap | Best at | Never give it |
|---|---|---|---|---|
| smart | `zai/glm-5.3` · `/opencode-delegate` | 4 | decomposition, research, subtle logic, hard detail, worker briefs | bulk typing |
| flash | `zai/glm-5.3-flash` · `/opencode-delegate` | 7 | implementation where reliability matters, mechanical sweeps, bulk work — the default code lane | subtle decomposition |
| swarm | Gemini 3.8 Flash · `/agy-delegate` | admission-bound — dispatch until the admission gate refuses | reviewing lane work, prepping briefs, second opinions, cleanup, light jobs | smart-lane work — the weakest model |
| reviewer and decision maker | `opus` · harness-dependent channel: bg-agent or delegate relay | few | heavy review passes, genuine decision points | raw code or data — briefs in, reports out |

Cap is each model's **learned concurrency limit** — the saturation target during implementation, not a ceiling to sit under. These are stress-tested values: re-test from time to time and edit here as quotas, accounts and the box change.

## Limits and how to manage them

- Drain signature: a run ends `failed` with no error message = out of quota, not broken code. Check before re-dispatching.
- zai account — shared by smart + flash, one 5h window; four concurrent lanes can drain it by halfway, so spend it on hard work. On a drain the `quota` switch governs. `wait` (default) parks the work till reset, then continues the same tasks on the same lanes — partial work survives in worktrees. `rotate` re-routes only Flash-sized remainders to the swarm lane; smart-lane work holds for the reset.
- agy — one 5h window per account; accounts are named `1` and `2`. Switch on drain: `accounts use agy 1` (or `2`). Both drained → the swarm lane waits for the first reset.
- opus — the limit is price and context, not quota. Every contact is a brief-in/report-out decision point; see the skill's Verdict.
