# AI Coding Agent Skills

This repository is a content collection of skills for AI coding agents. Each file in `.claude/commands/` is a standalone workflow guide and a Claude Code slash command.

## What this repo is

A set of markdown skill files covering practical workflows for using coding agents reliably in real software projects. Skills cover: repo setup, safe shell access, GitHub integration, MCP security, ticket writing, feature development, debugging, bug fixing, refactoring, test generation, TDD, code review, security review, UI verification, performance, migrations, dependency upgrades, releases, incidents, and CI guardrails.

## How it's structured

```
.claude/commands/
├── agents-md.md                 # /agents-md              — Set up AGENTS.md for a repo
├── safe-shell.md                # /safe-shell             — Configure safe shell permissions
├── github-connect.md            # /github-connect         — Connect agent to GitHub
├── mcp-secure.md                # /mcp-secure             — Harden MCP server setup
├── agent-ticket.md              # /agent-ticket           — Write agent-ready tickets
├── agent-feature.md             # /agent-feature          — Build a new feature end-to-end
├── agent-debug.md               # /agent-debug            — Investigate unknown problems
├── agent-bugfix.md              # /agent-bugfix           — Bug fix workflow
├── agent-refactor.md            # /agent-refactor         — Safe refactoring workflow
├── agent-tdd.md                 # /agent-tdd              — Test-driven development workflow
├── agent-test-gen.md            # /agent-test-gen         — Generate tests for existing code
├── agent-pr.md                  # /agent-pr               — Turn local changes into a clean PR
├── agent-review.md              # /agent-review           — Code review workflow
├── agent-security-review.md     # /agent-security-review  — Security-focused review
├── agent-ui-check.md            # /agent-ui-check         — Verify frontend changes
├── agent-performance.md         # /agent-performance      — Investigate and fix performance
├── agent-migration.md           # /agent-migration        — Database and API migrations
├── agent-dependency-upgrade.md  # /agent-dependency-upgrade — Upgrade packages safely
├── agent-docs.md                # /agent-docs             — Update docs after code changes
├── agent-release.md             # /agent-release          — Prepare a release
├── agent-incident.md            # /agent-incident         — Production incident response
└── ci-guardrail.md              # /ci-guardrail           — CI as the agent's safety net
```

## Conventions for editing skills

- Keep each skill focused on one workflow
- Steps should be concrete and executable, not conceptual
- Every skill must have explicit **exit criteria** — a verifiable checklist
- Include a **"Common shortcuts to avoid"** section countering the most tempting shortcuts
- Avoid referencing specific tool versions that will become stale — reference the tool by name
- When you update a skill for changed tooling, note what changed at the top of the file

## What this repo is NOT

- Not a documentation site (no build step, no framework)
- Not a tutorial series (skills are reference, not learning material)
- Not tied to any single agent platform — skills work with Claude Code, Cursor, Copilot, and others

## Shell access

The agent may run the following commands without asking:

- Read-only filesystem: `find`, `ls`, `cat`, `head`, `tail`, `grep`, `wc`
- Git read-only: `git status`, `git log`, `git diff`, `git blame`, `git show`

The agent must ask before running anything else.

## Restricted commands (never run without explicit approval)

- `git push` — open a PR instead
- `git push --force` — never
- `rm`, `rm -rf` — irreversible file deletion
- `curl`, `wget`, `nc` — network access
- Any command writing outside the repo directory
