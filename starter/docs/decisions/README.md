# Decisions

One file per decision that outlives a single PR: `NNNN-short-title.md`, numbered in
order, written from `docs/graph/templates/decision-record.md`.

- A decision is PROPOSED until the owner (for product decisions) or the agent that
  owns the engineering choice records approval in the file. A model's proposal is
  not an approval.
- Never edit an accepted decision to mean something else: write a new one that
  supersedes it and link both ways.
- Every decision names its revisit trigger.
