# Sub-spec conventions

Detail-level guidance for slice files. The main flow is in `SKILL.md` — read this only when an edge case comes up.

## Filename anatomy

```
NN-{TOPLAN|BRAINSTORMED|AVAILABLE|IMPLEMENTED}-{BE|FE|DEVOPS}-YYYY-MM-DD-<short-kebab-name>.md
 |                |                                  |              |              |
 |                |                                  |              |              └─ ~3-word, distinguishable from siblings
 |                |                                  |              └─ date the slice was decomposed; never changes after creation
 |                |                                  └─ tier; matches the numeric range
 |                └─ lifecycle status; TOPLAN → BRAINSTORMED → AVAILABLE → IMPLEMENTED, advanced by renaming in place
 └─ two-digit, sortable, defines execution order
```

## Tier and numbering convention

| Range  | Tier   | Typical contents                                              |
|--------|--------|---------------------------------------------------------------|
| 01–09  | BE     | data model, migrations, API, jobs, auth                       |
| 10–19  | FE     | pages, components, state, API client, routing                 |
| 20+    | DEVOPS | CI, infra, deploy config, observability, flag rollout cleanup |

The tier identifier in the filename must match the numeric range — e.g. a slice numbered `12-` should be tagged `FE`, not `BE`. If you find yourself wanting to mismatch them, that's usually a sign the slice was misnumbered or the tier categorization is off.

## Status lifecycle

A slice file has exactly four filename states, advanced by renaming in place:

- **`TOPLAN`** — created during Step 3b; contains a starter prompt for `superpowers:brainstorming`. Every slice is born here.
- **`BRAINSTORMED`** — design doc written via brainstorming. File has been renamed `TOPLAN` → `BRAINSTORMED`. No implementation plan yet.
- **`AVAILABLE`** — implementation plan written via `superpowers:writing-plans`; plan file path recorded in the index. File renamed `BRAINSTORMED` → `AVAILABLE`. Slice is ready to implement.
- **`IMPLEMENTED`** — code is merged. File renamed `AVAILABLE` → `IMPLEMENTED`. Nothing else in the filename changes — same number, same tier, same date, same kebab-name.

Statuses only move forward. Don't rename a later state back to an earlier one — instead:

- If you want to re-brainstorm a `BRAINSTORMED` slice, edit the file in place.
- If you want to redo a plan, edit the plan file in place. The slice stays `AVAILABLE`.
- If implementation got reverted, write a new slice (next available number) describing the re-do work, and add a change log note pointing at the original. Don't roll the `IMPLEMENTED` file back.

### Why each phase has a gate

The skill enforces two gates: no plan-writing until every slice is `BRAINSTORMED`, no implementation until every slice is `AVAILABLE`. This isn't bureaucratic — it's the protection against the most expensive class of bug a big delivery has: a decomposition that looked fine on paper but breaks at slice boundaries.

- The **brainstorm gate** catches *contract* mismatches between slices (slice 04 needs a field that slice 01's design didn't include).
- The **plan gate** catches *implementation-shape* mismatches (slice 02 and 03 both want to put their controller logic in the same file with incompatible structures).

Catching either of these at the spec or plan stage costs minutes; catching them after slice 01 ships costs days and a migration.

## When you need to insert a slice mid-stream

You'll discover during brainstorming that a slice was missing. Two options:

1. **Letter suffix** (preferred when no slice in the same tier has shipped yet): insert `02a-` between `02-` and `03-`. Cheap, no cascading changes. The new slice is created at status `TOPLAN` like any other.
2. **Renumber** (only when needed for clarity, and only when no `IMPLEMENTED` files of that number exist): renumber later slices upward. Update the index's slice table and change log.

**Never renumber an `IMPLEMENTED` file.** It's a historical artifact — its number reflects the order things shipped in, not the current ideal ordering. Renumbering it would also break references in plan docs, commits, and PR titles.

When you insert a slice, the spec phase progress denominator grows: a `5 of 7` becomes `5 of 8`. That's correct — the index should reflect the truth that decomposition grew, not pretend it stayed the same.

## When you need to split a slice

If brainstorming reveals slice 03 actually has two distinct responsibilities, split it:

- Rename the original to `03a-{current status}-{tier}-YYYY-MM-DD-<first-responsibility>.md` (preserve whichever status it was in — TOPLAN, BRAINSTORMED, or AVAILABLE)
- Create `03b-TOPLAN-{tier}-YYYY-MM-DD-<second-responsibility>.md` (new TOPLAN file with a fresh starter prompt for the second responsibility)
- Update the index table to show two rows
- Add a change log entry explaining the split

If the original `03-` was already `IMPLEMENTED`, don't touch it. Just add new slices at the next available number.

## Slice file front matter

A slice file at any status (TOPLAN, BRAINSTORMED, AVAILABLE, IMPLEMENTED) should keep a small header pointing back at the index, so the file is readable on its own:

```markdown
> Part of the bigspec at [`./INDEX.md`](./INDEX.md). Slice **NN** of **<Y>** (tier: BE | FE | DEVOPS). Status: <TOPLAN | BRAINSTORMED | AVAILABLE | IMPLEMENTED>.
```

The TOPLAN template (`references/toplan-template.md`) ships with a richer version of this header. When the file's status changes, keep the header (just update the status line) and let it sit above the design doc.

## What goes in the slice spec vs. the index

| Lives in the slice spec       | Lives in the index                  |
|-------------------------------|-------------------------------------|
| Architecture for this slice   | Architecture for the whole delivery |
| Data model changes this slice introduces | Constraints on the data model overall |
| API contract this slice exposes/consumes | Definition of done for the delivery |
| Test plan for this slice      | Slice list, status, change log      |
| Open questions specific to this slice | Open questions that don't belong to any single slice |

Rule of thumb: if removing slice N would also remove the information, it lives in slice N's spec. Otherwise it lives in the index.
