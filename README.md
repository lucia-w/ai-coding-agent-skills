# AI Coding Agent Playbook

Practical skills for using coding agents reliably in real software projects.

A collection of structured workflows for Claude Code, Cursor, Copilot, and other AI coding agents — covering repo setup, safe shell access, GitHub integration, MCP, code review, and CI guardrails.
---

## Skills

| Skill | Command | Category |
|-------|---------|----------|
| [Set up AGENTS.md](skills/agents-md.md) | `/agents-md` | Repo Setup |
| [Safe shell access](skills/safe-shell.md) | `/safe-shell` | Safety |
| [Connect to GitHub](skills/github-connect.md) | `/github-connect` | Tooling |
| [Secure MCP setup](skills/mcp-secure.md) | `/mcp-secure` | Safety |
| [Write agent-ready tickets](skills/agent-ticket.md) | `/agent-ticket` | Team Adoption |
| [Bug fix workflow](skills/agent-bugfix.md) | `/agent-bugfix` | Workflows |
| [Code review workflow](skills/agent-review.md) | `/agent-review` | Workflows |
| [CI as guardrail](skills/ci-guardrail.md) | `/ci-guardrail` | Quality |

---

## How to install

### Claude Code

Copy skills into your project's `.claude/commands/` directory:

```bash
mkdir -p .claude/commands
cp skills/*.md .claude/commands/
```

Each file becomes a slash command. For example, `skills/agents-md.md` becomes `/agents-md` in Claude Code.

### Cursor / Windsurf / other agents

Copy the skill content into your agent's custom instructions or rules files. The workflow steps work regardless of platform — the slash command format is optional.

---

## Using a skill

In Claude Code, type the command name:

```
/agents-md
```

The agent will follow the skill's workflow, check each step, and confirm exit criteria before finishing.

---

## Philosophy

- **Operational, not theoretical** — every step is something you can do right now
- **Date-aware** — tooling moves fast; skills note what they assume about the environment
- **Opinionated** — these are defaults that work, not every possible option
- **Short and replaceable** — each skill is a single file you can fork and adapt

---

