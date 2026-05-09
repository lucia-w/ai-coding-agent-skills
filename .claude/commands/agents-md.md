# /agents-md

Set up or improve the `AGENTS.md` (or `CLAUDE.md`) file for this repository so coding agents have the context they need to work safely and effectively.

**When to use:** Starting a new repo, onboarding an agent to an existing codebase, or when an agent keeps making avoidable mistakes.

---

## Steps

### 1. Understand the repo

Before writing anything, read:
- `README.md` — what this project does and for whom
- Directory structure — key folders and what lives in them
- `package.json`, `pyproject.toml`, `go.mod`, or equivalent — tech stack and scripts
- Existing CI config — what checks must pass

Checkpoint: Can you describe the project in two sentences and list the main commands (install, test, build, lint)?

### 2. Draft the repo context section

Write a short description (3–5 lines) covering:
- What the project does
- Primary language and framework
- Who uses it (internal tool, public API, end-user app, etc.)

```markdown
## Project

This is a Next.js web app serving as the customer dashboard for Acme Corp.
Primary stack: TypeScript, Next.js 15, Tailwind CSS, PostgreSQL via Prisma.
Used by internal ops teams and external customers.
```

### 3. List the safe commands

Document commands the agent can run without human approval. These are typically read-only or reversible:

```markdown
## Safe commands (run freely)

- `npm test` / `npm run test:watch`
- `npm run lint`
- `npm run build` (read-only build check)
- `npm run typecheck`
- `git status`, `git diff`, `git log`
```

### 4. List the restricted commands

Document commands that require explicit human confirmation before running:

```markdown
## Restricted commands (ask before running)

- Any `git push` or `git push --force`
- `npm publish` or `yarn publish`
- Database migrations: `prisma migrate deploy`
- Any `rm -rf` or bulk delete
- Deployment commands: `fly deploy`, `vercel --prod`, etc.
- Secret rotation or env var changes
```

### 5. Document conventions

Add the coding conventions an agent needs to follow, especially ones not obvious from the code:

```markdown
## Conventions

- Use named exports, never default exports
- All async functions must have explicit error handling
- Tests live in `__tests__/` next to the file they test
- Branch names: `feat/`, `fix/`, `chore/` prefix
- Commit messages follow Conventional Commits
```

### 6. Note important constraints

Add anything that would surprise a new engineer — hidden dependencies, gotchas, environment requirements:

```markdown
## Important constraints

- Never commit `.env` files — use `.env.example` only
- The `legacy/` directory is read-only — do not refactor it
- Node 20+ required — check with `node --version`
- All DB changes require a migration file, never edit the schema directly
```

### 7. Review and trim

Read the full file back. Remove anything that:
- Duplicates what the README already says
- Applies to every project (obvious conventions)
- Is aspirational rather than actual

Keep it under 150 lines. Agents read the whole file on every invocation — longer files waste context.

---

## Exit criteria

- [ ] `AGENTS.md` (or `CLAUDE.md`) exists at the repo root
- [ ] Repo context section: what it does, tech stack, intended users
- [ ] Safe commands listed — agent can run tests and linting without asking
- [ ] Restricted commands listed — agent knows what needs approval
- [ ] Key conventions documented that aren't obvious from the code
- [ ] File is under 150 lines
- [ ] No duplicate information from README

---

## Common shortcuts to avoid

**"The README covers it."** Agents don't always read the README in every context window. AGENTS.md is the single source of truth for agent behaviour in this repo.

**"I'll add more later."** A minimal AGENTS.md on day one is worth more than a perfect one that never gets written.

**"The agent is smart enough to figure it out."** Maybe. But explicit safe/restricted command lists prevent the most costly mistakes.
