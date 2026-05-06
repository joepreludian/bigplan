# Master Index Template

Copy the template below to `docs/bigspec/<topic>/INDEX.md` and fill it in. Replace anything in `<angle brackets>`.

---

```markdown
# <Big Delivery Name> — bigspec index

> **Orchestrated by the bigspec skill.** This file is the source of truth for what's in scope, the slice order, and what's been implemented. Each slice starts as a TOPLAN starter prompt, becomes a brainstormed design doc (BRAINSTORMED), gets a plan via `superpowers:writing-plans` (AVAILABLE), and ships as code (IMPLEMENTED).

## Context

**User-facing outcome:** <one or two sentences>

**Out of scope:** <what we are explicitly not doing>

**Constraints:** <existing schemas, frameworks, deploy/auth model, deadlines>

**Definition of done:** <what "shipped" means for the whole delivery>

## Slices

Filename format: `NN-{TOPLAN|BRAINSTORMED|AVAILABLE|IMPLEMENTED}-{BE|FE|DEVOPS}-YYYY-MM-DD-<kebab-name>.md`. Backend slices ship before frontend slices unless a DEVOPS slice has to come first to unblock them. **No slice begins implementation until every row below is at status `available` or further.**

| #  | Tier   | Slice          | Spec file | Plan file | Status |
|----|--------|----------------|-----------|-----------|--------|
| 01 | BE     | <short name>   | [`01-TOPLAN-BE-YYYY-MM-DD-<name>.md`](./01-TOPLAN-BE-YYYY-MM-DD-<name>.md) | — | toplan |
| 02 | BE     | <short name>   | [`02-TOPLAN-BE-YYYY-MM-DD-<name>.md`](./02-TOPLAN-BE-YYYY-MM-DD-<name>.md) | — | toplan |
| 10 | FE     | <short name>   | [`10-TOPLAN-FE-YYYY-MM-DD-<name>.md`](./10-TOPLAN-FE-YYYY-MM-DD-<name>.md) | — | toplan |
| 11 | FE     | <short name>   | [`11-TOPLAN-FE-YYYY-MM-DD-<name>.md`](./11-TOPLAN-FE-YYYY-MM-DD-<name>.md) | — | toplan |
| 20 | DEVOPS | <short name>   | [`20-TOPLAN-DEVOPS-YYYY-MM-DD-<name>.md`](./20-TOPLAN-DEVOPS-YYYY-MM-DD-<name>.md) | — | toplan |

**Status values (filename token in parens):**
`toplan` (`TOPLAN`) → `brainstormed` (`BRAINSTORMED`) → `available` (`AVAILABLE`) → `implementing` → `done` (`IMPLEMENTED`)

## Phase progress

```
Spec phase:           <X> of <Y> specs brainstormed
Plan phase:           <X> of <Y> plans written
Implementation phase: <X> of <Y> slices shipped
```

(Update these lines — and emit them in the conversation — every time a slice's status flips. Each phase has its own gate: do not begin a phase until the previous phase reads `Y of Y`.)

## Open questions

- <question that came up during decomposition and was deferred to a specific slice>
- <…>

## Change log

- <YYYY-MM-DD>: index created with N TOPLAN slices
- <YYYY-MM-DD>: slice 03 split into 03a / 03b after brainstorming revealed two responsibilities
- <YYYY-MM-DD>: slice 01 brainstormed; status TOPLAN → BRAINSTORMED
- <YYYY-MM-DD>: spec phase complete (Y of Y brainstormed)
- <YYYY-MM-DD>: slice 01 plan written; status BRAINSTORMED → AVAILABLE
- <YYYY-MM-DD>: plan phase complete (Y of Y plans written; ran in parallel)
- <YYYY-MM-DD>: slice 01 implemented; status AVAILABLE → IMPLEMENTED
```

## Notes for the orchestrator

- Keep the table tight — one row per slice, one line each. If you need to explain a slice in more than a phrase, that explanation belongs in the slice's own spec/TOPLAN, not here.
- The **Spec file** column always links to the slice file as it currently exists on disk. Update the filename portion of the link every time the file is renamed (`TOPLAN` → `BRAINSTORMED` → `AVAILABLE` → `IMPLEMENTED`). Stale links break.
- The **Plan file** column starts as `—`, then becomes a relative link to `docs/superpowers/plans/YYYY-MM-DD-NN-<slice-name>.md` once the plan is written in Step 5b.
- Update the **Phase progress** block on every flip. The user uses these counters to gauge how far each phase has progressed.
- Update the **Change log** every time a slice changes status or the slice list is restructured. Future-you will want to know why slice 03 got split, or why the plan phase took 2 days.
- Commit the index alongside whichever event caused the change. The index and the slice files should never disagree.
