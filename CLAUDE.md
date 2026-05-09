# AI Coding Agent Playbook

This repository is a content collection of skills for AI coding agents. Each file in `skills/` is a standalone workflow guide.

## What this repo is

A set of markdown skill files covering practical workflows for using coding agents reliably in real software projects. Skills cover: repo setup, safe shell access, GitHub integration, MCP security, ticket writing, bug fixing, code review, and CI guardrails.

## How it's structured

```
skills/
├── agents-md.md        # /agents-md  — Set up AGENTS.md for a repo
├── safe-shell.md       # /safe-shell — Configure safe shell permissions
├── github-connect.md   # /github-connect — Connect agent to GitHub
├── mcp-secure.md       # /mcp-secure — Harden MCP server setup
├── agent-ticket.md     # /agent-ticket — Write agent-ready tickets
├── agent-bugfix.md     # /agent-bugfix — Bug fix workflow
├── agent-review.md     # /agent-review — Code review workflow
└── ci-guardrail.md     # /ci-guardrail — CI as the agent's safety net
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

## Safe commands

```
git status, git log, git diff
cat, ls, find, grep
```

## Restricted

```
git push (open a PR instead)
```
