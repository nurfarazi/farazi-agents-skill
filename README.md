# farazi-agents-skill

Custom Claude Code skills for planning and executing multi-phase engineering work with dedicated subagents.

## Skills

- **`skills/agent-orchestrator-planner`** — Planning-only orchestrator for bugs and features. Asks clarifying questions, fans out parallel explorer agents to map the codebase, and produces a phased implementation plan with acceptance criteria saved as a `.md` file. Never writes or edits code.
- **`skills/agent-orchestrator-executor`** — Execution orchestrator that implements a phased plan (typically produced by `agent-orchestrator-planner`) using dedicated subagents per phase. The orchestrator never implements code itself — it delegates implementation, code review, and progress tracking to specialized agents, runs a code review after every phase, and keeps the plan document updated with live progress.

Together they form a plan → execute pipeline: `agent-orchestrator-planner` produces the phased plan, `agent-orchestrator-executor` carries it out phase by phase with review gates in between.

## Installation

These are [Claude Code](https://claude.com/claude-code) skills. Install into your personal skills directory (`~/.claude/skills/`) for all projects, or into `<project-root>/.claude/skills/` for a single project.

### Option A — npx (no clone required)

Uses [`degit`](https://github.com/Rich-Harris/degit) via `npx` to fetch just the skill folder straight from GitHub:

```bash
npx degit nurfarazi/farazi-agents-skill/skills/agent-orchestrator-planner ~/.claude/skills/agent-orchestrator-planner
npx degit nurfarazi/farazi-agents-skill/skills/agent-orchestrator-executor ~/.claude/skills/agent-orchestrator-executor
```

Replace `~/.claude/skills/...` with `.claude/skills/...` to install into the current project instead. Requires Node.js/npm; nothing is installed globally.

### Option B — clone

```bash
git clone https://github.com/nurfarazi/farazi-agents-skill.git
cp -r farazi-agents-skill/skills/agent-orchestrator-planner ~/.claude/skills/
cp -r farazi-agents-skill/skills/agent-orchestrator-executor ~/.claude/skills/
```

After either option, restart Claude Code (or start a new session). The skills are picked up automatically and Claude invokes them when a request matches their description (e.g. "plan this feature", "execute the plan").

## Usage

- `plan a feature to do X` / `plan this bug fix` → triggers `agent-orchestrator-planner`, which asks clarifying questions and writes a phased plan to `docs/plans/`.
- `execute the plan` / `implement the plan` / `run the plan` → triggers `agent-orchestrator-executor`, which works through the plan phase by phase, delegating to subagents, running code review after each phase, and committing incrementally in an isolated git worktree/branch.

## License

MIT — use freely, adapt to your own workflow.
