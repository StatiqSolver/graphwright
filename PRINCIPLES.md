# PRINCIPLES — why the system is shaped this way

This is the reasoning behind the kit, distilled from the owner's *Graph Engineering
Field Guide* (full text in [reference/](reference/Graph_Engineering_Field_Guide.html))
and from months of running an AI-agent-built product repo with it. Read
it once before bootstrapping; come back when a design choice isn't obvious.

The rules here are principles, not machinery. How the new repo implements them is
its own business, as long as it can pass the final design test at the bottom.

---

## 1. The spine: four graphs, one authority per fact

Delivery with AI agents runs on four kinds of relationship. Making them explicit
and queryable, instead of leaving them in prose and chat transcripts, is what
"graph engineering" means.

| Graph | The question it answers | Where its facts live (the authority) |
|---|---|---|
| **Work** | What can safely start now? | GitHub: milestones, parent issues, sub-issues, and "blocked by" links. Acyclic. |
| **Code / contract** | What must a worker read, and what else does this change affect? | The code itself: module boundaries enforced by an import-rule check, capability contracts, `upstreams.yaml`, decision records |
| **Evidence** | What is actually proven about this exact commit? | Check runs and review records on the PR's head SHA; screenshots and logs referenced from them |
| **Workflow state** | Which step is allowed next? | PR and issue state plus labels: build → check → review → bounded repair → merge, or park |

**Choose authority once, by field.** Requirements live in the issue tracker (or the
approved spec). Code identity lives in Git. Test outcomes live in captured run
evidence. Permissions live in trusted policy (`AGENTS.md`, rulesets, settings files).
Everything else is a **derived view**: rebuilt from those sources, never edited by
hand, never a second place to change the plan. In the Field Guide's words, a
generated index can be rebuilt, but "a second independently editable roadmap
requires synchronization, conflict resolution, and ownership."

The source project learned this the expensive way: at one point it had three
dependency authorities that disagreed (a large JSON edge list, several hand-built
files, and GitHub, which held one real dependency across hundreds of open issues).
**Make "blocked by" the only dependency authority from day one**, before anyone
builds a second list.

The placement rule: anything a person or agent needs to *decide* (what's ready,
what depends on what, whether something is accepted, waived, or approved) is
committed to Git or recorded on GitHub. Bulky material (logs, screenshots,
transcripts) can live elsewhere, but a committed record points at it.

**When the product spans more than one repo or pins upstream projects**, a release
is identified by a manifest that pins every repo's revision, every upstream's
revision, the lockfiles, and the fixture set. One commit hash cannot identify a
distributed product. For a product assembled from open-source projects, that's
most releases.

### Keep development memory and product memory apart
The graph that describes how the software is built (issues, decisions, reviews,
agent notes) is not the graph the product itself uses for its users' data. They can
share patterns, but they need separate permissions, storage and retention. Internal
developer notes must never flow into a user-facing agent's context. For an AI
engineering tool that will likely have its own knowledge graph, decide this split
on day one.

### The five planes (where responsibility sits)
Logical boundaries, not services to deploy:
- **Control** owns the approved plan, what's eligible, budgets and escalation. It
  never invents scope or relaxes acceptance.
- **Execution** makes bounded model calls and isolated edits. It never gives
  final acceptance or expands its own privileges.
- **Artifact** stores immutable versions, logs and screenshots. A file doesn't exist
  just because an agent said so.
- **Graph** links records. It never guarantees an extracted link is true.
- **Evaluation** runs rubrics, checks and oracles. Green checks don't grant release.

"The graph connects knowledge and work; it does not become the worker, the file
store, the evaluator, or the source of permission."

---

## 2. Done is a vector, not a green badge

Track these separately and never average them into one "done":

1. **Implemented** — code merged.
2. **Automatically verified** — the checks passed *on that exact commit*.
3. **Domain-qualified** — correct against independent known answers.
4. **User-qualified** — seen working by a person in a real browser or app.
5. **Released** — running in production, by an authorized promote.
6. **Current** — nothing it depended on has changed since.

The allowed outcomes of any check are **pass, fail, unrun, blocked, inconclusive**,
under four rules that are never relaxed:

- **Skip is never pass.** A test that skips when the thing it tests is missing is
  worse than no test.
- **Prose is not execution evidence.** "Looks good except for a few edge cases" is
  not an acceptance record.
- **A worker may not relax its own rubric.** Workers can propose better tests; a
  separate review changes the rules.
- **A changed candidate needs reassessment.** After any fix, rebase or merge, the
  affected evidence is void until re-run. In the Field Guide's example,
  "evaluation-42 EVALUATES candidate-B" is precise; "EVALUATES latest" is not.

---

## 3. The loop: one bounded candidate at a time

A useful loop has a goal, permitted actions, an evaluator, a state change and a
stopping condition:

> understand a bounded problem → make one candidate change → collect evidence →
> accept, repair within budget, or park.

- **Use deterministic code for state and authority; use models for judgment within
  that state.** Who may merge, what's ready and what passed are code and
  configuration, never a prompt.
- **Repairs are bounded.** Review → one fix with regression evidence → a re-review
  of the earlier findings plus anything the fix broke (not a full re-audit) → if
  still blocked, one fresh diagnosis and one materially different fix → if still
  blocked, **park it and let independent work continue**. A parked item with its
  evidence preserved is a legitimate stop, not a failure to hide.
- **Escalate specifically.** One decision needed, why it can't be inferred, the
  options with their consequences, what's preserved, what else can still run, and
  the exact condition to resume. Never a transcript dump.
- **Restartable by design.** Checkpoint after each step. A fresh session must be
  able to resume from Git and GitHub alone, and must never assume an action didn't
  happen just because it saw no acknowledgement. Classify external effects (read
  only, safely repeatable, not repeatable) and inspect remote state before replaying
  anything ambiguous.
- **Budgets count the whole tree.** Child agents, retries and repairs all draw from
  the parent's budget, and an advisory sentence in a prompt is not a cap.

---

## 4. Context: deliver the relevant subgraph, not the history

- A worker gets a **context packet**: objective, acceptance, authority, the baseline
  commit, the shared contracts it touches, known counterexamples, its permitted
  capabilities, and its budget. Retrieval is bounded (a few typed edges, at most two
  hops), with an explicit list of what was left out and how to fetch it.
- If a mandatory section is missing, the packet isn't **sufficient to start**: the
  worker splits the task or asks, and never quietly drops a permission, a
  contradiction or a prerequisite to fit a budget.
- Recheck freshness at dispatch, not just when the packet was built.
- **Reviewers get the contract and the diff, not the builder's narrative.** The
  builder's persuasive summary is exactly what a reviewer shouldn't anchor on.
- Pick **one** packet shape per project and keep it. The source project ended up
  with two differently shaped contracts, and the rule it had to write afterwards
  was: don't run the second shape as a parallel source of truth; map its ideas
  into the one shape.
- Introduce any new context-selection mechanism in **shadow mode** first: build
  the packet and log it while the old behavior still runs, compare, then enforce.

Start with hand-written "Read first" sections in issues. Automate only what you
find yourself repeating. In the source project's comparison run, agents given
nothing but a well-written issue (acceptance criteria, allowed paths, required
tests) matched agents given a custom-built context bundle backed by an extra
model: the extra system bought nothing measurable.

---

## 5. Lean delivery: earn every piece of machinery

The source project built roughly 47,000 lines of custom delivery machinery
(schedulers, ledgers, gates, merge lanes, receipts; an estimate from a line count),
and an audit judged about 80% of it replaceable by built-ins or deletable. What the
audit measured over about a month:

- A hand-kept ledger of unfinished work grew from 34 to 94 open rows; 62 of the 94
  never became issues, so nothing that schedules work ever saw them. Its 27
  consistency-check failures were all the ledger tripping over itself; none caught
  a false "done".
- A custom issue-closer closed 7 issues in 107 runs. GitHub's native "Closes #N"
  closed 133 in the same window.
- Of 130 failures from its "quality" gates, about 7% were real defects and about
  42% were the process checking its own paperwork.
- A self-merge lane that a required check existed to feed merged 4 of 199 PRs.
- Known-answer tests on the computational core (mostly hand-derived expected
  outputs): zero failures and zero noise over the month, as a cheap standing
  guard. The audit judged a second AI vendor's review the highest-yield check: it
  found serious problems (a failed load reported to the user as a success; an
  estimate shown as if it were computed) that no automated gate caught. Browser
  tests caught real regressions, at roughly one real catch per three or four noise
  failures in CI.
- The defect that reached users (a wrong number on a results screen) passed every
  check, because every test stopped at the core and none checked what the screen
  actually said.

The lessons, in rule form:

- **No new automation unless it replaces a requirement or closes a demonstrated
  gap.** "It would be nice" and "a template suggested it" are not gaps.
- **Built-ins first.** GitHub already provides sub-issues, "blocked by", issue
  search for unblocked work, Projects with views and charts, rulesets, auto-merge,
  required checks, and native "Closes #N". The agent CLIs already provide
  worktrees. The host platform already provides preview deploys and rollback.
  Custom code is for the project's own policy, and even that should be small.
- **Measure a gate by what it catches.** Track true positives. A gate whose
  failures are mostly bookkeeping is a tax: fold its real checks into the main
  gates command and delete the rest. Known-answer tests and an independent second
  reviewer caught real defects; paperwork gates largely didn't.
- **Test the thing users see.** Known answers are needed at both boundaries: the
  core's output, and the exact text the user reads after formatting, rounding and
  labelling.
- **Required checks: one per concern, clearly named.** The source project settled
  on three: deterministic correctness (`checks`), real-environment behavior
  (`browser`), and independent review of the exact commit (`independent-review`).
  Fewer isn't simpler if one check silently bundles unrelated concerns; when it
  fails, nobody knows what broke. Inside a check, run each gate as a named step.
- **Audits are events, not systems.** A one-time audit that finds real gaps is
  valuable. Turn its findings into ordinary issues with a label; don't build a
  permanent second tracker to keep them.
- **Delegation is real or it isn't.** If an agent is trusted to judge something,
  let it decide with evidence on the record. Asking the owner to rubber-stamp a
  batch of verdicts the agent already reached adds a step and no safety.
- **One of each.** One tracker, one plan, one decision log, one glossary, one
  instructions file. Add a second only when the first measurably fails.
- **Add before remove.** Never delete a workflow a required check still depends
  on. Change the ruleset first, then delete.
- **Retire on four points:** code, workflow, external service, and credential. A
  "removed" tool whose key still works isn't retired.
- **Prove the whole loop, including an interruption,** before relying on it, and
  plant defects (negative controls) to prove each gate can fail.

### The eight anti-patterns (from the Field Guide)

| Anti-pattern | Why it fails | Prefer |
|---|---|---|
| The giant orchestrator conversation | Every transcript becomes more context to carry and misread | Small packets, versioned artifacts, coordination in code |
| The graph as a truth machine | Structured mistakes become reusable assumptions | Source-backed claims, labelled inferences, independent evaluation |
| Task count as progress | Many small tasks can hide a broken user journey | Accepted end-to-end outcomes |
| The swarm before the reducer | Work grows faster than it can be checked or integrated | Define integration, dedup and evidence before fanning out |
| The prompt as a permission boundary | A model can ignore prose; side effects still happen | Enforced capabilities, sandboxes, scoped approvals |
| The latest note as current truth | Recency gets confused with authority | Pinned sources, explicit supersession |
| Green tests as universal readiness | Fixtures get mistaken for real integration | Separate implemented / verified / qualified / released |
| The new platform before the first result | Orchestration consumes the project it was meant to speed up | A small shadow index and one measured pilot |

### Don't build these early

| Not yet | Build it when |
|---|---|
| A graph database | Traversal, scale or concurrency actually exceeds JSON plus a script |
| A new orchestrator, or a separate orchestrator repo | Never by default; the hard integrations remain whatever runs the state machine |
| A vector store or semantic extraction | IDs and explicit links stop answering the questions |
| Multiple writers on shared code | Changes stop touching shared contracts |
| Large parallel fan-out | A reducer, a budget and an integration step exist and are proven |
| A standing monitor or rescue agent | Never alongside bounded repair budgets; two rescue systems fight |
| A shared toolkit repo | A second product actually needs the same code |

---

## 6. Autonomy is earned in tiers

- **Tier A — read and analyze.** Always allowed.
- **Tier B — change an isolated candidate** (a branch in its own worktree), no
  production access. The normal agent tier.
- **Tier C — publish** (merge, deploy, message, spend, touch an upstream repo), only
  under a scoped, explicit authorization tied to the exact candidate and target.
  In this kit that authorization comes from the owner, per action.

Prompt text never grants a tier. Settings, rulesets and credentials do.

---

## 7. Measure from day one what's hard to add later

Structure gets built early because agents can't work without it. Two things
don't, and the source project found both missing when it wanted to compare
approaches:

- **Provenance.** Record the model and runtime that actually ran, not the one that
  was configured. A role label in a config file isn't evidence of what served the
  request.
- **Cost per accepted outcome.** All attempts, repairs, reviews and diagnoses,
  divided by outcomes that were actually accepted. Zero accepted outcomes means
  undefined, not free. Report savings only alongside accepted scope and escaped
  defects.

Cheap to start: put the runtime, model and rough usage in each PR's evidence
section, and use a usage tracker for the subscriptions.

---

## 8. The final design test

> "Can a fresh contributor trace every important output to its objective, source
> versions, artifact, run, evaluation, and unresolved limits? If not, improve the
> records and controls before adding more agents."

If a fresh agent, given only an issue number, can find what to do, what's been
done, what's proven, and what's still open, the system is working, however it's
built.
