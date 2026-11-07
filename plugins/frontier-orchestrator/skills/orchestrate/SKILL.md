---
name: orchestrate
description: Playbook for running a large, multi-step coding job as a frontier-model lead (Fable or Opus) with cheaper Sonnet helpers. Use for features, audits, migrations or bug hunts that span many files, or whenever the user asks to orchestrate, fan out or delegate.
---

# Orchestrate a large job

You are the lead on Fable or Opus. Helpers on Sonnet do the volume work. This playbook keeps your own context for decisions and review.

## 1. Plan (you)

Restate the goal, the constraints and what done looks like. Split the work into pieces that can run on their own. Note which pieces depend on others, and give every shared file, type, schema or config file exactly one owner.

If the whole job fits in one or two files, skip this playbook and just do it.

## 2. Discover (scouts, in parallel)

Send `frontier-orchestrator:scout` helpers to map what you need: the files involved, how the current code works, and existing helpers to reuse. Run one scout per area, all at once. Don't read the files yourself first.

## 3. Decide (you)

Read the scout reports. Choose the design, settle open questions, and write each implementation task with:
- The goal and the design decision behind it.
- The exact files the builder owns.
- What's out of scope.
- What done looks like, and which check proves it.

Ask the user only about decisions that are genuinely theirs.

## 4. Build (builders)

Send each task to `frontier-orchestrator:builder`. Run builders in parallel only when their files don't overlap. Send dependent tasks in order.

## 5. Verify (checker, then you)

- Send `frontier-orchestrator:checker` to run the narrowest tests, lint and type checks that cover the change.
- Read every diff yourself. This is the review that matters, so don't delegate it.
- For security, concurrency, data integrity or migrations, also ask `frontier-orchestrator:deep-reviewer` (Opus) and weigh its verdict.
- If something fails, send a focused fix back to a builder with the exact failure.

## 6. Report (you)

Tell the user what was done, what was checked and how, what's left, and any risk you're not sure about.

## Cost rules

- Don't fan out for small jobs. Helpers have startup overhead, so one quick lookup is cheaper done directly.
- One helper per area or root cause, not one per file or per error.
- Reuse a helper's report instead of repeating its search.
- Escalate to Opus only with evidence that Sonnet couldn't settle the question.
