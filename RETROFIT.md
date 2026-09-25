# RETROFIT — bringing Graphwright into an existing repo, in order

Use this path when the repo already has code, history, users, or its own way of
working. (For an empty repo, use [GREENFIELD.md](GREENFIELD.md).)

A retrofit is harder than a fresh start for one reason the Field Guide names
directly: the first problem "is often not missing code. It is disagreement among
the checkout, issue tracker, local work, deployed behavior, and historical notes."
So the order is: **see everything, change nothing → decide one authority per fact →
add the new rails beside the old → prove them → switch → only then remove.** The
product keeps shipping the whole time.

This path is modelled on a real retrofit: the source project's September 2026
reorganization, which moved a repo with about 47,000 lines of custom delivery
machinery onto this system in stages, without pausing releases.

Read first: [README.md](README.md) → [PRINCIPLES.md](PRINCIPLES.md) → this file →
[starter/AGENTS.md](starter/AGENTS.md). Keep [LESSONS.md](LESSONS.md) open; most of
its git and CI lessons were learned during exactly this kind of work.

Same levels as everywhere else: **[Invariant]**, **[Default]**, **[Later: trigger]**.
Every step ends with an **exit test**. Record evidence in one retrofit tracking
issue, starting with the Graphwright commit you're working from
(`git -C graphwright rev-parse HEAD`), and add `graphwright/` to
`.git/info/exclude` so the kit itself is never committed.

---

## Ground rules for the whole retrofit  [Invariant]

- **Don't tidy to look orderly.** Don't clean up dirty files, reset branches,
  rewrite history, drop stashes, or close broad sets of issues just to make the new
  system appear consistent. Every piece of local work is preserved until the owner
  says otherwise.
- **Add before remove.** The new thing runs beside the old one and is proven
  before the old one goes. Never delete a workflow a required check still depends
  on: change the ruleset first, then delete.
- **Don't build what's already there.** When the source project was audited against
  the Field Guide, it already had 8 of the 10 capabilities through ordinary Git,
  issues and CI. Check before building; rename nothing for vocabulary's sake where
  the current thing works.
- **Freeze, don't fight, the old machinery.** Existing custom delivery tooling gets
  fixes only, no new features, until it's retired.
- **Run the retrofit as ordinary PRs from one session**, outside any existing
  automation that might try to "help".
- **The owner's newest statement wins** over any plan or doc in the repo.

---

## Stage A — Information and state (no behavior changes)

### A.1 Inventory every working state  [Invariant]
Before changing anything, list:
- every branch (local and remote), worktree, stash, unpushed commit and dirty file,
  with how far each is ahead of or behind trunk;
- open PRs and their state; what is actually deployed (read the host's own
  deployment record, not a doc);
- what the rulesets or branch protection actually require (read them through the
  API);
- every place work is tracked: issues, Project boards, milestones, TODO files,
  ledgers, plan and status documents, dashboards, memory notes;
- every agent instruction file (`CLAUDE.md`, `AGENTS.md`, editor rules files,
  skill folders) and where they contradict each other;
- every workflow and script, and for each gate, what it has caught: true positives
  versus noise over the last month of CI history;
- how secrets are handled today, and every third-party dependency that ships to
  users, with its license.

Don't assume remote `main` is the newest state; local branches and worktrees often
hold work that never reached it.

**Exit:** the inventory is posted on the retrofit issue, and nothing in the repo
has changed.

### A.2 Adopt a baseline and preserve local work  [Invariant]
Pick the adopted baseline commit and record what it excludes and why. Preserve
everything else as refs, not stashes: push unmerged local work to archive branches
or tags, and export stashes to named refs. Snapshot worktrees before touching
them. Remember that a recursive delete of a worktree containing a directory
junction deletes the junction's target too.

**Exit:** the owner and a fresh agent session agree on the baseline, the preserved
local work, the unresolved conflicts, and the next safe task. Restoring one
preserved item has been tried and works.

### A.3 The authority map  [Invariant]
This is the heart of the retrofit. For each kind of fact, list every place it lives
today and choose one authority:

| Fact | Usual authority after retrofit | Everything else becomes |
|---|---|---|
| What we'll build | GitHub milestones, issues, sub-issues | archived or migrated |
| What depends on what | GitHub "blocked by" | deleted (a second dependency list always drifts) |
| What's unfinished but shipped | `known-gap` issues | the old ledger, frozen as an archive |
| Why we chose something | `docs/decisions/` | linked from the decision, or archived |
| What passed | check runs and review records on the head commit | derived views |
| What's deployed | the host's deployment record | never a status doc |
| Policy | `AGENTS.md`, rulesets, settings | merged into `AGENTS.md` or deleted |
| Upstream versions and licenses | `upstreams.yaml` | derived |

Write it as a decision record, and get the owner's yes, phrased as consequences
("the ledger stops being where unfinished work lives; issues are").

**Exit:** decision record accepted. Every fact kind has exactly one authority, and
every other location is marked derived, to-migrate, or archive.

### A.4 One rulebook  [Default]
Merge the existing instruction files into one `AGENTS.md` from the starter
template, with `CLAUDE.md` importing it. Keep the project's own truth rules and
hard-won rules (with their reasons); add Graphwright's invariants; resolve
contradictions (the source project found one policy sentence copied into eight
places, worded inconsistently). This is the cheapest high-value step, so do it
early.

**Exit:** fresh Claude Code and Codex sessions give the same, correct answers to
five questions about the repo's rules.

### A.5 Board, labels, forms  [Default]
Add the starter's labels, issue forms, PR template (with its "Known gaps" section)
and the Project board views (Ready, In progress, Blocked, Known gaps) beside
whatever exists. Nothing old is removed yet.

**Exit:** the Ready view shows what the definition says it should.

---

## Stage B — Switch the guardrails

### B.1 One gates command; `checks` in parallel  [Invariant]
Fold every existing check that has caught real problems into one gates command.
Drop what only ever caught its own paperwork, but keep a list of it for the owner.
Watch for real behavior tests bundled inside tooling you plan to retire (the source
project nearly deleted a refactor-safety test suite that lived inside a code-graph
tool): relocate those first.

For rules the existing code already breaks (an import boundary, a lint rule),
add them as **ratchets**: record today's violations as an allowed baseline that
may only shrink, so new violations fail and old ones get paid down over time.
Don't hold the retrofit hostage to a big-bang cleanup.

Run the new `checks` beside the old required checks.

**Exit (negative control):** planted defects turn `checks` red on the exact head
commit. The old checks still run.

### B.2 Migrate records by disposition  [Invariant]
Every item in an old tracker (ledger rows, TODO files, plan documents) gets exactly
one disposition:
- **still real** → an issue (`known-gap`, bug, or work item);
- **already fixed** → closed with evidence;
- **duplicate** or **superseded** → pointer to the survivor;
- **deferred** → the reason, the user-visible outcome, and a reopen trigger;
- **invalid** → why.

The completeness test is "every old item has a disposition", not "every old item
became a new issue": that just imports stale noise. Keep a small disposition map
(old id → where it went) if code comments cite the old ids, then freeze the old
tracker as an archive and add no rows to it. Convert every real dependency into a
"blocked by" link.

Move history out of the working tree: an archive tag plus an `ARCHIVE.md` listing
what left and where to find it.

**Exit:** every old item has one disposition; the old tracker says "frozen" at the
top; the Ready view is correct.

### B.3 Shadow, then prove  [Invariant]
Run a few real work items through the new path while the old one stays
authoritative. Then:
- **the interruption test**: write the pass criteria first, stop a session
  mid-item, and have a fresh agent resume from only "resume issue #N";
- **replay real history** for anything that makes decisions (e.g. feed a new review
  check the actual sequence of a past PR's rounds, and confirm it passes, blocks
  and escalates where it should);
- **compare** the new path's verdicts with the old one's on the same items, and
  read the disagreements by hand.

**Exit:** the pre-written criteria pass, and the owner has seen the comparison in
plain terms.

### B.4 Switch the ruleset  [Invariant]
Change the ruleset to require the new contexts (`checks` first; `browser` and
`independent-review` when they've met their triggers and passed their own negative
controls). Avoid switching required checks in the middle of a sensitive release
unless the owner says go. Confirm `main` is green afterwards.

**Exit:** the ruleset lists exactly the intended contexts; a planted defect is
blocked from merging (not just red); `main` is green.

---

## Stage C — Remove what's no longer reachable

### C.1 Retire on four points  [Invariant]
For each retired mechanism, confirm all four:
1. **code** — deleted, or deliberately kept and documented;
2. **workflow** — no schedule or trigger still runs it;
3. **service** — no process, runner, cron job or hosted instance still runs it;
4. **credential** — every key or token it used is revoked.

Code is the easy one. A forgotten cron job or a live key nobody remembers is the
usual leftover.

**Exit:** a four-point checklist per retired mechanism, posted on the retrofit issue.

### C.2 One deletion sweep  [Default]
Delete the unreachable code in one sweep, after stage B's exit tests. Clean up stale
branches and worktrees only from the preserved snapshot of A.2, and bulk deletions
wait for the owner's explicit go.

**Exit:** `main` is green; the inventory from A.1, re-run, shows only what the
authority map says should exist.

---

## Pacing

The stages run back to back: each starts when the previous stage's exit test
passes. The source project first planned a week per stage with a pause for a launch
week. Stage A took hours, and the owner dropped the pause. The binding constraints
are the ordering rules (add before remove; ruleset before deletion), not the
calendar.

## What usually needs an owner decision during a retrofit

Batch these, phrased as consequences, each with a recommendation:
- the authority map (A.3), especially retiring any tracker the owner personally uses;
- freezing existing automation;
- which items get deferred rather than fixed (B.2);
- switching required checks (B.4);
- bulk deletion of branches or worktrees (C.2);
- anything in [DECISIONS-FOR-OWNER.md](DECISIONS-FOR-OWNER.md) the existing repo
  hasn't already answered. Pre-fill each from what the repo already does, and ask
  only what's genuinely open.
