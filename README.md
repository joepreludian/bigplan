# bigspec

> **A Claude Code skill that decomposes large webapp deliveries into ordered, sequenced sub-specs — backend slices first, then frontend, with hard phase gates between brainstorming, planning, and implementation.**

Big features die when you spec them as one giant document. `bigspec` is a skill for [Claude Code](https://claude.com/claude-code) that recognizes when a user request is "too big for one spec" and walks the user through a structured decomposition: a master `INDEX.md`, one starter prompt per slice, then a phased flow where every slice is brainstormed → planned → implemented in lockstep across the whole delivery.

It pairs with the [superpowers](https://github.com/anthropics/claude-plugins) skills (`brainstorming`, `writing-plans`) — bigspec is the orchestrator that decides what the slices are and what order they ship in; brainstorming and writing-plans do the per-slice authoring.

---

## Install

### Option A — Direct download (`.skill` file)

The repo ships a packaged `bigspec.skill` zip. To install:

```bash
# from the repo root, or wherever you have bigspec.skill
mkdir -p ~/.claude/skills
unzip bigspec.skill -d ~/.claude/skills/
ls ~/.claude/skills/bigspec/    # SKILL.md + references/
```

Restart Claude Code (or open a new session) and the skill is available. Verify by asking Claude something that should trigger it — see the example prompts below.

To update later: drop a new `bigspec.skill` over the existing dir, or `rm -rf ~/.claude/skills/bigspec && unzip bigspec.skill -d ~/.claude/skills/`.

### Option B — Git clone + symlink (recommended for hacking)

If you want to follow upstream changes or iterate on the skill yourself:

```bash
git clone https://github.com/joepreludian/bigplan ~/code/bigspec   # or wherever
ln -s ~/code/bigspec ~/.claude/skills/bigspec
```

Edits to `~/code/bigspec/SKILL.md` apply immediately on the next Claude Code session — no rebuild step. `git pull` to get updates.

### Verifying the install

In a Claude Code session, run:

```
/skills
```

You should see `bigspec` in the list. If not, check that `~/.claude/skills/bigspec/SKILL.md` exists and the YAML frontmatter is intact.

---

## When does the skill engage?

bigspec has a **hard size threshold**. It only engages when **all three** hold:

1. Work decomposes to **≥5 sub-specs**.
2. At least **2 backend slices** (data model, API, jobs, auth, etc.).
3. At least **2 frontend slices** (pages, components, state, etc.).

Below that bar — single endpoint, single page, pure refactor — the skill defers to `superpowers:brainstorming` directly. The threshold is strict; the skill explicitly refuses to pad slices to clear the bar.

**Triggers it on:**

- "I need a big plan for ...", "I want to build the whole flow for ..."
- A request that lists 5+ deliverables touching both backend and frontend
- Anything spanning data model + API + UI + jobs

**Doesn't trigger on:**

- "Add a `/healthz` endpoint"
- "Fix this bug"
- "Refactor this module"

---

## How to use it

### Example session

You write:

> *"I want to add project boards. Users can create boards, drag cards between columns, share read-only links, and get a weekly digest email of activity. Stack: rails + next.js + postgres."*

Claude (with bigspec loaded):

1. **Captures the brief** — asks about user-facing outcome, out-of-scope, constraints, definition of done. (One question at a time.)
2. **Proposes a slice list** — typically something like:
   - `01 BE board data model`
   - `02 BE board CRUD API`
   - `03 BE share-token API`
   - `04 BE digest email job`
   - `10 FE board list page`
   - `11 FE board editor with drag-and-drop`
   - `12 FE public share view`
3. **Asks you to confirm** the slice list before writing anything to disk.
4. **Writes**, in one atomic commit:
   - `docs/bigspec/project-boards/INDEX.md` — master index with phase counters
   - `docs/bigspec/project-boards/NN-TOPLAN-{BE|FE|DEVOPS}-YYYY-MM-DD-<kebab>.md` — one starter-prompt file per slice
5. **Stops** and tells you the spec-phase gate state: `Spec-phase gate: 0 of 7 — closed.`

From there, you advance through three phases, with a hard gate between each:

```
Spec phase   ─►  brainstorm each TOPLAN     →  rename to BRAINSTORMED
                 progress: "X of Y specs brainstormed"

         ── GATE ── (no plan-writing until X == Y) ──

Plan phase   ─►  superpowers:writing-plans   →  rename to AVAILABLE
                 (parallelizable on opt-in)
                 progress: "X of Y plans written"

         ── GATE ── (no implementation until X == Y) ──

Impl phase   ─►  build the slice              →  rename to IMPLEMENTED
                 default sequential; parallel only after a file-overlap audit
```

### Lifecycle vocabulary

Each slice file's status is encoded in its filename and advances by **renaming in place**:

```
NN-TOPLAN-{tier}-YYYY-MM-DD-<name>.md      # starter prompt; never brainstormed
NN-BRAINSTORMED-{tier}-YYYY-MM-DD-<name>.md # design doc written
NN-AVAILABLE-{tier}-YYYY-MM-DD-<name>.md    # plan written; ready to implement
NN-IMPLEMENTED-{tier}-YYYY-MM-DD-<name>.md  # merged
```

The numeric prefix preserves execution order; the status token gives you an immediate visual signal of what's left to ship. The date stays fixed at the slice's authoring date even after renames — it's a historical record, not a "last touched" timestamp.

### Tier numbering

| Range  | Tier   | Typical contents                                            |
|--------|--------|-------------------------------------------------------------|
| 01–09  | BE     | data model, migrations, API, jobs, auth                     |
| 10–19  | FE     | pages, components, state, API client, routing              |
| 20+    | DEVOPS | CI, infra, deploy config, observability, flag rollouts     |

Backend ships before frontend so frontend slices target a real, agreed API contract instead of an imagined one.

### Parallel safety

- **Plan phase** is safe to parallelize by default — plans only read each other's contracts; they don't share files. The skill asks before launching.
- **Implementation phase** is sequential by default. The skill runs a [file-overlap audit](references/parallel-implementation-checklist.md) when you opt into parallelism: any two slices that touch the same file are forced to run sequentially. Soft-merging is explicitly forbidden.

### Context discipline

Big deliveries can blow up Claude's context. The skill instructs the orchestrator to:

- Dispatch each brainstorm / plan / implementation as a **subagent** (fresh context per call).
- Or, if running inline, ask you to `/compact` between slices.
- Always commit to disk before clearing context — disk is the source of truth.
- Re-read only `INDEX.md` plus the immediate slice's files, never the whole tree.

---

## Benchmark results

The skill was developed iteratively and graded against 8 evaluation prompts covering trigger, decline, and mid-flow behaviors. Each eval was run twice — once with the bigspec skill loaded, once as a no-skill baseline — and graded against 5–14 programmatic assertions per case.

### Aggregate (iteration 2)

| Metric    | With skill         | Baseline           | Delta      |
|-----------|--------------------|--------------------|------------|
| Pass rate | **100%** ± 0%      | 47% ± 20%          | **+53 pts**|
| Time      | 264.3s ± 233.5s    | 94.1s ± 34.4s      | +170.2s    |
| Tokens    | 49,072 ± 19,323    | 22,545 ± 3,333     | +26,527    |

### Per-eval pass rate

| # | Eval                                       | With skill    | Baseline      |
|---|--------------------------------------------|---------------|---------------|
| 1 | Explicit "big plan" trigger                | 11/11 (100%)  | 3/11 (27%)    |
| 2 | Implicit large scope (no trigger words)    | 10/10 (100%)  | 3/10 (30%)    |
| 3 | Single deliverable — should NOT trigger    | 5/5 (100%)    | 4/5 (80%)     |
| 4 | Borderline 4 slices — should NOT trigger   | 6/6 (100%)    | 3/6 (50%)     |
| 5 | Phase-flow + gate vocabulary               | 14/14 (100%)  | 3/14 (21%)    |
| 6 | Mid-flow plan-phase parallelism prompt     | 8/8 (100%)    | 4/8 (50%)     |
| 7 | Implementation-phase file-overlap audit    | 8/8 (100%)    | 5/8 (63%)     |
| 8 | Context hygiene with 9 large slices        | 7/7 (100%)    | 4/7 (57%)     |

### What the metrics mean

- **+53 pts pass rate** says the skill is teaching Claude behaviors it would not reach on its own — not just lightly nudging existing behavior. (A "skill that doesn't help" typically shows <10 pts delta.)
- **+170s and +27k tokens** is the cost of reading the skill + producing the index/TOPLAN files. The cost only pays off when the decomposition is real; that's why the threshold is strict.
- **Baseline behavior** uniformly reverted to "build me the thing" mode — single mega-design-docs, or even working code — which is the wrong shape for a multi-slice delivery. The skill's job is to make Claude stop and decompose first.

### Caveat

These are N=1 runs per cell. The signal is strong enough that noise won't change the verdict, but a confidence pass at N=3 has not been done yet.

---

## Repo layout

```
bigspec/
├── SKILL.md                                    # the skill itself
├── README.md                                   # this file
├── bigspec.skill                               # packaged zip (build artifact)
└── references/
    ├── spec-index-template.md                  # INDEX.md template
    ├── toplan-template.md                      # TOPLAN starter-prompt template
    ├── sub-spec-conventions.md                 # filename / status / split rules
    └── parallel-implementation-checklist.md    # file-overlap audit procedure
```

---

## Dependencies

- **[Claude Code](https://claude.com/claude-code)** — the harness this skill runs in.
- **[superpowers](https://github.com/anthropics/claude-plugins)** — bigspec invokes `superpowers:brainstorming` for each slice's spec and `superpowers:writing-plans` for each slice's plan. Install the superpowers plugin (or the equivalent skills) before relying on bigspec end-to-end.

---

## Contributing / iterating on the skill

The skill was built with the [skill-creator](https://github.com/anthropics/claude-plugins) workflow: draft → run test prompts in subagents → grade outputs → iterate. Eval definitions live in `evals/evals.json`. To re-run the benchmark, see the existing iteration outputs under `../bigspec-workspace/iteration-{1,2}/`.

If you find a case where the skill mis-triggers (engages on something below threshold) or under-triggers (skips a clearly-big delivery), open an issue with the user prompt — those cases are the most useful evals to add.

---

## License

See `LICENSE` (if present) or treat as MIT-equivalent until specified.
