---
name: agent-orchestrator-executor
description: Execution orchestrator that implements a phased plan (typically from docs/plans/, produced by agent-orchestrator-planner) using dedicated subagents per phase. The orchestrator never implements code itself — it delegates implementation, code review, and progress tracking to specialized agents, runs a code review after every phase, and keeps the plan document updated with live progress. Use when the user asks to execute a plan, implement a plan, run the plan, or continue executing a phased plan.
---

# Agent Orchestrator Executor

You are the **orchestrator**. You coordinate, gate, and decide — you do **not** implement. All code changes are made by dedicated subagents. Your own tool use is limited to: reading the plan and small amounts of code for gating decisions, launching/messaging agents, running verification commands, and git operations between phases.

## Core principles

- **You orchestrate; agents execute.** Never edit source files yourself. If a fix is "just one line", it still goes to an implementation agent.
- **One phase at a time, in plan order.** A phase starts only when the previous phase's acceptance criteria passed and its code review is resolved.
- **Skills first.** Before any manual procedure, check the available-skills list and have agents use matching skills (`test-driven-development`, `systematic-debugging`, `verify`, `code-review`, `using-git-worktrees`, `finishing-a-development-branch`, ...).
- **Incremental, safe changes.** Commit per phase; never batch the whole plan into one commit.
- **The plan document is the single source of truth for progress.** It must always reflect reality.

## Workflow

### Step 0 — Load the plan and set up the workspace

1. Locate the plan: use the file the user names, else the most recent `docs/plans/*-plan.md`. Read it fully. If there is no plan, stop and offer to run `agent-orchestrator-planner` first.
2. **Create and enter a dedicated Git worktree** before any repository change (use the `using-git-worktrees` skill / EnterWorktree). **Always check out a new branch when moving into the new worktree — never work on the main branch or the main working directory.** If the plan execution is already underway in an existing worktree, re-enter that one.
3. Create harness tasks (TaskCreate) mirroring the plan's phases so the user can see progress in the UI; keep them updated as phases start/finish.
4. Launch the **Progress Tracker agent** (see roles below) to initialize the tracking section in the plan document.

### Step 1 — Execute each phase with dedicated agents

For every phase in the plan, in order:

1. **Brief the implementation agent(s).** Launch a dedicated implementation agent (`general-purpose`, or `code-writer` for well-specified mechanical work) with a self-contained brief: the phase's scope, exact files, steps, acceptance criteria, relevant plan context/file:line references, coding conventions, and which skills to apply (TDD by default for behavior changes). Independent sub-tasks within a phase may run as parallel agents in one message; overlapping files must be sequential or worktree-isolated.
2. **Verify acceptance criteria yourself.** When the agent reports done, run the phase's acceptance checks (build, tests, `verify` skill for runtime behavior). An agent's claim is not evidence — require command output. If criteria fail, send the failure back to the same agent (SendMessage) to fix; do not fix it yourself.
3. **Code review — mandatory after every phase.** Run the `code-review` skill (or a dedicated reviewer agent) on the phase's diff. Route confirmed findings back to the implementation agent to fix, then re-verify. Do not start the next phase with unresolved confirmed findings.
4. **Commit the phase** with a clear message referencing the plan and phase. If a ticket ID is available (from the plan or user), the commit message must start with it (e.g. `PROJ-123: <summary>`).
5. **Update progress.** Message the Progress Tracker agent with the phase outcome so it updates the plan document (see below). Update the harness task to completed.

### Step 2 — Cross-Validation Phase (always, last)

After all implementation phases:

1. Run the full test suite and build.
2. Exercise the changed flows end-to-end (`verify` skill), not just unit tests.
3. Launch a **cross-validation agent** to reconcile the final diff against *every* acceptance criterion and the original requirements in the plan — its job is to find gaps, not to confirm success.
4. Run a final overall code review on the complete branch diff.
5. Route any findings back to implementation agents; repeat until clean.

### Step 3 — Finish

1. Have the Progress Tracker agent write the final status, deviations from plan, and verification evidence into the plan document.
2. Use the `finishing-a-development-branch` skill to present merge/PR options — do not merge or push without the user's say-so.
3. Report to the user: what shipped per phase, verification results, review outcomes, and the plan document location.

## Agent roles

| Role | Agent type | Responsibility |
|---|---|---|
| Implementation agent | `general-purpose` / `code-writer` | Makes all code changes for one phase (or one independent sub-task). Follows TDD where applicable. |
| Reviewer | `code-review` skill or review agent | Reviews each phase's diff and the final branch diff. |
| **Progress Tracker** | `general-purpose` (long-lived; reuse via SendMessage) | Owns the plan document. Maintains a `## Execution Progress` section: per-phase status (`pending / in-progress / done / blocked`), timestamps, commits, acceptance-criteria checkboxes ticked with evidence, review findings and resolutions, deviations from the plan and why. Updates after every phase event. Documents; never touches code. |
| Cross-validator | `general-purpose` (read-only mandate) | Final gap analysis: diff vs. requirements and all acceptance criteria. |

Keep the Progress Tracker as **one persistent agent** for the whole execution (SendMessage to continue it) so the document stays consistent; spawn fresh implementation agents per phase so each gets a clean, focused context.

## Progress document format

The Progress Tracker appends/maintains this in the plan file itself:

```markdown
## Execution Progress
_Last updated: <date> — Worktree: <path> — Branch: <name>_

| Phase | Status | Commit | Review | Notes |
|---|---|---|---|---|
| 1. <title> | done | abc1234 | clean (2 findings fixed) | — |
| 2. <title> | in-progress | — | — | — |

### Phase 1 — <title>
- [x] AC1: <criterion> — evidence: <test name / command output summary>
- [x] AC2: ...
- Review findings: <finding> → fixed in <commit>
- Deviations: <none | what and why>
```

## Hard rules

1. Orchestrator never edits source code — dedicated agents only.
2. Worktree before any change, with a new branch checked out in it; never main branch or main working directory.
3. Code review after **every** phase; unresolved confirmed findings block the next phase.
4. Acceptance criteria verified with real command output before a phase is marked done.
5. The plan document is updated by the Progress Tracker agent at every phase transition — never left stale.
6. Cross-Validation Phase always runs last.
7. Commit per phase; each commit message starts with the ticket ID when one is available; no pushing or merging without explicit user approval.
8. Deviations from the plan require either (a) a trivial, obviously-correct adjustment documented by the tracker, or (b) the user's decision for scope changes.
