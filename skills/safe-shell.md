# /safe-shell

Configure shell access permissions for a coding agent so it can work effectively without being able to cause irreversible damage.

**When to use:** Setting up an agent in a new project, after an agent runs a command it shouldn't have, or when reviewing your permission config.

---

## Steps

### 1. Identify what the agent actually needs

List the commands the agent needs to do useful work in this repo. Start from what tasks you want the agent to perform:

| Task | Commands needed |
|------|----------------|
| Run tests | `npm test`, `pytest`, `go test ./...` |
| Check types | `npm run typecheck`, `tsc --noEmit` |
| Lint | `npm run lint`, `ruff check .` |
| Read the codebase | `find`, `grep`, `cat`, `ls` |
| Explore git history | `git log`, `git diff`, `git status`, `git blame` |
| Install dependencies | `npm install`, `pip install` |
| Build | `npm run build` |

Checkpoint: If you only allow these, can the agent complete 80% of the tasks you'll give it?

### 2. Build your allowlist in AGENTS.md

Explicit allowlists are safer than blocklists. Document what is allowed, not just what is banned:

```markdown
## Shell access

The agent may run the following commands without asking:
- Read-only file system: `find`, `ls`, `cat`, `head`, `tail`, `grep`, `wc`
- Git read-only: `git status`, `git log`, `git diff`, `git blame`, `git show`
- Tests: `npm test`, `npm run test:unit`, `npm run test:e2e`
- Quality checks: `npm run lint`, `npm run typecheck`, `npm run build`
- Package install: `npm install` (no publish)

The agent must ask before running anything else.
```

### 3. Configure tool permissions (Claude Code)

In `.claude/settings.json`, use the permissions field to lock down what tools and commands are auto-approved:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test:*)",
      "Bash(npm run lint)",
      "Bash(npm run typecheck)",
      "Bash(npm run build)",
      "Bash(git status)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(find:*)",
      "Bash(grep:*)"
    ],
    "deny": [
      "Bash(git push:*)",
      "Bash(npm publish:*)",
      "Bash(rm -rf:*)",
      "Bash(curl:*)",
      "Bash(wget:*)"
    ]
  }
}
```

Checkpoint: Has someone other than you reviewed these permissions? A second pair of eyes catches gaps.

### 4. Block the high-risk commands explicitly

Even if you trust the agent, make high-risk commands require approval. Add to your deny list:

```
# Irreversible file operations
rm -rf, find -delete, truncate

# Network access (can exfiltrate data or call external APIs)
curl, wget, nc, ncat, python -c 'import urllib'

# Publishing and deployment
npm publish, yarn publish, pip publish
git push --force, git push origin main
fly deploy, vercel --prod, kubectl apply

# Secret access
cat .env, printenv, env | grep -i secret
```

### 5. Test the configuration

Give the agent a test task that should succeed with the allowlist, then try to get it to run a restricted command. Verify the permission system blocks it.

---

## Exit criteria

- [ ] Explicit allowlist exists (in AGENTS.md or settings file)
- [ ] High-risk commands are in the deny list: `rm -rf`, `git push`, `npm publish`, network fetch
- [ ] Agent has been tested with a real task and didn't hit unnecessary permission blocks
- [ ] Permissions are version-controlled (in `.claude/settings.json` or equivalent)
- [ ] Secrets are not accessible via shell commands the agent can run

---

## Common shortcuts to avoid

**"I'll just approve each command as it comes up."** You will approve things without reading them closely, especially after the tenth prompt. Explicit configuration is reviewed once and applied consistently.

**"Blocking curl is too restrictive."** If your agent needs to call external APIs, create a specific allowlist entry for the exact domains it needs. Don't open all network access.

**"The agent won't do anything destructive."** It won't intentionally. But a misunderstood prompt, a prompt injection from a file the agent reads, or a typo in a command can cause real damage.
