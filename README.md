# farazi-agents-skill

Custom Claude Code skills for planning and executing multi-phase engineering work with dedicated subagents.

## Skills

- **`skills/agent-orchestrator-planner`** — Planning-only orchestrator for bugs and features. Asks clarifying questions, scales exploration to task size (skip / single explorer / parallel explorer fan-out), and produces a phased implementation plan with acceptance criteria saved as a `.md` file. Never writes or edits code.
- **`skills/agent-orchestrator-executor`** — Execution orchestrator that implements a phased plan (typically produced by `agent-orchestrator-planner`) using dedicated subagents per phase. The orchestrator never implements code itself — it delegates implementation and progress tracking to specialized agents, and runs a single code review at the end (Cross-Validation Phase) against the full branch diff, keeping the plan document updated with live progress.
- **`skills/closing-ceremony`** — End-of-task ritual that hardens a branch before it ships: thermonuclear code-quality review, apply fixes, simplify pass, targeted re-review of touched files, live end-to-end browser verification for UI changes, then push and open the PR. Explicit invocation only (`closing ceremony` / `/closing-ceremony`) — not auto-triggered.

Together they form a plan → execute → ship pipeline: `agent-orchestrator-planner` produces the phased plan, `agent-orchestrator-executor` carries it out phase by phase with a final cross-validation review, and `closing-ceremony` does the final hardening pass and opens the PR.

## Installation

These are [Claude Code](https://claude.com/claude-code) skills. Install into your personal skills directory (`~/.claude/skills/`) for all projects, or into `<project-root>/.claude/skills/` for a single project.

### Option A — npx (no clone required)

Uses [`degit`](https://github.com/Rich-Harris/degit) via `npx` to fetch just the skill folder straight from GitHub:

```bash
npx degit nurfarazi/farazi-agents-skill/skills/agent-orchestrator-planner ~/.claude/skills/agent-orchestrator-planner
npx degit nurfarazi/farazi-agents-skill/skills/agent-orchestrator-executor ~/.claude/skills/agent-orchestrator-executor
npx degit nurfarazi/farazi-agents-skill/skills/closing-ceremony ~/.claude/skills/closing-ceremony
```

Replace `~/.claude/skills/...` with `.claude/skills/...` to install into the current project instead. Requires Node.js/npm; nothing is installed globally.

### Option B — clone

```bash
git clone https://github.com/nurfarazi/farazi-agents-skill.git
cp -r farazi-agents-skill/skills/agent-orchestrator-planner ~/.claude/skills/
cp -r farazi-agents-skill/skills/agent-orchestrator-executor ~/.claude/skills/
cp -r farazi-agents-skill/skills/closing-ceremony ~/.claude/skills/
```

After either option, restart Claude Code (or start a new session). The skills are picked up automatically and Claude invokes them when a request matches their description (e.g. "plan this feature", "execute the plan").

## Usage

- `plan a feature to do X` / `plan this bug fix` → triggers `agent-orchestrator-planner`, which asks clarifying questions and writes a phased plan to `docs/plans/`.
- `execute the plan` / `implement the plan` / `run the plan` → triggers `agent-orchestrator-executor`, which works through the plan phase by phase, delegating to subagents, committing incrementally in an isolated git worktree/branch, and running a single cross-validation code review at the end.
- `closing ceremony` / `run the closing ceremony` / `/closing-ceremony` → triggers `closing-ceremony`, which hardens the branch (review, fix, simplify, re-review, verify) and opens the PR. Explicit invocation only.

## License

MIT — use freely, adapt to your own workflow.
