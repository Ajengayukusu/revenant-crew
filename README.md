# REVENANT Crew — The Repo Resurrection Crew

A Kiro Crew app made of three specialised agents that work in sequence to
bring a dead GitHub repository back to life: one agent digs up the facts,
one writes the resurrection plan, and one proves — with tests that actually
ran — whether the revival worked.

This is the companion Crew repo for **REVENANT**, a Kiroween 2026 project
(Resurrection category). The main REVENANT web app performs the same
diagnosis deterministically in the browser; this Crew packages the three
agents behind it so a developer can run the whole resurrection workflow
inside Kiro.

## The crew

| Agent | File | One job |
|---|---|---|
| **Gravedigger** | [`agents/gravedigger.md`](agents/gravedigger.md) | Scanner. Collects facts about a repo — **facts only, no opinions, no score of its own**. |
| **Necromancer** | [`agents/necromancer.md`](agents/necromancer.md) | Resurrection spec planner. Turns the facts into a Kiro spec pack, **smallest change first, no big-bang rewrites**. |
| **Watcher** | [`agents/watcher.md`](agents/watcher.md) | Property-based tester. Writes and runs the tests that guard a revival, and **never upgrades "not run" to "passed"**. |

Full pipeline, artefact names, and schemas: [`workflow.md`](workflow.md).

## How a user runs the crew on a dead repo

You need: a public GitHub repository you believe is dead or dying, and Kiro
with these agent definitions available.

1. **Pick the corpse.** Note the repo as `owner/repo` or a full GitHub URL.
   Example used throughout this repo: `request/request`.
2. **Run Gravedigger.** Give it the repo identifier (sample prompt in
   `agents/gravedigger.md`). It fetches repo metadata from GitHub and
   dependency data from the package registry (npm/PyPI), and writes a
   **Death Certificate data file** — `death-certificate-data.json`, plus a
   human-readable `death-certificate.md`. Any field it cannot fetch is
   recorded as missing, never estimated.
3. **Run Necromancer.** Give it the Death Certificate data file and the
   target repo's README (sample prompt in `agents/necromancer.md`). It
   writes a Kiro spec pack at `.kiro/specs/<repo>-resurrection/` containing
   `requirements.md` (what "alive" means for this repo), `design.md`
   (which dead pieces get replaced with what), and `tasks.md` (ordered,
   smallest-first, each task verifiable).
4. **Do the revival.** Work through `tasks.md` in order — yourself, or by
   letting Kiro execute the spec. Stop at the first task that makes the
   repo install and run at all before attempting any larger change.
5. **Run Watcher.** Give it the spec pack and the resulting code change
   (sample prompt in `agents/watcher.md`). It writes property-based tests
   for the invariants the revival must hold, runs them, and writes
   `watcher-report.md` naming the exact command it ran, its output, and
   any invariant that failed, with the smallest reproduction.
6. **Read the verdict honestly.** A repo is "resurrected" only when
   Watcher's report shows tests that actually ran and passed. A report
   that says "not run" means exactly that — the revival is unproven.

## Example: `request/request`

Input given to Gravedigger:

```text
owner/repo: request/request
```

Excerpt of the Death Certificate Gravedigger produced (full file:
[`examples/request-request/death-certificate.md`](examples/request-request/death-certificate.md)):

```text
Repository:        request/request
Resurrection Score: 15 / 100 — Fossilised
Dependencies:      20+ declared dependencies analysed
Deprecated:        request (the project itself is deprecated),
                   har-validator
```

Excerpt of the plan Necromancer produced from that certificate (full file:
[`examples/request-request/resurrection-plan.md`](examples/request-request/resurrection-plan.md)):

```text
1. Freeze the baseline: record the current install/test failure exactly.
2. Pin the runtime the code was last known to work on; make it install.
3. Replace the smallest deprecated piece first (har-validator call sites).
4. Only then plan the request -> maintained HTTP client migration,
   one call site at a time, tests green after each step.
```

Excerpt of the standard Watcher holds every revival to:

```text
- A resurrection score is always between 0 and 100, for any input.
- Adding a deprecated dependency never raises the score.
- A report may say "passed" only for tests it ran, with the command
  and output recorded. "Not run" is never rewritten as "passed".
```

## Status — honest note

- The three agent definitions in `agents/` are **ready**: each has a role,
  inputs, outputs, hard rules, and a sample Kiro Crew prompt.
- The main REVENANT app is **live** and performs the Gravedigger-style
  analysis deterministically in the browser (GitHub API + package
  registries), so the diagnosis works with no agent and no API key.
- The crew definitions have **not yet been run inside Kiro** as a packaged
  Crew app. Packaging these definitions as a one-click Kiro Crew app and
  doing the first end-to-end Crew run is the next step. Nothing in this
  repo claims otherwise.
- The `request/request` example under `examples/` is real output data from
  REVENANT's analysis of that repo; the `pushed_at` timing there is
  approximated from that analysis rather than re-fetched live for this
  document.

## Repository layout

```text
README.md                          <- this file
LICENSE                            <- MIT, (c) 2026 Ajeng Ayu Kusuma Wardani
workflow.md                        <- the numbered crew pipeline + schemas
agents/
  gravedigger.md                   <- scanner agent definition
  necromancer.md                   <- spec planner agent definition
  watcher.md                       <- PBT tester agent definition
examples/
  request-request/
    death-certificate.md           <- filled example: Gravedigger output
    resurrection-plan.md           <- filled example: Necromancer output
```

## Licence

MIT — see [`LICENSE`](LICENSE). Copyright (c) 2026 Ajeng Ayu Kusuma Wardani.

Built for Kiroween 2026 as the companion to REVENANT, by Ajeng Ayu Kusuma
Wardani.
