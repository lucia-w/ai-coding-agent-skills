# AI Coding Agent Skills

Practical slash-command skills for using coding agents reliably in real software projects.

A collection of 22 structured workflows for Claude Code, Cursor, Copilot, and other AI coding agents — covering everything from repo setup and ticket writing to debugging, testing, migrations, releases, and incident response.

---

## Skills

### Repo Setup

| Skill | Command |
|-------|---------|
| [Set up AGENTS.md](.claude/commands/agents-md.md) | `/agents-md` |
| [Safe shell access](.claude/commands/safe-shell.md) | `/safe-shell` |
| [Connect to GitHub](.claude/commands/github-connect.md) | `/github-connect` |
| [Secure MCP setup](.claude/commands/mcp-secure.md) | `/mcp-secure` |

### Planning

| Skill | Command |
|-------|---------|
| [Write agent-ready tickets](.claude/commands/agent-ticket.md) | `/agent-ticket` |

### Development

| Skill | Command |
|-------|---------|
| [Build a new feature](.claude/commands/agent-feature.md) | `/agent-feature` |
| [Investigate unknown problems](.claude/commands/agent-debug.md) | `/agent-debug` |
| [Bug fix workflow](.claude/commands/agent-bugfix.md) | `/agent-bugfix` |
| [Safe refactoring](.claude/commands/agent-refactor.md) | `/agent-refactor` |

### Testing

| Skill | Command |
|-------|---------|
| [Test-driven development](.claude/commands/agent-tdd.md) | `/agent-tdd` |
| [Generate tests for existing code](.claude/commands/agent-test-gen.md) | `/agent-test-gen` |

### Review & Quality

| Skill | Command |
|-------|---------|
| [Clean up and open a PR](.claude/commands/agent-pr.md) | `/agent-pr` |
| [Code review](.claude/commands/agent-review.md) | `/agent-review` |
| [Security review](.claude/commands/agent-security-review.md) | `/agent-security-review` |
| [Verify frontend changes](.claude/commands/agent-ui-check.md) | `/agent-ui-check` |

### Maintenance

| Skill | Command |
|-------|---------|
| [Investigate and fix performance](.claude/commands/agent-performance.md) | `/agent-performance` |
| [Database and API migrations](.claude/commands/agent-migration.md) | `/agent-migration` |
| [Upgrade dependencies safely](.claude/commands/agent-dependency-upgrade.md) | `/agent-dependency-upgrade` |
| [Update docs after code changes](.claude/commands/agent-docs.md) | `/agent-docs` |
| [Prepare a release](.claude/commands/agent-release.md) | `/agent-release` |
| [Production incident response](.claude/commands/agent-incident.md) | `/agent-incident` |

### CI

| Skill | Command |
|-------|---------|
| [CI as guardrail](.claude/commands/ci-guardrail.md) | `/ci-guardrail` |

---

## How to install

### Claude Code

From this repository, copy the commands into another project's `.claude/commands/` directory:

```bash
mkdir -p .claude/commands
cp .claude/commands/*.md /path/to/your-project/.claude/commands/
```

Each file becomes a slash command. For example, `agent-bugfix.md` becomes `/agent-bugfix` in Claude Code.

### Cursor / Windsurf / other agents

Copy the skill content into your agent's custom instructions or rules files. The workflow steps work regardless of platform — the slash command format is Claude Code specific.

---

## Using a skill

In Claude Code, type the command name:

```
/agent-bugfix
```

The agent will follow the skill's workflow, check each step, and confirm exit criteria before finishing.

---

## Philosophy

- **Operational, not theoretical** — every step is something you can do right now
- **Opinionated** — these are defaults that work, not every possible option
- **Short and replaceable** — each skill is a single file you can fork and adapt
- **Exit criteria first** — every skill has a verifiable checklist so you know when you're done
