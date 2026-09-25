# Design: the `independent-review` check

**Level:** [Later] — build when a self-hosted runner can hold a second vendor's
CLI signed in on the owner's subscription. Until then, get a fresh-context review
of each PR's head commit by hand (a new session that didn't write the code), and
say in the PR which vendor reviewed it.

This is a design, not code to port. The source project's implementation grew to
several hundred lines of script and around a hundred decision tests, almost all of
them earned by specific failures. Start with the core and add a part only when its
failure shows up.

## What it's for

A second reviewer from a different AI vendor judges the **exact head commit** of
every PR, and its verdict is a **required status check**, so it can block a merge.

Why this exists:
- One vendor's reviewers share blind spots. In the source project a second-vendor
  pass over eleven PRs already approved by one vendor found four serious problems.
- Built-in PR review bots comment but don't produce a gating status tied to a
  commit, so they can't block a merge on their own.
- A review of an older commit says nothing about the current one.

## The core (build this first)

1. **Trigger.** On `pull_request` (opened, synchronize, reopened, ready for review)
   for PRs targeting `main` (and `integration/**` once those exist). Skip PRs from
   forks: the runner holds the owner's sign-ins.
2. **Runner.** One self-hosted runner with a dedicated label (e.g. `review`) that
   only it carries. It has the reviewer CLIs installed and signed in with the
   owner's subscriptions, never paid API keys.
   - Install each CLI's full distribution: a bare binary may lack the helpers it
     needs to run commands.
   - Pin the model the way the CLI accepts it (some accept only an alias); prove it
     with a dry run before relying on it.
3. **Exact commit.** Check out the PR's head SHA, review the diff against its base,
   and post the result on that SHA. A new push means a new review.
4. **Prompt.** The diff, the PR text, the linked issue's acceptance criteria, and a
   severity rubric (P0 = unsafe or broken for users … P3 = minor), asking for a
   structured answer: a verdict plus findings, each with a title, severity,
   file:line and evidence. Keep the prompt in a file and check its hash in the job,
   so it can't be quietly softened.
5. **Gate.** Pass only when the verdict is approve **and** every finding is P2 or
   P3. Anything else fails. A crash, timeout or unparseable answer fails: skipped
   is never a pass.
6. **Fallback.** Only a clear out-of-quota error from the primary reviewer switches
   to the other vendor (the one that didn't write the code, if known), and the PR
   gets the `review: single-vendor` label. If neither is available, the check fails
   and the change waits. An owner-set repository variable such as
   `REVIEWS_OFF_UNTIL=YYYY-MM-DD` turns reviews off: PRs get `unreviewed` and
   nothing auto-merges.

## Parts to add when their failure shows up

- **Round memory** (add when re-reviews start re-raising settled points). One bot
  comment per reviewed head records the round. From round 2, the reviewer gets the
  earlier rounds and must re-check each open finding by title first.
- **Tiers and budgets** (add with AGENTS.md "Good enough"). Map paths to a tier
  (critical / user-facing / behind the scenes). Each tier has a fix budget and a
  ceiling; at the ceiling the check labels the PR `needs:decision` and stops
  asking for more.
- **Triage past the budget.** Findings outside the change's files, tests nobody
  asked for, and (below critical) things unreachable before release get filed as
  `known-gap` + `from-review` issues and stop blocking. Anything that could hurt a
  user, their data or their money keeps blocking.
- **Written decisions.** A finding is declined by a PR comment that quotes its
  title and cites evidence: file:line, a named test, or a link to an earlier
  decision. Only comments from people with write access count, and that is checked
  through the API, not inferred. An unrelated comment clears nothing, and a
  decision stops applying once the code it cites changes.
- **Credential hygiene.** The sign-in tokens are only present in the steps that
  need them. Any reviewer output shaped like a credential is withheld from the PR.
  Anyone who can push workflow changes can read the runner's sign-ins; the owner
  accepts that risk explicitly, in a decision record.

## Prove it before making it required

1. **Negative controls.** Plant defects in throwaway PRs (a type error, a stale
   commit, a flipped condition, a skipped review) and confirm each is not green on
   its exact head. Commit before planting a mutation, so reverting it can't destroy
   uncommitted work.
2. **Comparison.** Replay 5–10 historical PRs with known outcomes and compare. Read
   any disagreements by hand; a stricter check is fine if its extra blocks are real.
3. **Replay real history** for each added part (e.g. feed round memory the actual
   sequence of a past PR's rounds and check it passes, blocks and escalates where
   it should).
4. Only then add `independent-review` to the ruleset's required contexts.

## What to expect

- **Port the calibration, not just the prompt.** The source project's first run of
  its new check rated two planted serious defects as minor and let them pass, until
  the older review path's severity rules (a "severity floor") were carried over.
- **Exact-commit review serializes work.** Any later push voids the review, so
  agents rationally wait for every other check to go green before spending a
  review. That roughly doubled wall-clock time per PR. Accept it knowingly.
- **A single AI reviewer is a sample, not proof of absence.** In a comparison, the
  new check blocked every commit the old path blocked, but for its own reasons; it
  missed three of the five specific serious bugs the old path had named.

- A review gate reviewing its own changes never reaches "nothing left to say".
  Adversarial targets hit their ceiling; bring the owner the decision, don't keep
  patching.
- Count review rounds by distinct reviewed commits, not by the number of verdict
  comments.
- After two review passes in a row find new serious problems in the same area,
  stop patching and redesign that area once.
