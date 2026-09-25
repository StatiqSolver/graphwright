# Design: releasing to production

**Level:** [Later: the first real deploy target]. The rule "merging to `main` never
deploys production" is [Default] from day one.

## Shape

```
PR ──checks──▶ main ──(owner runs "promote", with a reason)──▶ release ──▶ production
                                                                      └─▶ smoke test on the live URL
```

- Production builds from a `release` branch (or a release tag), never from `main`.
  Preview deploys per PR are fine and useful.
- **Promote** is a manual workflow (`workflow_dispatch`) that only the owner starts,
  with a required `reason`. It fast-forwards `release` to a commit on `main` and
  never force-pushes.
- Promote **refuses** a commit that isn't on `main`, isn't ahead of what's
  currently live (read from the host's own deployment record, not from a PR's old
  green run), or whose required checks on `main` aren't all green *now*. `main` can
  go red after a merge. An override input can exist, but it writes its reason into
  the permanent record.
- A `release-protection` ruleset stops anything but the promote workflow from
  moving `release`.
- Commit the rulesets as JSON in the repo (`.github/rulesets/`) and apply them with
  `gh api`. The source project kept them only as platform settings, and then nobody
  could see what they required without an API call.

## After deploy: a smoke test that knows what it's judging

- Trigger on the host's deployment-status event, filter in the job (that event
  can't be filtered at the trigger), and test the live URL through the core
  journey.
- Keep three outcomes apart: **the harness broke** (tooling failed, site not
  judged: fix the harness, don't roll back), **the site broke after a human
  release** (open an incident issue; the owner decides), and **the site broke after
  an automated release** (roll back and halt automation, then open an incident).
- Make the verdict a **named step that always runs** (`if: always()`), and have
  anything downstream read that step's conclusion by name. A job exit code hides the
  difference between "site broken" and "an artifact upload failed".
- If a privileged action (a rollback needing a protected secret) must follow an
  event whose ref isn't `main`, split it: a secret-less workflow receives the event
  and dispatches a second, privileged workflow pinned to `main`, passing only an
  opaque id that the privileged workflow re-derives everything from.

## Rollback: know what it undoes

The host's "roll back to the previous deployment" undoes **the deployed app only**.
It does not undo database migrations, billing side effects, data changes, or
separately deployed backend functions, and it may pause automatic domain
assignment until the next promote. Write a one-page runbook before the first
release that says, for each kind of change, whether rollback or a fix-forward
undoes it.

Also: "redeploy" on most hosts re-points to an already-built artifact rather than
rebuilding, so build-time environment values are frozen inside it. Verify what's
actually served, not the deployment's metadata.
