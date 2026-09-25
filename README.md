# Graphwright

*A wright is a maker: a shipwright builds ships, a playwright builds plays. A
graphwright builds with graphs.*

Graphwright is a way of running a software project that AI coding agents build:
**graph engineering** (four graphs, one authority per fact, everything else
derived), **GitHub as the plan of record**, **evidence tied to the exact commit**,
and a **lean set of checks** that each earn their place. This repo is the kit that
sets it up in a brand-new repo from day one: the rules, the GitHub structure, the
checks and the planning model. It deliberately doesn't choose your stack or your
architecture.

Built 2026-09-25 from the owner's *Graph Engineering Field Guide* and the lessons of
a mature AI-agent-built product repo (its audits, its reorganization, and its
agents' memory), with none of that product's baggage.

## Two paths

| Starting from | Follow | In short |
|---|---|---|
| **An empty or brand-new repo** | [GREENFIELD.md](GREENFIELD.md) | Charter → rails → plan as GitHub issues → one full loop → resume test |
| **An existing repo** (code, history, users, its own habits) | [RETROFIT.md](RETROFIT.md) | See everything, change nothing → one authority per fact → add beside the old → prove → switch → remove |

Everything else in this repo (principles, lessons, designs, the starter files) is
shared by both paths.

## For the owner: three things to do

1. **Open the repo** (create it first, if it's new) and start an agent session in
   it (Claude Code, using your strongest model).
2. **Point it at Graphwright** by saying one of:
   > Clone `StatiqSolver/graphwright` into a `graphwright/` folder inside this repo,
   > read `graphwright/README.md`, and follow `GREENFIELD.md`.

   > Clone `StatiqSolver/graphwright` into a `graphwright/` folder inside this repo,
   > read `graphwright/README.md`, and follow `RETROFIT.md`.

   The agent keeps that folder out of the new repo's Git history (it adds it to
   `.git/info/exclude`), records which Graphwright version it used, and deletes the
   folder once bootstrap is finished; the files that matter will have been copied
   into the repo by then.
3. **Answer the batch of decisions** it brings you (listed in
   [DECISIONS-FOR-OWNER.md](DECISIONS-FOR-OWNER.md)). Each has a sensible default,
   so the agent keeps working while you think.

After that, you work the way you already do: describe what you want, answer
product questions, merge, and approve releases.

## For the agent: read in this order

1. **This file**, all of it.
2. [PRINCIPLES.md](PRINCIPLES.md) — the model (four graphs, one authority per fact,
   done-is-a-vector, bounded loops, lean delivery) and why.
3. [GREENFIELD.md](GREENFIELD.md) for a new repo, or [RETROFIT.md](RETROFIT.md)
   for an existing one — the ordered setup with an exit test per step. This is your
   script.
4. [starter/AGENTS.md](starter/AGENTS.md) — the rulebook template that becomes the
   repo's `AGENTS.md` (in a retrofit, merged with the rules the repo already has).
5. [LESSONS.md](LESSONS.md) — skim now; return whenever something surprises you.
6. [designs/](designs/) — read each one when your path points you at it.

## What's in the kit

| Path | What it is | Goes into the new repo? |
|---|---|---|
| `README.md`, `GREENFIELD.md`, `RETROFIT.md`, `PRINCIPLES.md`, `LESSONS.md`, `DECISIONS-FOR-OWNER.md` | The handoff itself | No; the kit folder stays uncommitted and is deleted after bootstrap (copy any lesson worth keeping into `.claude/lessons/` first) |
| `starter/` | Files for the repo's root (copied in a greenfield; merged in a retrofit): `AGENTS.md` (the rulebook), `CLAUDE.md`, `upstreams.yaml`, `NOTICES.md`, `.gitattributes`, `.env.tpl`, `ARCHIVE.md`, `.github/` (checks workflow, ruleset, issue forms, PR template, labels), `.claude/`, `docs/` (graph templates, decisions) | Yes: the first commit (greenfield) or stage A (retrofit) |
| `starter/docs/graph/templates/` | The Field Guide's nine templates, verbatim: work packet, context packet, run receipt, evaluator rubric, decision record, escalation record, bounded run, pilot plan, project charter | Yes |
| `designs/` | How to build things **when their trigger arrives**: independent review, the derived graph index, building on upstreams, agent settings and worktrees, release and promote | Only as reference |
| `reference/` | The full *Graph Engineering Field Guide* (open the HTML in a browser) and its source register | Optional; it's the primary source |

## How prescriptive this is

Every rule carries one of three levels:

- **[Invariant]** — never relaxed. Verify state before acting; skip is never pass;
  shipped means seen working; the owner authorizes anything outward-facing; one
  authority per fact; secrets never touch disk.
- **[Default]** — the starting choice. Change it with a decision record in
  `docs/decisions/` when it stops fitting: branch model, check names, review tiers,
  folder layout, label set.
- **[Later: trigger]** — add only when the named trigger happens. Until then its
  absence is not a gap, and nobody should file it as one.

If a default fights the project, change it on the record. If an invariant fights
the project, ask the owner.

## Tried and retired in the source project: don't rebuild these

Each of these was built, run for real, and retired or frozen because a built-in or
a simpler rule did the job better. They're listed so they aren't rediscovered as
bright ideas:

- **A ledger file of unfinished work** (with id locks and a collision checker).
  It grew faster than it closed, most of its rows never reached the tracker the
  planner read, and its gate failures were almost all paperwork. Replaced by
  `known-gap` labelled issues.
- **Custom "receipt" files to close issues.** Native "Closes #N" closed far more
  issues than the custom closer did. Use the native one.
- **A self-built wave/swarm factory** (lanes, a merge slot, a check farm, a custom
  scheduler). Replaced by GitHub's dependencies and search, native worktrees,
  self-hosted Actions runners, and required checks.
- **Paperwork gates** (manifests, conformance cards, "a test file changed" rules).
  Their real checks folded into one gates command; the rest went.
- **A paid-API review lane.** Reviews now run on the owner's subscriptions only.
- **A third-party orchestrator** and **local LLMs**. Both abandoned; don't propose
  them again without a new reason.

## Keeping Graphwright current

Graphwright improves as projects use it. When a lesson learned in a project turns
out to be general (it would have helped on day one anywhere), propose it back here
as a PR, with the incident that taught it. The owner merges it. Projects don't
auto-sync from Graphwright; each records the version it started from and pulls in
later changes deliberately, one decision at a time.

## Hygiene

This kit contains no secrets, no credentials, and no private data from the source
project. If you add any while bootstrapping, it goes in the secret manager, not here.
