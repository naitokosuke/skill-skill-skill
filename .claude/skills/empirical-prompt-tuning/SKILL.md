---
name: empirical-prompt-tuning
description: A method for iteratively improving agent-targeted text instructions (skills / slash commands / task prompts / CLAUDE.md sections / code-generation prompts) by having a bias-free executor actually run them and evaluating from both sides (executor self-report + caller-side metrics). Loop until improvements plateau. Use right after creating or substantially revising a prompt or skill, or when you suspect the cause of an agent's misbehavior is ambiguity on the instruction side.
---

# Empirical Prompt Tuning

The author of a prompt cannot judge its quality. The clearer it feels to the writer, the more likely a different agent reading it will get stuck. The core of this skill is to have a bias-free executor actually run it, evaluate from both sides, and iterate. Don't stop until improvements plateau.

## When to use

- Right after creating or substantially revising a skill / slash command / task prompt
- When an agent isn't behaving as expected and you want to attribute the cause to instruction-side ambiguity
- When you want to harden a high-importance instruction (frequently-used skill, automation-core prompt)

When not to use:
- One-off, throwaway prompts (evaluation cost isn't worth it)
- When the goal isn't to improve success rate but just to bake in the writer's subjective preferences

## Workflow

### Iteration 0 — description / body alignment check

Static, no dispatch needed.

- Read the triggers / use cases the frontmatter `description` advertises
- Read what the body actually covers
- If they diverge, align description or body before iter 1
- For example, a description claiming "navigation / form filling / data extraction" but a body that's only the `npx playwright test` CLI ref
- If you skip this, the subagent will "reinterpret" the body to match the description, so the skill scores well even though it doesn't really meet the requirements (false positive)

### Baseline setup

Fix the target prompt and prepare two things.

- 2-3 evaluation scenarios (1 median + 1-2 edge). Realistic tasks where the target prompt would actually be applied.
- A requirements checklist for accuracy calculation. For each scenario, list 3-7 items the deliverable must satisfy. Accuracy % = items met / total items. Lock this in advance (no shifting later).

### Bias-free read

Have a "blank-slate" executor read the instructions. Dispatch a fresh subagent with the Task tool. Don't substitute self-rereading (it's structurally impossible to objectively view text you just wrote). To run multiple scenarios in parallel, place multiple Agent calls in a single message. For dispatch-incapable environments, see the "Environment constraints" section.

### Execution

Pass a prompt that follows the subagent invocation contract (below) and have the subagent run the scenario. The executor produces an implementation or output and ends with a self-report.

### Two-sided evaluation

From the returned result, record an executor self-report (extracted from the subagent's report body) covering unclear points, discretionary completions, and places where the template was hard to apply.

Also record the caller-side measurements (the judgment rules are defined here in one place; refer back to this section elsewhere):

- Pass / fail counts as pass (○) only when every item tagged `[critical]` is met (○). If even one is × or partial, it's a fail (×). Labels are binary — only ○ / ×.
- Accuracy is the requirements-checklist completion rate %. ○ = full, × = 0, partial = 0.5; sum and divide by total.
- Step count uses the `tool_uses` field in the Task tool's return-value usage meta as-is. Include Read / Grep — don't filter out.
- Duration uses the Task tool usage meta's `duration_ms`.
- Retries — how many times the subagent redid the same decision, extracted from the subagent's self-report (the caller can't measure this).
- On failure, add a one-liner under "unclear points" naming which `[critical]` item failed (for cause tracing).

The requirements checklist must include at least one `[critical]`-tagged item (otherwise pass-judgment is vacuously true). Don't add or remove `[critical]` after the fact.

### Apply diff

Add the minimum fix to the prompt to close an unclear point. One iteration, one theme (multiple related fixes are OK; unrelated fixes go in the next iteration).

Before applying, state explicitly which checklist / judgment-text item this fix is intended to satisfy. Fixes inferred from axis names alone tend to miss (see "Patterns of fix propagation").

### Re-evaluate

Repeat the bias-free read through apply-diff loop with a new subagent (don't reuse the same one — it's learned the previous improvements). Increase parallelism when iterations stop yielding plateau.

### Convergence

Rough rule — stop when two consecutive iterations have zero new unclear points and metric improvements are below threshold (see below). For high-importance prompts, require three consecutive.

## Evaluation axes

- Pass / fail — did the executor produce the intended deliverable (binary)? Minimum bar.
- Accuracy — what % of requirements does the deliverable meet? Degree of partial success.
- Step count — tool calls / decision steps the executor used. Indicator of instruction waste.
- Duration — the executor's `duration_ms`. Proxy for cognitive load.
- Retries — how often the same decision was redone. Signal for ambiguity.
- Unclear points (self-report) — bullets enumerated by the executor. Qualitative improvement material.
- Discretionary completions (self-report) — decisions not specified in the instruction. Surfaces implicit specs.

Weighting is qualitative-first (unclear points, discretionary completions) with quantitative (time, step count) supporting. Chasing only time-shrinkage starves the prompt.

### Qualitative reading of `tool_uses`

If you only look at accuracy, structural skill problems hide. Use `tool_uses` as a relative value across scenarios to expose structural defects:

- If one scenario is 3-5× or more above others, that skill is likely decision-tree index-shaped with low self-containment — the executor is being forced to descend into references
- A typical case is all scenarios at `tool_uses` 1-3 except one at 15+, meaning no recipe for that scenario inside the skill, so the executor is searching across `references/`
- In iter 2, add an "inline minimum complete example" or guidance on "when to read references/" to the top of `SKILL.md`; `tool_uses` drops sharply

Even at 100% accuracy, lopsided `tool_uses` is grounds for triggering iter 2. "Stop when accuracy is good enough" misses structural defects.

### Patterns of fix propagation (conservative / overshoot / zero)

Fix → effect isn't linear. Pre-estimates can land in three patterns:

- Conservative drift (estimate > actual) — a single fix targeted multiple axes but only moved one. "Multi-axis fixes tend to miss."
- Overshoot (estimate < actual) — a single piece of structural information (e.g. command + config + expected output combined) satisfied multiple axes' judgment text at once. "Information combinations are structurally multi-axis-effective."
- Zero drift (estimate > 0, actual = 0) — a fix inferred from an axis name didn't reach any of the actual judgment text. "Axis name and judgment text are different things."

To stabilize this, before applying the diff have the subagent verbalize "which judgment text does this fix satisfy". Without binding at the threshold-text level, estimate accuracy doesn't materialize. When introducing a new evaluation axis, also concretize each scoring point's criterion at threshold-text granularity (e.g. "all explicit", "complete minimum runnable config" — at a granularity where the subagent can decide what makes a 2-point answer).

## Subagent invocation contract

The prompt passed to the executor takes this shape. This is the input contract for "two-sided evaluation":

```
You are an executor reading <target prompt name> with a blank slate.

## Target prompt
<full body of the target prompt, or a path to Read>

## Scenario
<one-paragraph scenario setup>

## Requirements checklist (items the deliverable must satisfy)
1. [critical] <item to include in the minimum bar>
2. <regular item>
3. <regular item>
...
(Judgment rules are centralized in "Workflow 4. Two-sided evaluation / Caller-side measurements". At least one [critical] is required.)

## Task
1. Execute the scenario per the target prompt and produce the deliverable.
2. End by responding in the report structure below.

## Report structure
- Deliverable: <produced artifact or execution-result summary>
- Requirements met: ○ / × / partial (with reason) per item
- Unclear points: places in the target prompt where you got stuck or hesitated to interpret (bullets)
- Discretionary completions: places not specified that you filled in by your own judgment (bullets)
- Retries: how many times you redid the same decision and why
```

The caller extracts the self-report portion from the response and pulls `tool_uses` / `duration_ms` from the Agent tool's usage meta to fill the evaluation-axes table.

## Environment constraints

In environments where dispatching a fresh subagent isn't possible (you're already running as a subagent, the Task tool is disabled, etc.), don't apply this skill.

- One workaround is to ask the parent session's user to spawn a separate Claude Code session and delegate
- Another is to skip the evaluation and explicitly report to the user "empirical evaluation skipped: dispatch unavailable"
- Don't substitute self-rereading (bias contaminates the result; the conclusion can't be trusted)

### Structural-review mode

When you only want to check the textual coherence and clarity of a skill / prompt — not run an empirical evaluation — explicitly mark it as structural-review mode. State in the subagent prompt "this is structural review mode: text consistency check, not execution". This lets the subagent skip the environment-constraints skip behavior and return a static review. Structural review is a complement to empirical evaluation, not a replacement (it can't be used for the consecutive-clear convergence judgment).

## Stopping criteria

- Convergence (stop) — two consecutive iterations satisfying every one of these conditions:
  - New unclear points stay at 0
  - Accuracy improvement vs prior is ≤ +3 points (saturated, e.g. 5% → 8%)
  - Step-count change vs prior is within ±10%
  - Duration change vs prior is within ±15%
  - Overfit check — at convergence, add one previously-unused hold-out scenario and evaluate. If accuracy drops 15+ points below the recent average, you've overfit. Go back to baseline scenario design and add an edge case.
- Divergence (suspect the design) — 3+ iterations without unclear-point reduction means the prompt's design is likely wrong. Stop patching and rewrite the structure.
- Resource cutoff — stop when importance and improvement cost stop balancing (the "ship at 80 points" call).

## Reporting format

For each iteration, record and present to the user in this form:

```
## Iteration N

### Changes (diff vs prior)
- <one-line fix summary>

### Results (per scenario)
| Scenario | Pass/Fail | Accuracy | steps | duration | retries |
|---|---|---|---|---|---|
| A | ○ | 90% | 4 | 20s | 0 |
| B | × | 60% | 9 | 41s | 2 |

### Unclear points (new this iteration)
- <Scenario B>: [critical] item N failed — <one-line cause>   # always include on failure
- <Scenario B>: <other observation, one line>
- <Scenario A>: (none new)

### Discretionary completions (new this iteration)
- <Scenario B>: <what was filled in>

### Next fix
- <one-line minimum fix>

(Convergence judgment: X consecutive clears / Y left until stop)
```

## Red flags (watch for rationalizations)

- "I can re-read it myself, same effect" — you can't "objectively view" what you just wrote. Always dispatch a fresh subagent.
- "One scenario is enough" — one scenario overfits. Minimum 2, ideally 3.
- "Got zero unclear points once — done" — could be coincidence. Lock in convergence after two consecutive.
- "Let's fix several unclear points at once" — you won't know which fix worked. One iteration, one theme.
- "Split each related micro-fix into its own iter" — trap in the other direction. "One theme" is a semantic unit; 2-3 related micro-fixes can share one iter. Splitting too far blows up iter count.
- "Metrics look good, ignore qualitative feedback" — time-shrinkage can also be a starvation signal. Qualitative is primary.
- "Faster to rewrite from scratch" — right answer if 3+ iters didn't reduce unclear points. Before that, it's an escape.
- "Reuse the same subagent" — it's learned the previous improvements. Always dispatch fresh.

## Common failures

- Scenarios too easy or too hard — neither produces a signal. One realistic median, one edge.
- Metrics-only — chasing only time-shrinkage strips out important explanations and the prompt becomes brittle.
- Too many changes per iteration — you can't trace which fix worked. One fix, one iteration.
- Tuning scenarios to the fix — making scenarios easier so the unclear points appear closed defeats the purpose.

## Related

- `superpowers:writing-skills` — TDD-style skill creation, sharing the same "subagent baseline → fix → re-run" essence as this skill
- `retrospective-codify` — locking in lessons after a task; use this skill during prompt development and `retrospective-codify` after the task ends
- `superpowers:dispatching-parallel-agents` — etiquette for running multiple scenarios in parallel
