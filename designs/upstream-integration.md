# Design: building on open-source upstreams

**Level:** [Invariant] for the rules; [Default] for the choices.

This product is defined by bringing existing open-source projects together. The
source project had one such dependency and learned the rules the hard way; here
they apply from day one.

## Rules

1. **Register before import.** An upstream gets an `upstreams.yaml` entry (license
   read from its LICENSE file at the pinned revision, pin, mode, placement,
   adapter path) before any of its code lands.
2. **Pin exactly.** A tag or full commit SHA. Never a floating branch, never
   "latest".
3. **One door per upstream.** Only its adapter imports it. The core and the apps
   talk to the adapter's contract. Enforce this with an import-boundary rule in
   the gates command, so a stray import fails `checks`.
4. **Contract tests at the door.** Each adapter has tests that pin the behavior we
   rely on, plus known-answer fixtures: inputs whose correct outputs we know
   independently of the upstream (hand-computed, from a reference, or from a
   second implementation). These are what make an upstream bump safe.
5. **A bump is its own PR.** Change the pin, run the contract tests and known
   answers, read the upstream's changelog for the range, and record anything that
   changed behavior.
6. **License policy is the owner's**, decided at bootstrap. Anything that falls
   outside it needs a decision record and an entry in `NOTICES.md`. Pattern from
   the source project: a copyleft component shipped as a separate, replaceable file
   so users can substitute their own build, deliberately not integrity-pinned for
   that reason, and documented in `NOTICES.md`.
7. **Never change an upstream's repository without authorization.** Opening an
   issue or PR upstream is publishing; it needs the owner's yes.

## Choosing an integration mode

| Mode | Use when | Watch out for |
|---|---|---|
| Package (npm, PyPI, crates, …) | It's published and versioned | Transitive licenses; lockfile is part of the pin |
| Subprocess / CLI | It's a separate program, or its license must stay at arm's length | Startup cost; version drift on users' machines; bundle it or check its version |
| Network service (local or remote) | It's heavy, stateful, or not embeddable | Availability, auth, latency; the adapter owns retries and timeouts |
| WASM / in-browser | It must run in the web app without a server | Bundle size, threading, memory; license terms on distribution |
| Vendored copy | It's small, unmaintained, or needs a patch | You now maintain it; record the upstream SHA it came from |
| Submodule | Rarely; you need its full source tree at a pin | Tooling friction for agents and worktrees; prefer the others |
| Fork | Last resort, when a fix won't be accepted upstream | A fork is its own upstream entry with a rebase policy |

## Placement: where each piece runs

Local app plus web app means every capability has a placement question: browser,
local process, or server. Record it per upstream in `upstreams.yaml` and per
capability in decision 0001. Placement drives licensing (does the code ship to
users?), security (can an in-product agent reach the user's files?), and what the
web app can do when the local app isn't running. Settle it in writing before
building the second capability.

## Release manifest  [Later: the first release involving more than one repo or a pinned service]

A generated file per release that pins this repo's commit and every upstream's
revision, copied from `upstreams.yaml`. It answers "what exactly did users get?"
without anyone reconstructing it from memory.
