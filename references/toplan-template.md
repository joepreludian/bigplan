# TOPLAN file template

A TOPLAN file is a starter prompt for `superpowers:brainstorming`. When the spec phase reaches this slice, brainstorming reads this file and produces the actual design doc — replacing the content in place — after which the file is renamed `TOPLAN` → `BRAINSTORMED`. The plan phase later renames it again to `AVAILABLE` once a plan has been written via `superpowers:writing-plans`.

Write one TOPLAN file per slice during Step 3b, **all in the same pass**. Doing them in one sitting keeps the dependency graph between slices consistent.

## Filename

```
docs/bigspec/<topic>/NN-TOPLAN-{BE|FE|DEVOPS}-YYYY-MM-DD-<kebab-name>.md
```

`YYYY-MM-DD` is today's date — the date the slice was decomposed, which equals the date the TOPLAN file is created. It does not change when the file is renamed to `AVAILABLE` or `IMPLEMENTED` later.

## Template

```markdown
> **TOPLAN slice — starter prompt for `superpowers:brainstorming`.**
> Part of the bigspec at [`./INDEX.md`](./INDEX.md). Slice **NN** of **<Y>** (tier: BE | FE | DEVOPS). Status: TOPLAN.
> When brainstorming completes, this file's content will be replaced with the full design doc and the file will be renamed `TOPLAN` → `BRAINSTORMED`. A later plan-writing pass renames it again to `AVAILABLE`.

# <Slice short name>

## Scope

<One paragraph: what this slice produces. Concrete and bounded — name the files, endpoints, components, or jobs that will be touched. If you can't name them yet, the slice isn't sharp enough; tighten it before continuing.>

## Out of scope

- <Thing this slice does NOT touch — usually owned by an adjacent slice. Name the slice that owns it.>
- <…>

## Inputs / dependencies

This slice builds on contracts produced by:

- **Slice <NN>** — <what we depend on; e.g. "the `boards` table created in 01">
- **Slice <NN>** — <…>

If this is a leaf-level backend slice with no upstream dependencies, write "none — this slice is at the root of the dependency graph."

## Outputs / contracts

This slice produces these contracts that later slices will consume:

- <e.g. "POST /api/boards endpoint accepting {title, description} and returning {id, slug}">
- <e.g. "the `boards.slug` column, unique, ≤ 64 chars">
- <e.g. "a server-side event `board.created` payload of {board_id, owner_id}">

Be specific. Vague contracts here ("an endpoint to create boards") force every downstream slice's brainstorming to re-derive the same details.

## Open questions for brainstorming

- <Specific question this slice's brainstorming session needs to resolve. e.g. "Should slug be user-supplied or auto-generated from title?">
- <…>

## Success criterion

<One sentence: how we'll know this slice's spec is good. e.g. "A backend engineer can read the resulting spec and have no ambiguity about the data model, endpoint shape, or auth rules.">

---

## Brainstorming instructions

When invoked on this slice, `superpowers:brainstorming` should:

1. Read `./INDEX.md` for the shared brief (outcome, out-of-scope, constraints, definition-of-done).
2. Read this TOPLAN file for the slice's specific scope.
3. Resolve every open question above with the user.
4. Produce a normal design doc (architecture, components, data flow, error handling, testing) **in this same file**, replacing this content from the `# <Slice short name>` heading downward. Keep the header pointer (just update the `Status:` line to `BRAINSTORMED`).
5. After the user approves the design, signal back so bigspec can rename the file `TOPLAN` → `BRAINSTORMED` and update the index. Plan-writing happens in a separate phase.
```

## What makes a good TOPLAN

- **Specific contracts.** The whole point of writing TOPLAN files for every slice up front is to lock the contracts so brainstorming doesn't drift. If your `Outputs / contracts` section is vague, brainstorming will fill in the blanks differently for each slice and you'll have to reconcile later.
- **Named dependencies.** Reference upstream slices by number, not by code that doesn't exist yet. "Depends on slice 02's `POST /api/boards` endpoint" — not "depends on the boards endpoint we'll build."
- **Real open questions.** Don't pad the open-questions list. Brainstorming will ask its own questions; you only need to surface things you and the user already noticed but couldn't resolve at decomposition time.
