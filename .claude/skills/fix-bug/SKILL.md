---
name: fix-bug
description: Fix any bug by isolating a repro and reducing code until the root cause is obvious
---

# How to Fix Any Bug

Based on Dan Abramov's [How to Fix Any Bug](https://overreacted.io/how-to-fix-any-bug/). Use this when diagnosing a bug whose root cause is unclear.

## Core principle

A reproduction is the test. It specifies what to do, what should happen, and what actually happens. Without a repro you are debugging blind — stop and get one first.

## Workflow

- Establish a repro that reliably shows the bug
- Verify the bug is present right now
- Remove something from the relevant code
- Run the repro again
- If the bug still reproduces, keep the removal and repeat the removal step with more code
- If the bug vanishes, write down the theory, revert, and retry removing a smaller chunk
- Continue until the code is minimal and the cause is obvious

The goal is to shrink the code to the smallest form that still reproduces the bug. At that point the root cause usually reveals itself.

## Rules

- Only remove code when the bug still reproduces. Never remove code "to see what happens" — that breaks well-founded progress
- Commit after every successful removal so you can always reset to a known-good reduced state
- If you simplify the repro itself (e.g. replace a visual check with a measurement), confirm the simplified repro can still show both failure and success before relying on it
- Prefer reduction over theorizing. Theories are cheap; a smaller repro is proof
- If a debugging partner (human or AI) is giving wrong answers, it is probably because they cannot run the repro. Make the repro verifiable before asking for help

## When reduction is hard

- If the code cannot be reduced without infrastructure changes, reduce the input/state instead: strip fields, shorten inputs, disable features, remove middleware
- If the bug is flaky, repro reliability is the first bug to fix
- If removing anything makes the bug vanish, you have not yet isolated — widen the repro to include more state before reducing again
