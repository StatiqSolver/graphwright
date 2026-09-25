# Design: how the agents get their rules, settings and workspaces

**Level:** [Default].

## One rulebook, two readers

- `AGENTS.md` at the repo root is the rulebook. It's the converging cross-vendor
  convention: Codex reads it natively.
- `CLAUDE.md` is a delivery adapter holding only `@AGENTS.md`, which imports the
  rulebook into Claude Code's context. It adds no policy. If a future Claude Code
  version reads `AGENTS.md` natively, check that before shrinking `CLAUDE.md`
  further; loading the file twice is harmless, missing it is not.
- Keep `AGENTS.md` under about 200 lines, and well under Codex's default
  project-instruction size limit (`project_doc_max_bytes`, 32 KiB at the time of
  writing; verify), past which the file is cut off without warning.
- The source project kept its rules in a differently named file and generated
  `AGENTS.md` from it, with a CI drift check and an editor hook. That machinery
  exists only because there were two files. With one file, it isn't needed.
- Nested `AGENTS.md` files in subfolders (e.g. one per upstream adapter) are fine
  for local facts ("this adapter wraps X at pin Y; its contract tests are Z"). They
  never restate or override root policy.

## Claude Code settings (`.claude/settings.json`)

The starter file does three things:

1. **Denies reading secret files** (`.env`, `.env.local`, `.env.*.local`, the
   production and staging variants, at any depth) and **denies the secret
   manager's reveal commands** (`op read`, `op item get --reveal`) in both Bash and
   PowerShell. The agent can still run commands *with* secrets injected by
   `op run`; it just can't read a value. Assume an agent will try these, and block
   them in settings, not in prose.
2. **Sets `CLAUDE_CODE_SUBAGENT_MODEL` to `inherit`.** Any other value silently
   overrides every subagent's explicit model choice, so all model routing becomes
   a no-op with no error. Then pass an explicit model tier on every subagent call.
3. **Disables bypass-permissions mode and auto-enabling project MCP servers.**

Build the `allow` list from real use: add narrow, read-only or already-safe commands
when their prompts become repetitive. Don't start with a broad allowlist.

[Later] A Stop hook that runs the fast part of the gates when the tree is dirty,
failing open (it reports, it doesn't block). Add it when agents repeatedly end
turns with type errors.

## Codex

- Codex reads `AGENTS.md`. Trust the repo in Codex's own config so it loads it.
- Codex has no file-level read deny like Claude Code's. The deny rules are defense
  in depth; the actual guarantee, for both agents, is that no file with real secret
  values exists on disk, because `op run` injects them into the process.
- Run it on the owner's subscription sign-in, never an API key, unless the owner
  approves spending.
- When Codex is driven from inside a Claude session or workflow, treat it as a
  builder only. Your own agent keeps the tests, commits and review. Launch it from
  the main session as a monitored background process; a short-lived workflow step
  gets torn down before a slow external job finishes.

## Worktrees

- One branch per task, one worktree per branch, one writer per worktree. Use the
  agent CLI's built-in worktree support where it exists; otherwise
  `git worktree add ../<repo>-wt/<slug> -b <agent>/<slug>`.
- Refuse to reuse a branch name that already exists.
- Keep worktrees outside the main checkout's folder (nested worktrees make bulk
  deletes and searches dangerous) and out of ordinary searches.
- **On Windows, never let a worktree contain a directory junction** (for example a
  linked `node_modules`) that you might later delete recursively: a recursive
  delete follows the junction and wipes the target, which another session may be
  using. Install dependencies per worktree, or remove the junction itself first.
- If an editor's Git integration misbehaves after `git worktree add` (some can't
  parse the `extensions.worktreeConfig` setting Git adds), write a small idempotent
  repair script and run it after every worktree creation. Only if it happens.

## Model routing

Name a tier on every subagent dispatch:
- the cheapest tier for search, inventory and mechanical edits;
- the middle tier for ordinary execution (the default);
- the top tier for judgment, review and hard bounded problems;
- never a tier above the driver's own. A lower tier can't meaningfully judge a
  higher one's work, and a session can't raise its own model; the owner picks
  the session model.

Delegate work that's genuinely independent and big enough to pay for its own
context (roughly: more than three tool calls, more than five files or 500 lines,
or a lot of raw output to digest). Keep the judgment calls, the final
verification and anything user-visible in the driving session. Read a file after
a subagent says it wrote it.
