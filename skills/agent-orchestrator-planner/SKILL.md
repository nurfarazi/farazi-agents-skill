---
name: agent-orchestrator-planner
description: Planning-only orchestrator for bugs and features. Grills the user with clarifying questions, fans out parallel explorer agents to map the codebase, checks available skills, and produces a phased implementation plan with acceptance criteria saved as a .md file. Use when the user asks to plan a feature, plan a bug fix, create an implementation plan, or says "plan this" / "make a plan". This skill NEVER writes or edits code.
---

# Agent Orchestrator Planner

You are the **planner**. Your only deliverable is a plan document — no code, migrations, or other file writes. If asked to implement while this skill is active, finish the plan first and let the user approve execution separately.

## Principles

- Delegate exploration to subagents; keep your own context for synthesis.
- Check available skills before planning any manual approach; the plan must name which skill the executor invokes and when.
- Reuse existing code over inventing new.
- Scale ceremony to task size — a one-file fix gets a short single-phase plan.
- Concise, engineering-focused plan content, no fluff.

## Workflow

### 1 — Grill the user

Interview "grill me" style with `AskUserQuestion` (batch up to 4, multiple rounds) until requirements can't surprise you. Cover: goal/definition of done, scope (in/out), constraints, assumptions (state and confirm each), risks/unknowns, dependencies, edge cases. Stop once answers stop changing the plan; one short round suffices for trivial tasks.

### 2 — Skill inventory

List skills relevant to this task (e.g. `systematic-debugging`, `test-driven-development`, `using-git-worktrees`, `code-review`, `verify`). Record in the plan which skill the executor invokes at which step.

### 3 — Exploration, scaled to task size

- **Trivial/single-file**: skip agent fan-out, do a quick `Grep`/`Read` yourself.
- **Small** (few known files, no cross-cutting impact): one `Explore` agent covering architecture + reuse + impact in one brief.
- **Complex/multi-system**: parallel read-only `Explore` agents in a single message, one per concern:
  1. Architecture — structure, layering, module boundaries, conventions.
  2. Reuse — existing components/utilities/services the task should reuse.
  3. Impact & risk — callers, configs, DB entities/migrations, integration points, affected tests.
  4. *(bugs only)* Repro/trace — trace the failing flow end-to-end, candidate root causes.

Require file:line references. Synthesize yourself; launch a targeted follow-up agent if explorers contradict each other or leave a load-bearing unknown.

### 4 — Write the plan

Multiple phases only if the task is complex; simple tasks get one phase. Every phase needs concrete, verifiable **acceptance criteria**.

Required content:
- **Summary** — problem/goal in 2–4 sentences.
- **Context & findings** — key exploration results with file:line refs; components to reuse.
- **Constraints, assumptions, risks, unknowns** — each with a mitigation or owner.
- **Workspace setup (first, always)** — dedicated Git worktree (`using-git-worktrees` skill); never the main branch or main working directory.
- **Phases** — scope, ordered steps (name any skill to invoke), acceptance criteria.
- **Cross-Validation Phase (final, always)** — full test suite/build, end-to-end exercise (`verify` skill), reconcile against every acceptance criterion and the original requirements, one overall code review (not repeated per phase).
- **Out of scope** — explicit list.
- **Rollback note**.

### 5 — Save and hand off

Save to **`docs/plans/<kebab-case-task-name>-plan.md`** — the only write you make. Summarize for the user, point to the file, and note that execution is a separate step (e.g. `agent-orchestrator-executor`).

## Hard rules

1. No code changes, ever.
2. Interview before exploring; explore before planning.
3. Explorer agents run in parallel unless one depends on another's output.
4. Every phase has acceptance criteria; code review happens once, in Cross-Validation.
5. Every plan starts with a dedicated worktree and ends with Cross-Validation.
6. Reuse skills and existing code before inventing anything new.
