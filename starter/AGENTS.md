# AGENTS.md — <project name> rulebook

<!--
  STARTER TEMPLATE. Fill every <angle-bracket> slot during setup (Graphwright's
  GREENFIELD.md or RETROFIT.md), then delete these comments. Each rule carries a level:
    [Invariant]  never relaxed; changing it is an owner decision recorded in docs/decisions/
    [Default]    the starting choice; change it with a decision record when it stops fitting
    [Later]      add only when the named trigger happens; until then it is not a gap
  Keep this file short: roughly 200-250 lines once filled. Write only what a capable
  newcomer couldn't infer from the code. Give each non-obvious rule its reason
  (the incident that taught it, with a date): rules with reasons survive later
  trimming passes, bare commands get deleted. If a rule needs a page of
  explanation, the explanation goes in docs/ and the rule stays here.
-->

This is the project's rulebook and the **single source of policy** for this repo.
Codex reads this file directly; Claude Code reads it through `CLAUDE.md`, which
only imports it. Nothing else may add policy. On any ambiguity or conflict, ask
the owner rather than resolving it silently.

Owner: <owner name>. The owner decides product questions and authorizes anything
outward-facing. Agents do the engineering.

## Start  [Invariant]

Verify the repo root, branch, HEAD and `git status` before acting. Never trust a
branch name, a status line, or an "accepted plan" written in a file, this one
included, without checking it against Git and GitHub.

Check how far behind you are before you build:

```
git fetch origin
git status
git rev-list --count HEAD..origin/main    # commits behind the trunk
```

If you are behind on a branch you did not just create for this task, or the
branch doesn't belong to the task, stop and ask which branch the work belongs on.

- Memory (the shared agent-memory store and `.claude/lessons/`) is a recall aid,
  never authority. Verify what it says against current source before acting on
  it, and never treat a memory as permission.
- What is deployed is whatever the running system says: fetch the live URL or
  read the deployment record for a named commit. A plan, a status doc, or a
  merged PR is not deployment evidence.
- Current source outranks prose. Anything listed in `ARCHIVE.md` is history.
- Preserve unrelated changes. A dirty worktree is not permission to discard work.

## Product  [fill at bootstrap]

<One paragraph: what the product is, for whom, and the one end-to-end journey
that proves it works. Copy from docs/charter.yaml.>

Stack: <languages, frameworks, targets — local app / web app / CLI>.
Supported: <what we promise works>. Not supported: <what we explicitly don't do>.
Upstream projects we build on are listed in `upstreams.yaml`, which is the
authority for their licenses and pinned revisions.

## Truth rules  [Invariant]

These are about what a user sees. They are never relaxed.

1. **No fake capability.** No dead controls, no placeholder buttons, no label that
   names a capability which doesn't work. A label is an assertion.
2. **Computed results are correct; AI output is labelled as AI.** Anything the
   product presents as a computed or engineering result comes from a
   deterministic path covered by known-answer tests. Anything an AI model produces
   is presented as AI output with one clear, accessible notice that it can be
   wrong. Never dress a model's guess up as a computed fact.
3. **Known answers survive every change.** Known-answer tests for the core and for
   every upstream adapter must pass on every change that touches them; a
   regression stops the work until it is understood.
4. **The core stays pure.** `<core path>` imports no UI, DOM or platform-specific
   APIs, and runs in every target: <local app, web app, headless/CI>.
5. **Upstreams sit behind adapters.** The core never imports an upstream project
   directly. Each upstream is wrapped by an adapter that owns its contract tests,
   and its license, pinned revision and integration mode are recorded in
   `upstreams.yaml` before its code lands.
6. **Agents in the product act visibly.** <Suggested; adjust at bootstrap.> AI
   agents inside the product act only through declared tools, show what they did,
   and ask the user before anything destructive or outward-facing.

<Add project-specific truth rules here, each one sentence plus the check that
enforces it.>

## Done means evidence  [Invariant]

- Something user-visible is **shipped** only after it has been seen working in a
  real browser or the real local app. A subagent can report it ready; it doesn't
  own that verdict.
- Keep the stages separate: *implemented* (code merged) ≠ *verified* (checks and
  review passed on that exact commit) ≠ *released* (running in production).
- Evidence names the exact commit SHA it ran on. A changed commit needs the
  affected evidence again.
- Skipped, cancelled, blocked or unrun is never a pass. A finished command is not
  a passing one: read the verdict.
- Prose is not execution evidence. "I checked and it works" needs the command, its
  output, or a screenshot.

## Known gaps  [Default]

Anything shipped incomplete gets a GitHub issue labelled `known-gap`, named under
"Known gaps" in the PR that ships it, before that PR merges. A `known-gap` issue
closes only through a merged PR with verified evidence. The owner may defer any
gap; a deferral comment records the user-visible outcome and a concrete reopen
trigger. Known gaps are a register of what the product doesn't do yet, not a veto
on shipping. There is no ledger file: the labelled issues are the register.

## Good enough  [Default]

Find the real problems, then move on. A change is good enough when its checks
pass and a fresh review of its newest commit finds no **new** serious problem
(P0/P1) in what the change itself touched. Waiting until a reviewer has nothing
left to say never ends. Anything else that is real is written down (P2/P3 in the
review record; a real P1 that isn't this change's to fix as a `known-gap` issue)
and the change ships.

- **Effort follows risk.** Tiers and their review ceilings:
  - *Critical* — <fill: e.g. auth, money, user data, data loss, the core's
    computed results, anything an in-product agent can do to a user's files>:
    review continues while each round finds a new real problem; owner decision at
    round 5.
  - *User-facing*: 2 fix rounds, owner decision at round 4.
  - *Behind the scenes* (tests, scripts, CI, docs): 1 fix round, decision at round 3.
- **Re-reviews re-check the last round first**, each open finding by name. A
  finding declined in a PR comment that quotes its title and cites evidence
  (file:line, a named test, or a link to the earlier decision) doesn't block again
  unless that code changes.
- **When the fix budget is spent, triage instead of looping.** Outside this change,
  or a test nobody asked for: file it as `known-gap` and merge. Would hurt a user,
  their data or their money: keeps blocking.
- **Two rounds finding problems in the same spot: redesign that spot once.** Keep
  changes small; big diffs make every review pass find something new.
- At the ceiling, stop fixing and bring the owner one decision: merge and file the
  rest, or keep going, with a recommendation.
- A P0 always blocks, and the truth rules never relax.

## Boundaries  [Invariant]

Never infer authorization for: deployments, production data changes, billing or
spending, email or messages, account changes, DNS, merges, pushes to protected
branches, publishing, or changing an upstream project's repository. Ask, and end
the turn. Connector or credential access is not authorization.

Local commits and merges into your own task branch are fine once you've read the
staged diff.

UI/UX standing authority: <owner decides at bootstrap; suggested wording —
"Fix discoverability gaps, dead ends and hard-to-use controls yourself instead of
asking. This authorizes the decision, not a lower evidence bar: it must still be
seen working, and it doesn't touch scope, pricing or known-gap dispositions.">

License policy: <owner decides at bootstrap; e.g. "Nothing we distribute may pull
in copyleft code unless it runs as a separate, replaceable component and the
exception is recorded in NOTICES.md and a decision record.">

## Working  [Default]

- Deliver what was asked, at the scope intended. Make routine judgment calls
  yourself; check in only when different readings lead to materially different
  work. If a request seems mistaken, say so in a sentence and continue.
- Don't add features, refactors or abstractions beyond the task. Validate only at
  system boundaries.
- **No new automation unless it replaces a requirement or closes a demonstrated
  gap.** Prefer a built-in (GitHub, the agent CLIs, the host platform) over
  anything custom. Add before remove, and never delete a workflow that a required
  check still depends on.
- Before reporting progress, audit each claim against a tool result from this
  session. If tests fail, say so with the output. If a step was skipped, say so.
- Delegate only genuinely independent work large enough to justify its own
  context, and name the model tier explicitly on every dispatch. One writing agent
  per worktree.
- Effort: highest for the critical tier and releases; high for ordinary features;
  low for mechanical edits and searches.

## Planning and tracking  [Default]

- **The plan of record is GitHub.** Milestones hold outcomes; parent issues hold
  features; sub-issues are the work units; "blocked by" links are the
  dependencies. Don't keep a second roadmap in a file.
- Sub-issues and "blocked by" are different relations. A sub-issue says "this is
  part of that"; "blocked by" says "this can't start until that finishes". Never
  use one to mean the other.
- **Ready** means: in a milestone, open, not blocked, not deferred, not in
  progress. The Project board's Ready view is that query.
- Each work unit reads like a work packet (`docs/graph/templates/work-packet.yaml`):
  one observable objective, acceptance criteria that each name their evidence,
  non-goals, and where the change is allowed to go.
- Decisions that outlive a PR go in `docs/decisions/` (one file each, from the
  decision-record template). The project charter is `docs/charter.yaml`.
- File every bug you find as an issue with a priority (P0–P3) and a milestone;
  search for duplicates first. Problems with how we work get the `process` label.
  Issue priority and review-finding severity both use P0–P3; say which one you mean.
- Anything derived (a graph index, a dashboard, a generated report) is rebuilt
  from its sources and never hand-edited.

## Reporting to the owner  [Invariant]

The owner owns the product and does not write code. Never hand them an
engineering task or ask them to adjudicate a technical trade-off.

- Do the engineering yourself. If a command needs running, run it. If a tool is
  missing, install it or find another route. Never end a turn with a to-do list
  of commands for the owner to paste.
- Where a technical choice exists, investigate, pick one, say why in a sentence,
  and proceed.
- The owner's decisions are product decisions: scope, pricing, what ships, what a
  user sees, and authorization for the Boundaries list. Bring those phrased as
  consequences, with a recommendation, batched.
- Report outcomes: what now works, what is at risk, what it costs. Keep paths,
  commands and diffs out unless asked.
- The owner's newest explicit statement wins over older plans and docs; update the
  older ones to match.

## Checks  [Default]

One command runs every local gate: `<gates command>`. The `checks` CI job runs
exactly that command, so "passes locally" and "passes in CI" mean the same thing.
When you add a gate, add it to that command, not beside it.

The contexts `main` actually requires are whatever the `main-protection` ruleset
lists; read it rather than assuming. If you need to know whether a check passes,
run it.

## Branches, worktrees and release  [Default]

- `main` is the trunk. Work happens on one branch per task, in its own worktree
  (the agent CLI's built-in worktree support, or `git worktree add`).
- Merging to `main` never deploys production. Production changes only through an
  explicit promote that the owner triggers, and the promote records the exact
  commit it shipped.
- Know the rollback path before the first deploy: an app rollback does not undo
  database, billing or data changes; those need a fix-forward plan.
- [Later: when several PRs need to ship together] integration branches
  (`integration/<date>-<topic>`) with the same required checks, and one batch PR
  to `main` that the owner merges.

## Secrets  [Invariant]

Secrets live in <the secret manager, e.g. a 1Password Environment>. Never read,
print, copy or create a `.env` with real values, and never paste a key into a
file, config or chat. Run anything that needs secrets through the manager
(e.g. `op run --environment <id> -- <command>`). `.env.tpl` lists the variable
names only. A new secret goes into the manager first, then its name into
`.env.tpl`.

## Memory  [Default]

Durable lessons go in `.claude/lessons/`, one per file, with a one-line summary at
the top: corrections and confirmed approaches, with why they mattered. Don't record
what Git already knows. Update rather than duplicate; delete what turns out wrong.
Cross-project lessons and session handoffs go in the owner's shared memory store.
