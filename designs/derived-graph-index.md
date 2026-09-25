# Design: the derived graph index and context packets

**Level:** [Later] — build the work-graph part when there are roughly 30+ open
issues, or the first time an agent picks the wrong next task. Build the evidence
and context parts when parallel agents start colliding or reading the wrong files.

## The one rule

The index is **derived**: rebuilt from its sources on every run, never edited by
hand, never written back to its sources. If it disagrees with GitHub or Git,
GitHub or Git is right and the index has a bug. It is not a second roadmap.

No graph database, no orchestrator service. A script, the GitHub GraphQL API,
`git`, and a JSON file are enough until a measured need says otherwise.

## Sources (each fact has one authority)

| Fact | Authority |
|---|---|
| What we intend to build, and its dependencies | GitHub issues: milestones, sub-issues, "blocked by" |
| What the code is | Git commits |
| Which modules depend on which | The import graph, read from the code |
| What passed, on which commit | Check runs and review records on the PR head SHA |
| Why we chose something | `docs/decisions/` |
| Which upstream version we use | `upstreams.yaml` |

## Output: `graph.json` (generated, git-ignored or attached to a run)

Nodes: issue, PR, commit, check run, review round, decision, upstream, module.

Edges (use a small fixed set):
- `BLOCKED_BY`, `CHILD_OF` — work graph
- `CLOSES`, `TOUCHES` (PR → module) — links work to code
- `DEPENDS_ON` (module → module, module → upstream adapter) — code/contract graph
- `VERIFIED_BY` (PR head commit → check run or review round) — evidence graph
- `SUPERSEDES` (decision → decision), `PINS` (release → upstream revision)

## Queries worth having

- **Ready set:** open leaves with no open blocker, in a milestone, not deferred,
  not in progress. Must equal the board's Ready view; a mismatch is a bug.
- **Unblocks the most:** reverse reachability over `BLOCKED_BY`.
- **Stale evidence:** a PR whose latest check or review ran on a commit that isn't
  its head.
- **Collision risk:** Ready items whose likely modules overlap, so parallel agents
  don't edit the same files.
- **Context seeds:** for one issue, the modules its sibling and parent PRs touched,
  the contracts those modules depend on, and the decisions that cite them.

## Context packets (what a worker must read)

When dispatching an agent on a work item, build a context packet shaped like
`docs/graph/templates/context-packet-contract.json`:

- the mandatory sections: objective, acceptance, authority, baseline commit,
  shared contracts, known counterexamples, capabilities, budget;
- bounded retrieval: start from the issue, follow only the allowed edge types, at
  most two hops;
- an explicit omissions list (what was left out, why, and how to fetch it) and a
  conflicts list;
- `sufficient_to_start`: false if a mandatory section is missing. Then the worker
  stops and asks instead of guessing.

Start by writing packets by hand in the issue's "Read first" section. Automate only
what you find yourself doing repeatedly.
