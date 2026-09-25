# NOTICES.md — third-party code we ship

One section per third-party component whose license carries obligations beyond
"keep the copyright notice" (anything not MIT/BSD/Apache-style), and for any
exception to the license policy in `docs/decisions/`. `upstreams.yaml` stays the
authority for pins; this file holds the compliance argument.

<!-- Copy this block per component.

## <Component name> — <SPDX license>

- **What ships:** <exact file(s) users receive, and in which app: web / local>
- **Pinned:** <version>, upstream commit <sha>, built with <toolchain + image digest>
- **How it's linked:** <separate file loaded at runtime / subprocess / service>;
  never statically linked into our code.
- **Why this complies:** <the argument, e.g. "users can substitute their own build
  of this file">
- **Property that must not be "improved" away:** <e.g. "deliberately not
  integrity-pinned, because pinning would stop users substituting a build">
- **Rebuild / replace recipe:** <steps, with hashes of the shipped artifact>
- **Decision record:** docs/decisions/<nnnn>-<title>.md
-->
