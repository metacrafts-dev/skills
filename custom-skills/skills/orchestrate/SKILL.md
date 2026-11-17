---
name: orchestrate
description: Rules of execution — plan in the main session, run everything else through the delegate fleet at maximum safe parallelism, orchestrate host resources, clean up. Invoked by hand.
disable-model-invocation: true
argument-hint: "[load=N] [quota=wait|rotate] — load: max % of box resources (default 50); quota: on a 5h quota drain, wait for the reset (default) or rotate work to the swarm lane"
---

# Orchestrate

## Verdict

**The main session knows, thinks, decides, and orchestrates — the fleet performs.**
It orchestrates implementation, review, and hardening; it never performs them. No typing code, no running tests, builds, or gate commands, no digesting raw code or data in this session — a job, however small, belongs to a worker. Context here is the scarcest resource: a minute spent performing what a worker could perform is waste. Fill every lane before doing work inline.

**The main session delegates the work — including the heavy actions of reviewing and hardening.** Relays (`/opencode-delegate` and its kin) carry all implementation; sub-agents/delegates carry every helper job — pre-calculations, summaries, reports, gate and test runs, and consuming code or data to return it in a required shape (flash and swarm lanes); everything runs in parallel, every lane filled. **Planning and orchestration both target the maximum concurrent delegates available: every lane at cap, refilled the moment a worker reports — with the host-load gates (admission, heavy slots, envelope) as the only ceiling, never caution. The gates watch the host's CPU and RAM against the load threshold and queue every heavy build/test/run through its slot to hold the box just under it — code dispatch never queues behind them.** The main session reads only the final report the delegates hand back. Its context is a precious and scarce resource, preserved maximally by delegating any work that would fill it — and the pricier the model, the stricter this holds (the priciest seat — reviewer or consultant — reads reports, never raw input).

**Consulting a smarter model is a decision point, not a hand-off.** The session consults the decision-responsible model — the registry's decision maker if reachable in this harness, else the smartest model actually available — only at genuine decision points, over whatever channel the harness offers: a native bg-agent running that model, or a delegate relay. None smarter than this session reachable → the decision stays here. Carefully: batch the questions, send a brief (findings, options, recommendation) rather than raw context, then proceed on the answer.

## Fleet

Lanes are fixed; who fills them is not. The model registry — `<repo>/.agents/orch-models.md` if present, else `models.md` beside this skill — names per lane the model, cap, quota and drain drill. Read it once at invocation and again on any quota event; edit it freely (models change monthly — it is the human's call, re-point via `/delegate-setup`).

| Lane | Work |
|------|------|
| smart | decomposition, research, subtle logic, hard detail, writing worker briefs |
| flash | implementation where reliability matters, mechanical sweeps, bulk work — the default code lane |
| swarm | reviews lane work, preps briefs, second opinions, cleanup, light jobs |
| reviewer and decision maker | heavy review passes and genuine decision points that would burn main-session context |

Caps are per lane. Aim for every lane at cap — 10+ concurrent workers is normal; the load ceiling and the capacity gate (below), not caution, set the limit. An idle lane is waste.

## Loop

1. **Plan here.** This session holds the contract and knows the deliverable: decompose into independent tasks, each with a done criterion, splitting any task too big for one worker across several. One smart worker may draft the breakdown; this session approves it. A plan with fewer parcels than the active lanes' concurrency caps is a planning bug: issue the implementation contract, then carve it into at least cap-many partial parcels per lane, and orchestrate between them.
2. **Fill the lanes.** Dispatch until every lane is at cap; the moment a worker reports, split what remains and refill its lane. Tasks running short is a split signal, never an idle excuse — carve smaller parcels and keep every model at its concurrency cap. A real sequential dependency is the only reason a lane sits idle.
3. **Brief every worker** per opencode-delegate (goal, constraints, real gate commands, report contract, no commits) **plus a cleanup clause**: the worker deletes its own test/debug artifacts — build output, test containers, coverage, logs, temp fixtures — before reporting. Scoped questions and the SESSION CONTEXT block come from context-guard.
4. **Stay on every relay — and keep it visible.** No fire-and-forget: collect on completion notification, read `result.json`, never transcripts. Every running delegate, task, and background job is always represented in the harness visually — a named bg-agent/teammate or visible background task, never a hidden process — so the human can see at a glance what is running. A placeholder for the human's eye consumes no tokens, and if the harness demands a model, the weakest one at least effort.
5. **Review in stages — orchestrate it, never perform it.** Verification workers re-run the gates; review workers read the diffs; this session dispatches them and reads what comes back. When many reports land at once — a review swarm — one swarm agent collates them into a single report before this session reads anything. A worker whose last delivery was noisy (many corrections) must have a swarm agent review and fix its next delivery before it reaches this session. Heavy reviews go to the reviewer lane. **Final judgment and responsibility stay in this session — always; the work of reviewing never does.**
6. **Report every round.** Each round with the human ends with the whole picture, composed from worker and cluster reports, never re-derived from raw output: **progress** (parcels done / in-flight / remaining), **state** (per lane and per cluster: healthy, blocked, drained), **estimated steps left**, and **estimated time to finish the assignment as a whole**. Estimates are stated as estimates and revised each round — silence on any of the four is a bug.

## Cluster orchestrators

A task too big to track worker-by-worker from here — many parcels across lanes, progress-tracking that would bloat this session — goes to a cluster orchestrator instead of direct dispatch: one smart-lane agent given the task contract and a budget of workers, counted against the lane caps and the same gates. It splits the task, dispatches, and supervises its own sub-agents, delegates and relays; it answers only to this session. It plans and dispatches the same way this session does: its whole worker budget spent at maximum concurrency — every slot filled and refilled, the same host-load gates enforced, never a worker idle while independent work waits. This session should spawn one whenever a task is that big — managing its workers directly from here is context waste.

Its report contract — every report, in this order:

1. **State** — each worker it manages: done, in-flight, or blocked, one line each.
2. **Decisions** — every decision it took, one-line why each.
3. **Progress and estimate** — steps done / remaining for its task as a whole, and estimated time to finish.
4. **Results** — deliverables, gate outcomes, diffs ready for review, and cleanup confirmed: its workers left no test/debug artifacts behind.
5. **Deviations** — anything that diverged from the contract it was given.
6. **Ask** — present only when it stops: the exact decision this session must make, with options and its recommendation.

It reports when its whole task is complete, and the moment it hits something needing this session's attention or a decision — nothing between: no micro-steps, no sub-task chatter, no reasoning process. Yet every report is a complete image: after reading one, this session can answer the human's round report (progress, state, steps, time to finish) without asking anything further. Delegates it manages report to it the same way — estimates included — so the numbers roll up.

This session supervises a cluster orchestrator like any relay: no fire-and-forget, notice a dead or drained one, collect on notification. It never sits on a stop, and never grows scope to work around one. Its nested workers surface to the human through its own visible harness entry and the State roster of every report — the visibility rule holds inside clusters too; nothing runs unnamed or unseen.

## Resources

`/orchestrate load=N` — N is the ceiling: the maximum share of the box's resources, as a percentage, the fleet may consume. Omitted → `load=50`. N sets the appetite; the gates are the floor and hold at every N: at low N keep fewer lanes hot and dispatch in smaller waves; at high N fill every lane to cap.

This skill names gates, not paths. The active repo/session binds them — declare the binding commands in the repo's AGENTS.md or the host config; where nothing is declared, the session improvises equivalents with these semantics before any heavy work, degrading to one heavy lane at a time:

- **admission** — launch a worker only while the check prints OK (memory floor). If it refuses: park that dispatch — never run the work inline as a workaround — fill another lane and re-check before the next launch.
- **heavy** — host-wide, few slots (2 by default): every build/test command queues through it, low priority, holding its slot until memory and load clear.
- **serial queue** — a full gate run goes one at a time.
- **envelope** — all workers share one CPU/memory-capped slice.

Code in parallel is unlimited; tests and builds are what saturate a box. They queue through the gates — a worker blocked on a full test gate takes more code work instead of idling, and code lanes never wait on tests. After any test/artifact phase, spawn one swarm cleaner for safe quick wins: stale test containers, build output, coverage, logs, temp fixtures.

## Red flags — stop and re-read this skill

- Typing code, or running a test, build, or gate command, in this session — that is a job; hand it to a worker.
- The main session — or any reviewer lane — reading raw code or data that a cheaper delegate could have digested into a brief.
- Any lane idle while independent tasks wait.
- Bulk work on the smart lane while the flash or swarm lane has room.
- Accepting a delivery without an independent worker having re-run the gates and reviewed the diff.
- A running delegate or task with no visible harness representation.
- A round report missing any of: progress, state, steps left, time to finish.
- A cluster orchestrator narrating micro-steps — or gone silent past its check-in.
- Test artifacts left behind by a finished worker or test phase.
