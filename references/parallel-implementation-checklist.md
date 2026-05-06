# Parallel implementation safety checklist

Run this when the user has asked for parallel implementation in Step 6. The output is a partition of the `AVAILABLE` slices into three buckets: **safe-to-parallelize**, **must-be-sequential**, and **borderline**.

The decisive question is the same for every slice: **what files would this plan write to?** If two slices answer with overlapping file sets, they cannot run in parallel.

## Step 1 — Extract each plan's file set

For every `AVAILABLE` slice, open its plan file at `docs/superpowers/plans/<...>.md`. A well-formed superpowers plan lists files in each task's "Files:" block — `Create:`, `Modify:`, `Test:`. Aggregate them into a single set per slice:

```
slice 01 → { app/models/board.rb, db/migrate/2026..._create_boards.rb, spec/models/board_spec.rb }
slice 02 → { app/controllers/boards_controller.rb, config/routes.rb, spec/requests/boards_spec.rb }
slice 03 → { app/models/board.rb, app/services/share_token.rb, spec/services/share_token_spec.rb }
...
```

If a plan lists a file with a glob or vague descriptor ("the relevant route file", "all model specs"), flag the slice as **borderline** — see Step 4.

## Step 2 — Build the conflict graph

Two slices conflict if their file sets intersect. Build a graph: nodes are slices, an edge connects every pair that shares at least one file.

```
01 ── 03   (both touch app/models/board.rb)
02       (no edges)
04 ── 05   (both touch config/routes.rb)
```

In the example above, `02` is isolated; `01 / 03` and `04 / 05` are connected.

## Step 3 — Partition into buckets

- **Safe-to-parallelize** — slices that have NO edges in the conflict graph. They can all run concurrently in a single batch.
- **Must-be-sequential** — slices with at least one edge. These must run one at a time, in slice number order, after the safe batch finishes (or interleaved if the user prefers).
- **Borderline** — slices flagged in Step 1 because their file set couldn't be enumerated. Default to treating them as must-be-sequential; ask the user if uncertain.

Common shared files that almost always cause conflicts and should be checked specifically:

| Shared file                          | Why                                          |
|--------------------------------------|----------------------------------------------|
| `package.json` / `requirements.txt` / `Gemfile` / `pyproject.toml` | dependency adds collide on the lockfile |
| `schema.rb` / `schema.prisma` / `migrations/` directory | parallel migrations race on version numbers |
| Top-level router (`app/router.tsx`, `config/routes.rb`, `urls.py`) | route registrations collide |
| Shared component index files (`index.ts`, `mod.rs`)   | barrel exports collide |
| CI config (`.github/workflows/*.yml`)                 | new jobs collide on cursor position |
| `tsconfig.json`, `eslint.config.js`, `.gitignore`     | small but high-traffic |

If a slice's plan touches any of these AND another slice's plan touches the same one, conflict. Mark must-be-sequential.

## Step 4 — Token / cost sanity check

Parallel implementation bursts tokens — each subagent runs concurrently, multiplying the per-second token rate. Estimate roughly:

```
parallel_burst ≈ N_safe × (avg plan size + avg implementation budget per task)
```

If `N_safe` is large (say, 8+), warn the user that the burst will be heavy. Offer to cap the parallel batch size (e.g. 4 at a time) to keep usage bounded. The total token spend over the whole delivery is similar either way; the difference is whether it lands all at once or spread out.

## Step 5 — Present the audit to the user

Show the user, before launching any implementation:

```
Parallel-safety audit:

  Safe to run in parallel (3 slices):
    01-board-data-model
    04-board-digest-job
    20-flag-rollout-config

  Must be sequential — file conflicts (3 slices):
    02-board-crud-api  ╮
    03-board-share-tokens ├─ all three modify app/controllers/boards_controller.rb
    05-something       ╯

  Borderline (1 slice):
    11-board-editor-page — plan says "modify the appropriate page component", file set unclear

Proceed?
```

If the user says go, dispatch the safe batch in parallel, then walk through the must-be-sequential and borderline groups one slice at a time.

## What this checklist deliberately does not do

- It does NOT try to detect semantic conflicts that the file set wouldn't reveal (e.g. two slices that don't touch the same file but both add a column with the same name to different tables in different ways). That's beyond a static file-overlap check; surface it during brainstorming or plan-writing instead.
- It does NOT try to merge changes after the fact. The whole point is to avoid producing changes that need merging.
- It does NOT optimize for "minimum sequential time" — it optimizes for safety. If the user wants tighter scheduling, they can decide manually after seeing the audit.
