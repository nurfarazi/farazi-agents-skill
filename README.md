# farazi-agents-skill

Custom Claude Code skills for planning and executing multi-phase engineering work with dedicated subagents.

## Skills

- **`skills/agent-orchestrator-planner`** — Planning-only orchestrator for bugs and features. Asks clarifying questions, fans out parallel explorer agents to map the codebase, and produces a phased implementation plan with acceptance criteria saved as a `.md` file. Never writes or edits code.
- **`skills/agent-orchestrator-executor`** — Execution orchestrator that implements a phased plan (typically produced by `agent-orchestrator-planner`) using dedicated subagents per phase. The orchestrator never implements code itself — it delegates implementation, code review, and progress tracking to specialized agents, runs a code review after every phase, and keeps the plan document updated with live progress.

Together they form a plan → execute pipeline: `agent-orchestrator-planner` produces the phased plan, `agent-orchestrator-executor` carries it out phase by phase with review gates in between.

## Installation

These are [Claude Code](https://claude.com/claude-code) skills. To install:

1. Copy the skill folder(s) you want into your Claude Code skills directory:
   - Personal (all projects): `~/.claude/skills/`
   - Project-specific: `<project-root>/.claude/skills/`

   ```bash
   cp -r skills/agent-orchestrator-planner ~/.claude/skills/
   cp -r skills/agent-orchestrator-executor ~/.claude/skills/
   ```

2. Restart Claude Code (or start a new session). The skills will be picked up automatically and Claude will invoke them when a request matches their description (e.g. "plan this feature", "execute the plan").

## Usage

- `plan a feature to do X` / `plan this bug fix` → triggers `agent-orchestrator-planner`, which asks clarifying questions and writes a phased plan to `docs/plans/`.
- `execute the plan` / `implement the plan` / `run the plan` → triggers `agent-orchestrator-executor`, which works through the plan phase by phase, delegating to subagents, running code review after each phase, and committing incrementally in an isolated git worktree/branch.

## License

MIT — use freely, adapt to your own workflow.
