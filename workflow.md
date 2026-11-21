# REVENANT Crew — Workflow

The crew is a strict pipeline. Each agent consumes exactly the artefact
the previous agent produced, and produces exactly one artefact (or pack)
for the next. No agent skips a step, and no agent does another agent's job.

```text
User input (owner/repo)
        |
        v
[1] Gravedigger  ->  death-certificate-data.json (+ death-certificate.md)
        |
        v
[2] Necromancer  ->  .kiro/specs/<repo>-resurrection/ (requirements.md,
        |             design.md, tasks.md)
        v
[3] Revival work (developer or Kiro, following tasks.md in order)
        |
        v
[4] Watcher      ->  test files + watcher-report.md
        |
        v
[5] Verdict: resurrected only if watcher-report.md shows tests that
    actually ran and passed. Otherwise loop back to [2] with the
    report's failures as input for the next smallest task.
```

## Step 1 — Gravedigger produces the Death Certificate

**Input:** `owner/repo` or a GitHub URL, supplied by the user.

**Artefact passed on:** `death-certificate-data.json`
(human-readable twin: `death-certificate.md`)

### Schema — `death-certificate-data.json`

| Field | Type | Source | Notes |
|---|---|---|---|
| `repo` | string | User input / GitHub | Full `owner/repo` name |
| `url` | string | GitHub API | Canonical repo URL |
| `description` | string \| null | GitHub API | `null` if missing |
| `language` | string \| null | GitHub API | Primary language |
| `stars` | number \| null | GitHub API | Never estimated |
| `openIssues` | number \| null | GitHub API | Never estimated |
| `archived` | boolean \| null | GitHub API | `null` if unfetchable |
| `pushedAt` | string \| null | GitHub API | ISO 8601 `pushed_at` value as returned |
| `fetchedAt` | string | Gravedigger | ISO 8601 time this data was fetched |
| `dependencies` | array | Manifest + registry | One entry per declared dependency, see below |
| `missing` | array of strings | Gravedigger | Names of fields that could not be fetched, each with a reason |

### Schema — each entry of `dependencies[]`

| Field | Type | Source | Notes |
|---|---|---|---|
| `name` | string | Manifest | As declared in `package.json` / equivalent |
| `declaredRange` | string | Manifest | Version range as declared |
| `latestVersion` | string \| null | Registry (npm/PyPI) | `null` if unresolvable |
| `deprecated` | boolean \| null | Registry | Only `true` when the registry itself flags it |
| `deprecationMessage` | string \| null | Registry | Quoted verbatim where the registry provides one |

## Step 2 — Necromancer turns the certificate into a spec pack

**Input:** `death-certificate-data.json` (Step 1 artefact) + the target
repo's README.

**Artefact passed on:** the folder `.kiro/specs/<repo>-resurrection/`
containing the three files below.

### Schema — `.kiro/specs/<repo>-resurrection/requirements.md`

| Section | Contents |
|---|---|
| Alive definition | The repo-specific meaning of "alive", taken from the README's documented purpose |
| Requirements list | Numbered, each with an acceptance check (e.g. "install command exits 0", "repo's own test suite runs", "README's main example executes") |
| Out of scope | Anything explicitly not part of this resurrection |

### Schema — `.kiro/specs/<repo>-resurrection/design.md`

| Column / section | Contents |
|---|---|
| Dead piece | Each deprecated/dead item named in the Death Certificate — no omissions |
| Disposition | One of: `replace`, `pin-and-isolate`, `keep (reason stated)` |
| Replacement | The smallest viable modern replacement, where disposition is `replace` |
| Rationale | Why this is the smallest change that restores the function |

### Schema — `.kiro/specs/<repo>-resurrection/tasks.md`

| Field per task | Contents |
|---|---|
| Order | Sequential number; tasks are executed strictly in order |
| Change | The single, smallest change this task makes |
| Verification | The exact check that proves the task worked |
| Stop condition | What to do if verification fails (stop; feed the failure to Watcher / back to Necromancer — do not proceed to the next task) |

Rule for the whole pack: Task 1 is the smallest change that makes the
repo install and run at all. No task may be a big-bang rewrite.

## Step 3 — Revival work

**Input:** `.kiro/specs/<repo>-resurrection/tasks.md` (Step 2 artefact).

Performed by the developer, or by Kiro executing the spec. Tasks are done
strictly in order; a task whose verification fails stops the pipeline
until the failure is understood.

**Artefact passed on:** the code change (diff / working tree) plus a note
of which tasks were completed.

## Step 4 — Watcher writes and runs property tests

**Input:** the full `.kiro/specs/<repo>-resurrection/` pack (Step 2
artefact) + the code change (Step 3 artefact).

**Artefacts produced:**

| Artefact | Contents |
|---|---|
| Test files (e.g. `tests/resurrection.property.test.js`) | Unit tests for example-sized requirements; property-based tests (fast-check or equivalent) for every stated invariant |
| `watcher-report.md` | The verification record, schema below |

### Schema — `watcher-report.md`

| Field | Contents |
|---|---|
| `command` | The exact test command that was run |
| `ranAt` | When it was run |
| `summary` | Counts: passed / failed / not run |
| `results[]` | One row per requirement or invariant: name, status (`passed` \| `failed` \| `not run`), evidence or reason |
| `failures[]` | For each failure: the invariant/test name and the smallest reproduction found |
| `notRun[]` | For each test that did not run: the reason. "Not run" is a final status — it is never rewritten as "passed" |

## Step 5 — Verdict and loop

- **Resurrected:** `watcher-report.md` shows every requirement `passed`
  with a real command and output behind it.
- **Not yet:** any `failed` or `not run` row means the revival is
  unproven. The report's `failures[]` go back to the Necromancer, which
  adds the next smallest task to `tasks.md`, and the pipeline repeats
  from Step 3.

## Worked example

`examples/request-request/` contains a filled Step 1 output
(`death-certificate.md` for `request/request`: score 15/100, Fossilised,
20+ dependencies, `request` and `har-validator` deprecated) and its
Step 2 summary (`resurrection-plan.md`, smallest-first).
