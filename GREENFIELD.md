# GREENFIELD — setting up a brand-new repo, in order

This is the first agent session's script for an empty repo. (For a repo that
already has code or history, use [RETROFIT.md](RETROFIT.md).) Work through the phases in order. Every
step ends with an **exit test**: a thing you can observe, not a thing you believe.
Don't start a phase until the previous phase's exit tests pass. Record each exit
test's evidence (command output, a link, a screenshot) in the bootstrap tracking
issue you create in step 1.3.

Read first: [README.md](README.md) → [PRINCIPLES.md](PRINCIPLES.md) → this file →
[starter/AGENTS.md](starter/AGENTS.md) (the rulebook template). Skim [LESSONS.md](LESSONS.md); come back to it
whenever something surprises you.

Levels used below: **[Invariant]** do it as written. **[Default]** do it this way
unless a decision record says otherwise. **[Later: trigger]** skip until the
trigger happens, and don't file its absence as a gap.

The owner's decisions are batched in [DECISIONS-FOR-OWNER.md](DECISIONS-FOR-OWNER.md).
Send them early and keep working: every one has a default that applies until the
owner answers.

---

## Phase 0 — Understand what we're building (no code yet)

### 0.1 Charter  [Invariant]
Copy `starter/docs/graph/templates/project-charter.yaml` to `docs/charter.yaml`
and fill it with the owner in one short interview (batch the questions; ten at
most). The fields that matter most: the primary user, the one core journey that
proves the product works end to end, non-goals, and what is explicitly
unsupported.

**Exit:** `docs/charter.yaml` has no `<placeholder>` in `project`, `authority` or
`quality`, and the owner has said yes to it in chat.

### 0.2 Upstream survey  [Invariant]
This product is built by bringing existing open-source projects together, so the
upstreams are the first architecture decision, not a detail. For each candidate:

- read its LICENSE at a specific tag or commit (not the README badge);
- check its health: last release, active maintainers, open-issue trend, bus factor;
- decide where it would run: in the browser, in a local process, or on a server;
- decide how it would come in: package, vendored copy, submodule, subprocess, or
  network service;
- note what it would cost to replace.

Record each chosen upstream in `upstreams.yaml` and each rejected one in a single
line of the decision record that chose among them.

**Exit:** every upstream the first milestone needs has an `upstreams.yaml` entry
with a license, a pinned tag or SHA, a mode, and a placement. Any license that
conflicts with the owner's license policy has a decision record.

### 0.3 Placement and layout decision  [Default]
Write decision record 0001: where each capability runs and how the repo is laid
out. A starting shape that fits "local app + web app over shared capabilities":

```
<core>/         pure domain logic and capability contracts; no UI, no platform APIs
<adapters>/     one per upstream; the only code allowed to import that upstream
<apps>/web/     the web application
<apps>/local/   the local application (desktop shell, CLI, or local companion service)
<tools>/        the capability contracts exposed to AI agents (e.g. as an MCP server)
fixtures/       known-answer inputs and outputs
docs/           charter, decisions, graph templates
```

The idea worth keeping even if the folders change: **one capability contract,
several front doors.** The same typed capability should serve the UI, the local
app, a CLI and an AI agent, so a capability is tested once and every surface
tells the truth about it.

**Exit:** decision 0001 accepted, including a one-line reason for the language and
framework choices and the simplest alternative that was rejected.

### 0.4 Spikes, if needed  [Default]
If an integration mode is uncertain (can this upstream run in the browser? does
its local API do what we need?), run a time-boxed spike on a throwaway branch.
A spike's output is a decision record, never merged code. Delete the branch or
promote a rewrite through a normal work item.

**Exit:** every uncertain placement in 0001 is either proven by a spike or
explicitly marked as a risk in the decision.

---

## Phase 1 — Lay the rails (before any feature)

### 1.1 Create the repo and copy the starter  [Invariant]
Create the repo (where it lives, and whether it is public, are owner decisions).
Add `graphwright/` to `.git/info/exclude` so the kit itself is never committed.
Record the Graphwright commit you bootstrapped from (`git -C graphwright rev-parse
HEAD`) in the first commit message and in `docs/charter.yaml`, so later you can
see which Graphwright changes this repo hasn't picked up. Copy everything in
`starter/` to the repo root. Fill the `AGENTS.md` slots you
can already fill from phases 0.1–0.3, and leave the owner-decision slots marked.
`AGENTS.md` is the one rulebook: Codex reads it natively and `CLAUDE.md` imports
it, so there is no second copy to keep in sync (see
[designs/agent-adapters.md](designs/agent-adapters.md)).

**Exit:** first commit on `main`; `git status` clean; `CLAUDE.md` imports `AGENTS.md`,
and a fresh Claude Code session and a fresh Codex session each answer "what are
this repo's truth rules?" correctly.

### 1.2 Repository settings  [Default]
Using `gh`:
- default branch `main`; delete head branches on merge; allow auto-merge (the
  setting only; nothing uses it yet); pick one merge method and disable the others;
- issues and Projects on, wiki off;
- labels from `.github/labels.yml`;
- secret scanning and push protection on; Dependabot security alerts on.

**Exit:** `gh repo view --json` shows the settings; `gh label list` matches
`labels.yml`.

### 1.3 The one gates command and the `checks` check  [Invariant]
Create the single gates command (typecheck, lint, unit tests, known-answer tests,
the import-boundary rule "the core imports no upstream and no UI", and a license
check over dependencies). It runs each gate by name and, on failure, says which
one failed. Wire it into `.github/workflows/checks.yml` (splitting the gates into
named steps there is fine; they stay one `checks` context, which owns one concern:
deterministic correctness). Then apply the ruleset:

```
gh api repos/<owner>/<repo>/rulesets --method POST --input .github/rulesets/main-protection.json
```

(`integration_id` 15368 pins the context to GitHub Actions so nothing else can post
a fake green. If your plan lacks rulesets on private repos, use classic branch
protection with the same required context.)

**Exit (the negative control):** open a PR that deliberately breaks one gate (a
type error, a failing known answer, a core file importing an upstream). The
`checks` context must go red **on that PR's exact head commit**, and the merge
button must be blocked. Screenshot or link it, then close the PR without merging.
A gate that has never been seen failing is not a gate.

Also create the bootstrap tracking issue now and post this evidence to it.

### 1.4 Secrets  [Invariant]
Create the project's secret-manager environment (the owner's standard is a
1Password Environment), list the variable names in `.env.tpl`, and run everything
that needs a secret through the manager.

If local development or CI needs placeholder values, keep them in a committed
`.env.example` with obviously fake values that are inert by construction (for
example hostnames under the reserved `.invalid` or `.test` domains, so a stray click
can't reach a real third party), and keep it in step with what CI injects.

**Exit:** the app's dev command runs through the secret manager with no `.env`
file on disk; `git grep` for key-shaped strings finds nothing.

### 1.5 Agent settings  [Default]
Add `.claude/settings.json` with deny rules for reading env files and secret
stores, and the equivalent for Codex (see
[designs/agent-adapters.md](designs/agent-adapters.md)). Keep the permission list
short; the permission prompts themselves tell you what to add.

**Exit:** Claude Code, asked to read `.env` in this repo, is refused by the
settings, not by its own judgment. (Codex has no equivalent file-level deny; on
that side the guarantee is step 1.4's: no file with real values exists on disk.)

### 1.6 Walking skeleton  [Invariant]
Build the thinnest end-to-end path through every layer: one core capability, one
adapter over one real upstream, surfaced in the web app and the local app. Test it
at both boundaries: a known-answer test on the core's output, and a browser test
that asserts the exact text a user reads on screen. (In the source project the
defect that reached users passed every core test; only the screen was wrong.)

**Exit:** seen working in a real browser and in the local app (screenshots in the
tracking issue), and `checks` is green on that commit.

### 1.7 The Project board  [Default]
One Project for the repo, with views:
- **Ready** — open, in a milestone, not blocked, not deferred, not in progress;
- **In progress**;
- **Blocked** — has an open "blocked by" link or `needs:decision`;
- **Known gaps** — label `known-gap`.
Fields: Status, Priority, Tier, Size. (`gh` needs the `project` scope:
`gh auth refresh -s project`.) This board is the only dashboard.

**Exit:** each view shows what its definition says, checked by opening it.

---

## Phase 2 — Plan, as GitHub objects

### 2.1 Milestones, parents, sub-issues, dependencies  [Invariant]
Turn the charter into the plan of record:
- milestones for outcomes, each with a one-line "done means";
- parent issues for features;
- sub-issues for the work units, written with the work-item form (objective,
  acceptance with named evidence, non-goals, where the change goes, tier);
- "blocked by" links for every real dependency, and no others.

Don't write a PLAN.md. If a narrative helps the owner, generate it from the issues
and say it's generated.

Sub-issues mean "part of"; "blocked by" means "must finish first". Keep them
separate. Split work by observable outcome ("a user can import X and see Y"), not
by layer ("build the storage"). If two items would each need half of a shared
contract, keep them together or extract the contract as its own item first.

Before creating a large plan, have the other AI vendor critique it (one pass, not a
loop). In the source project that caught stale assumptions about existing coverage
and four pieces of unnecessary new automation.

**Exit:** the Ready view shows only unblocked leaves; there is no dependency cycle;
every leaf's acceptance criteria name their evidence.

### 2.2 The derived graph index  [Later: roughly 30+ open issues, or the first time an agent picks the wrong next task]
A script that rebuilds the work graph (and later the evidence graph) as JSON from
GitHub and Git, for planning queries such as "what unblocks the most?". It is a
derived view: rebuilt, never edited. See
[designs/derived-graph-index.md](designs/derived-graph-index.md).

---

## Phase 3 — Run the loop once, then prove you can resume it

### 3.1 One work item through the full loop  [Invariant]
Pick one Ready leaf. Build it on its own branch and worktree, run the gates, get
a fresh-context review of the exact head commit (a second vendor if one is
available, otherwise a fresh session that didn't write the code), repair within
the tier's budget, and bring the owner the merge. After merge, verify on `main`.

**Exit:** the PR shows the head SHA, the gates result, the review record, any
`known-gap` issues, and (for user-visible work) a screenshot.

### 3.2 The interruption test  [Invariant]
Write the pass criteria first. Then stop a session in the middle of a work item and
start a fresh agent with only "resume issue #N". It should find the branch, the
state of the work, and the next safe action from GitHub and Git alone.

**Exit:** the fresh agent meets the pre-written criteria. If it can't, fix what was
missing (usually: the issue didn't say what was done, or state lived only in chat)
before building more.

### 3.3 First lessons and handoff  [Default]
Write the first `.claude/lessons/` entries from anything that surprised you, and a
session handoff in the owner's shared memory store.

**Exit:** a fresh session can read the handoff and the Ready view and pick the right
next task.

---

## Phase 4 — Add on trigger, not before

Each of these was valuable in the source project **once its trigger arrived**, and
expensive when built early. When a trigger fires, add the smallest version, prove
it with a negative control, then make it required.

| Add | Trigger | Notes |
|---|---|---|
| `browser` as a required check | The first user journey exists and its suite has been green without retries for a week | Run specs against a built app, not a dev server; a test that can't fail is worse than none |
| `independent-review` check | A self-hosted runner holds a second vendor's CLI, signed in on the owner's subscription | Design: [designs/independent-review.md](designs/independent-review.md) |
| Production promote workflow | The first real deploy target exists | Owner-triggered; records the exact SHA; smoke-test after. Design: [designs/release-and-promote.md](designs/release-and-promote.md) |
| Integration branches + batch PRs | Several agent PRs need to ship together, or `main` churns faster than the owner can follow | Same required checks on `integration/**` |
| Auto-merge | Required checks, including independent review, have caught a planted defect, and the owner says yes | Only below the critical tier, and only into an integration branch at first |
| Release manifest | A release involves more than one repo, or pinned upstream services | Pins every repo and upstream revision; copied from `upstreams.yaml` |
| AI evaluation set | The first AI-generated feature | Known-answer cases with thresholds; runs on demand or nightly, not as a required check |
| Merge queue | PRs routinely go stale against `main` while waiting | Check the plan: GitHub's queue isn't on every plan; Mergify has a free tier |
| Self-hosted runners | Hosted Actions minutes run out, or a job needs the owner's subscription sign-ins or special hardware | Split runners by job cost (`heavy` / `light`) so builds never starve fast gates; read `runs-on` from a repository variable with a hosted fallback; clean up root-owned files after container jobs on persistent runners |
| A second repo or a shared toolkit | A second product actually needs the same code | Until then, one repo |
