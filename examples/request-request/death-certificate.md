# Death Certificate — request/request

> Example Gravedigger output, filled from REVENANT's analysis of this
> repository. The score below is produced by REVENANT's deterministic
> scoring code, not by the Gravedigger agent (Gravedigger supplies facts
> only). Timing for `pushed_at` is approximated from that analysis
> rather than re-fetched live for this example document.

## Identity

| Field | Value |
|---|---|
| Repository | `request/request` |
| URL | https://github.com/request/request |
| Description | Simplified HTTP client for Node.js (the project's own description) |
| Primary language | JavaScript |
| Archived | No — abandoned in practice, not archived on GitHub |
| Last push (`pushed_at`) | Years before this analysis (approximated from REVENANT's analysis; the repo has had no meaningful activity for a long time) |

## Verdict

| Field | Value |
|---|---|
| Resurrection Score | **15 / 100** |
| Classification | **Fossilised** |
| Dependencies analysed | 20+ declared dependencies |

A score this low means: long-idle, built on a deprecated foundation, and
not expected to install-and-run cleanly on a modern runtime without work.

## Cause of death

1. **The project itself is deprecated.** The `request` library was
   formally deprecated by its maintainers, and the npm registry marks
   the `request` package as deprecated. The repo is the corpse and the
   cause at the same time.
2. **Deprecated dependencies inside it.** The dependency tree includes
   packages the registry flags as deprecated — confirmed example:
   `har-validator`.
3. **Long idle time.** No meaningful pushes for years, so runtime,
   tooling, and transitive dependencies have all moved on without it.

## Deprecated findings (registry-flagged)

| Dependency | Deprecated | Note |
|---|---|---|
| `request` | Yes | The project itself; deprecated by its maintainers and flagged deprecated in the npm registry |
| `har-validator` | Yes | Flagged deprecated in the npm registry; used for HAR validation inside `request` |

Other declared dependencies (20+ analysed in total) were checked against
the registry as part of the same analysis; the two rows above are the
confirmed deprecated findings this example is built on.

## What this certificate does not claim

- It does not claim an exact `pushed_at` timestamp — the timing above is
  approximated from REVENANT's analysis for this example.
- It does not recommend fixes. Turning these facts into a plan is the
  Necromancer's job — see `resurrection-plan.md` in this folder.
- It does not claim any test was run against the repo. Verification is
  the Watcher's job, after a revival is attempted.
