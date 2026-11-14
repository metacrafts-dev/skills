---
name: orchestrate
description: Rules of execution — plan in the main session, run everything else through the delegate fleet at maximum safe parallelism, orchestrate host resources, clean up. Invoked by hand.
disable-model-invocation: true
argument-hint: "[load=N] [quota=wait|rotate] — load: max % of box resources (default 50); quota: on a 5h quota drain, wait for the reset (default) or rotate work to the swarm lane"
---

# Orchestrate

## Verdict

**The main session thinks, decides, and reviews — the fleet types.**
Context here is the scarcest resource: a minute spent typing what a worker could type is waste. Fill every lane before doing work inline.

**The main session delegates the work — including the heavy actions of reviewing.** Pre-calculations, summaries, reports, tests, and consuming code or data to return it in a required shape all go to cheaper models in the fleet (flash and swarm lanes); the main session reads only the final report the delegates hand back. Its context is a precious and scarce resource, preserved maximally by delegating any work that would fill it — and the pricier the model, the stricter this holds (the priciest seat — reviewer or consultant — reads reports, never raw input).

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
4. **Stay on every relay.** No fire-and-forget: collect on completion notification, read `result.json`, never transcripts. Every delegate runs as a visible teammate/bg-agent in this session — a placeholder for the human's eye, never a hidden process: it consumes no tokens, and if the harness demands a model, the weakest one at least effort.
5. **Review in stages.** Re-run gates, read diffs. When many reports land at once — a review swarm — one swarm agent collates them into a single report before this session reads anything. A worker whose last delivery was noisy (many corrections) must have a swarm agent review and fix its next delivery before it reaches this session. Heavy reviews go to the reviewer lane. **Final sanity review and responsibility stay in this session — always.**

## Resources

`/orchestrate load=N` — N is the ceiling: the maximum share of the box's resources, as a percentage, the fleet may consume. Omitted → `load=50`. N sets the appetite; the gates are the floor and hold at every N: at low N keep fewer lanes hot and dispatch in smaller waves; at high N fill every lane to cap.

This skill names gates, not paths. The active repo/session binds them — declare the binding commands in the repo's AGENTS.md or the host config; where nothing is declared, the session improvises equivalents with these semantics before any heavy work, degrading to one heavy lane at a time:

- **admission** — launch a worker only while the check prints OK (memory floor). If it refuses: park that dispatch — never run the work inline as a workaround — fill another lane and re-check before the next launch.
- **heavy** — host-wide, few slots (2 by default): every build/test command queues through it, low priority, holding its slot until memory and load clear.
- **serial queue** — a full gate run goes one at a time.
- **envelope** — all workers share one CPU/memory-capped slice.

Code in parallel is unlimited; tests and builds are what saturate a box. They queue through the gates — a worker blocked on a full test gate takes more code work instead of idling, and code lanes never wait on tests. After any test/artifact phase, spawn one swarm cleaner for safe quick wins: stale test containers, build output, coverage, logs, temp fixtures.

## Red flags — stop and re-read this skill

- Typing code a fleet worker could have typed.
- The main session — or any reviewer lane — reading raw code or data that a cheaper delegate could have digested into a brief.
- Any lane idle while independent tasks wait.
- Bulk work on the smart lane while the flash or swarm lane has room.
- Accepting a self-report without re-running gates and reading the diff here.
- Test artifacts left behind by a finished worker or test phase.
