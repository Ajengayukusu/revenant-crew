# Resurrection Plan — request/request

> Example Necromancer output, derived from `death-certificate.md` in this
> folder (Resurrection Score 15/100, Fossilised; 20+ dependencies;
> `request` itself and `har-validator` deprecated).
>
> In a real run this plan is written as a spec pack at
> `.kiro/specs/request-resurrection/` (`requirements.md`, `design.md`,
> `tasks.md`). This file is the human-readable summary of that pack's
> task list. Every step is smallest-first and verifiable; no step is a
> big-bang rewrite.

## What "alive" means for this repo

- It installs cleanly on a current Node.js LTS.
- Its own existing test suite runs and passes.
- A basic documented use — making an HTTP request to a URL and receiving
  the response — works end to end.

## Smallest-first step list

1. **Freeze the baseline.**
   Clone the repo untouched. Record exactly how installation and the
   existing test suite fail today (commands + output).
   *Verify:* a written baseline failure record exists. Nothing has been
   changed yet.

2. **Pin the last-known runtime.**
   Set the Node.js version the code was last maintained against (via
   `.nvmrc` / `engines`) and get `npm install` to complete under it,
   changing nothing else.
   *Verify:* install exits 0 under the pinned runtime.

3. **Make the existing test suite run at all.**
   Fix only what blocks the test runner from starting (runner config,
   missing dev tooling) — not library code.
   *Verify:* the test command executes and reports results, even if
   some tests fail.

4. **Replace the smallest deprecated piece: `har-validator` call sites.**
   HAR validation is a narrow, well-bounded feature. Swap the deprecated
   `har-validator` usage for a maintained validator (or a small local
   schema check) at those call sites only.
   *Verify:* existing tests still pass; no deprecated `har-validator`
   remains in the dependency tree.

5. **Isolate `request`'s public surface.**
   List the exact public API surface the tests and README exercise.
   This defines the contract any replacement must honour, before any
   replacement is chosen.
   *Verify:* a written API-surface list exists and every entry maps to
   a test or a README example.

6. **Migrate one call path from `request` internals to a maintained
   HTTP client.**
   Pick the single simplest request path (a plain GET with no advanced
   options) and re-implement it on a maintained client (for example
   Node's built-in `fetch` / undici), behind the same public surface
   from Step 5.
   *Verify:* the tests covering that path pass; all other paths are
   untouched.

7. **Repeat Step 6, one call path at a time.**
   Each remaining path (redirects, streaming, auth helpers, and so on)
   is its own step, in order of increasing complexity, tests green
   after each. No step migrates two paths at once.
   *Verify:* full existing test suite passes after each individual step.

8. **Unpin and modernise the runtime.**
   Only once the suite is green on the pinned runtime, move to a current
   Node.js LTS and fix what breaks, one breakage per change.
   *Verify:* install exits 0 and the full test suite passes on the
   current LTS.

9. **Hand over to the Watcher.**
   Watcher writes property-based tests for the invariants this revival
   must hold (for example: for any valid URL input, a GET returns a
   response object with a numeric status; deprecated dependencies never
   re-enter the tree) and runs the whole suite.
   *Verify:* `watcher-report.md` exists, naming the command that ran
   and its real results. Steps marked "not run" there mean the
   resurrection is not yet proven — they are never read as "passed".

## Explicitly not in this plan

- No rewrite of the library in a single step.
- No new features. Resurrection restores documented behaviour first;
  features are a later, separate spec.
- No claim that any step above has been executed for this example.
  This is the plan the Necromancer produces; execution and Watcher
  verification happen after it.
