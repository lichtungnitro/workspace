---
name: sequential-thinking
description: Drive a bounded, revisable, replayable reasoning process for non-trivial problems. Use whenever a task needs multi-step analysis, root-cause diagnosis, design planning, scope decomposition, trade-off comparison between a few candidate paths, or audit of an existing judgment — especially when the user's initial framing is vague, when premature conclusions are likely, or when you catch yourself about to answer a complex question in a single long prose blob. Prefer this over one-shot answers whenever the problem has interacting parts, hidden assumptions, or needs a trace someone else can review later.
license: MIT
---

# Sequential Thinking

The point of this skill is not "write more thoughts." It is to keep reasoning **moving forward under a budget, allow honest revision when evidence shifts, and force convergence to a conclusion**. The CLI is just the execution surface — this skill defines *when* to switch into step-wise reasoning and *how* to keep it from degenerating into loose rambling.

## Mission

Turn a messy problem into a reasoning process that is **bounded, revisable, and reviewable**:

- Clarify the problem before reaching for an answer.
- Let judgments be corrected mid-flight when new evidence appears, instead of defending the earlier frame.
- Compare a small number of alternative paths when complexity rises, rather than railroading one.
- Converge to a conclusion within a fixed step budget.
- Leave behind a replayable trace so the reasoning itself can be audited.

The failure mode this skill addresses is not "can't think" — it's "thinks too diffusely, concludes too early, and can't be re-examined afterwards."

## Core Capabilities

- **Iterative progress** — decompose a hard problem into a sequence of steps instead of trying for a complete answer in one shot.
- **Dynamic revision** — when new evidence lands, revisit and correct earlier judgments rather than stacking on a wrong premise.
- **Branch comparison** — when genuine alternatives exist, line them up and compare before committing.
- **Context anchoring** — hold the problem's scope and goal steady across steps so reasoning doesn't drift.
- **Forced convergence** — every session must end in a concrete judgment, not an open-ended "I could keep thinking."

## When to Use

Reach for this skill when:

- The problem has several interdependent reasoning steps.
- The scope, method, or even the real question is unclear and needs to be framed before it can be answered.
- Two or three candidate approaches deserve a side-by-side comparison rather than a pick-first-plausible.
- A prior judgment needs to be re-examined for gaps, weak evidence, or unstated assumptions.
- The user (or a future reader) will benefit from a replayable reasoning trace.

Do **not** use it for:

- Simple factual lookups.
- Tasks that can genuinely be finished in one step.
- Problems where the path is already obvious — adding steps just performs deliberation without producing any.
- Pure open-ended brainstorming where convergence is explicitly not wanted.

## Working Philosophy

- **Find the real question before the answer.** A good description of the symptom is not a diagnosis of the cause.
- **Revise the premise instead of defending it.** If an earlier step was wrong, say so and correct it — don't keep building on the bad foundation.
- **Remove complexity before adding solutions.** Identify the dominant constraint first; patching secondary issues first usually creates more accidental complexity.
- **Each step advances one increment.** State only what this step adds; don't restate the whole problem each time.
- **Always land.** "I could keep thinking" is not an acceptable terminal state — conclude with a judgment, even a tentative one with caveats.

## Installation & Runtime Model

This skill delivers the thinking discipline; a small CLI (`sthink`) provides the runtime — it manages state, cadence, persistence, and replay so you can focus on the reasoning.

Install once:

```bash
npm install -g sequential-thinking-cli
# or
pnpm add -g sequential-thinking-cli
```

After installation, use `sthink` as the command entry point.

## CLI Contract

You do not hand-write thought JSON. The runtime exposes three primary actions:

- `start` — open a session
- `step` — advance one step
- `replay` — review or export a finished session

### `start`

Accepts exactly four inputs:

- `name` — short, human-readable session label
- `goal` — what this session is trying to settle
- `mode` — `explore`, `branch`, or `audit`
- `totalSteps` — `5` or `8`

Mode guidance:

- Default to `explore` when unsure.
- Use `branch` only when the task is explicitly comparing candidate paths.
- Use `audit` only when the task is re-examining an existing judgment.

Step-budget guidance: `5` is the right starting size for most problems. Only upgrade to `8` when, mid-session, you hit genuine new complexity that `5` can't absorb — not preemptively.

### `step`

Accepts exactly:

- `content` — the substance of this step

Other context (session id, step index, mode, history) is restored by the runtime. Don't re-paste prior context into `content`; it wastes budget and blurs the step's contribution.

### `replay`

Reads a completed session and produces a replay document. Optionally export it to the current directory when the trace needs to be shared or archived.

## Recommended Workflow

```text
1. Confirm the problem actually needs sequential thinking — don't reach for it by default.
2. Ensure the CLI is installed.
3. `sthink start` with name, goal, mode, totalSteps.
4. `sthink step` per increment — each step carries only the new move, not the full backstory.
5. When new evidence contradicts an earlier step, revise explicitly instead of pretending otherwise.
6. At the convergence step, produce: conclusion, residual risks, and the next concrete action.
7. Use `sthink replay` if the trace needs to be reviewed, shared, or archived.
```

## Examples

The value of these examples is not the JSON, the flags, or the exact prose — it is in how each sequence **advances, revises, and converges**.

### Example 1 — Baseline progression

```bash
sthink start --name "query-diagnosis" --goal "Locate the root cause of query latency regression" --mode explore --totalSteps 5
sthink step --sessionPath "<session-path>" --content "Don't pick an optimization yet. First split the problem into layers: is this a single SQL regressing, an API-level N+1, or a higher-level call-graph amplification? Without that split, caching/indexing/rewrites are all just speculative patches."
sthink step --sessionPath "<session-path>" --content "Query logs show the user-detail endpoint triggering many repeated reads per request — a clear N+1 signature. But repetition may be a symptom rather than the cause; the bottleneck could equally be one expensive underlying query. Raising the step budget to 8 to confirm."
sthink step --sessionPath "<session-path>" --content "Converging: the dominant cause is batch-loading N+1 on the list page; a secondary contributor is a missing index on a join column that inflates each lookup. Fix order: eliminate the N+1 first, then add the index and verify tail latency. This hits the primary constraint and avoids prematurely introducing cache complexity."
```

### Example 2 — Revising a premise

```bash
sthink step --sessionPath "<session-path>" --content "Revising step 2 in light of the profiling data: the real bottleneck is not the N+1 itself but a missing index on the join column, which turns each lookup into a near-full scan. The N+1 still exists, but it is an amplifier, not the first-order cause. Reorder priorities accordingly."
```

### Example 3 — Decomposing a complex change request

```bash
sthink start --name "change-impact-analysis" --goal "Decompose a multi-part rule change by impact and priority" --mode explore --totalSteps 5
sthink step --sessionPath "<session-path>" --content "The user bundled several edits together; treating them uniformly would be a mistake. Sort by kind: mechanism/principle changes, numeric balance changes, interface-semantics changes, and documentation-implementation drift. Without this sort, ADR-level edits, design-doc edits, and code-contract edits will get mixed into one undifferentiated patch."
sthink step --sessionPath "<session-path>" --content "Build an impact matrix. Principle-level edits ripple into ADRs and system design. Balance tweaks touch rule tables, configs, and test baselines. Interface-semantics edits are the most dangerous because they silently break caller assumptions. The real risk is not the count of changes but whether any of them modify an implicit contract that several modules rely on but no document describes."
sthink step --sessionPath "<session-path>" --content "Converging: handle items that change system boundaries or call semantics first; numeric and UX tweaks come after. Fix docs and contracts before debating balance, or every downstream implementation and review will rest on a drifting premise. The ordering rule is not 'start with the most visible item' — it is 'start with whatever most corrupts shared understanding of the system.'"
```

### Example 4 — Branch comparison

```bash
sthink start --name "performance-tradeoff" --goal "Compare cache-first stopgap vs. query-level fix" --mode branch --totalSteps 5
sthink step --sessionPath "<session-path>" --content "Option A — introduce a cache to shave peaks. Ships fast, low surface-area change at the API layer, good short-term relief. Downside: it converts 'the database is slow' into 'cache coherence and invalidation are now in the hot path.' If the underlying query design is the real issue, this path bakes accidental complexity in permanently. Option B — index and rewrite the queries. Removes the bottleneck at its source and keeps the architecture simpler long-term. Costs: more careful validation of write amplification, lock contention, and regressions. Slower to land, but when the business model is stable it usually beats pre-emptive caching on the 'prefer simple' axis."
```

## Storage & Export Boundary

- The runtime persists session state and step records automatically.
- A session in the completed state can generate a replay document.
- `replay` supports export to the current directory for review, sharing, or archival.

## Heuristic Reminders

These are prompts to keep the reasoning honest — not hard gates. They are most useful when a step feels stuck or suspiciously smooth.

- **Problem definition** — are you describing a symptom or actually identifying a cause?
- **Evidence** — is this step grounded in observed facts, or in assumption?
- **Scope** — does this affect a single module, a whole system, or a cross-system contract?
- **Complexity** — are you removing essential complexity, or adding accidental complexity?
- **Convergence** — is there enough in hand to land a conclusion, or are you still diverging?

## Tips

- Don't hand-author thought JSON; let the runtime handle cadence, persistence, and replay.
- Don't make sequential thinking the default mode — use it only when a problem actually earns the overhead.
- When unsure about mode, start with `explore`.
- A `step`'s `content` carries only the new increment; skip the re-summary.
- When a premise turns out to be wrong, revise it openly — defending it costs more than correcting it.
- At the convergence step, explicitly surface: conclusion, residual risks, and next action.
- `replay` only works on sessions that have reached the completed state.
