# /github-connect

Connect a coding agent to GitHub so it can read issues, create branches, open pull requests, and post comments — without giving it broader access than it needs.

**When to use:** Setting up GitHub MCP for the first time, scoping down an existing integration, or after a security review flags over-permissioned tokens.

---

## Steps

### 1. Decide what the agent actually needs

Pick the minimum set of GitHub actions for your workflow. Common options:

| Use case | Permissions needed |
|----------|--------------------|
| Read issues and PRs for context | `issues: read`, `pull_requests: read` |
| Create branches and push code | `contents: write` |
| Open pull requests | `pull_requests: write` |
| Post review comments | `pull_requests: write` |
| Read CI check results | `checks: read`, `statuses: read` |
| Assign labels or milestones | `issues: write` |

Start with read-only access. Expand only when a specific workflow requires it.

### 2. Create a fine-grained personal access token

Go to GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens.

Configure the token:
- **Resource owner:** your org or personal account
- **Repository access:** selected repositories only (not "All repositories")
- **Permissions:** only the ones from step 1

Name it clearly: `claude-code-[repo-name]-[year]`.

Checkpoint: Does the token have access to only the repos this agent will work in?

### 3. Set up the GitHub MCP server

Add to your `.claude/settings.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

Set `GITHUB_TOKEN` as an environment variable — never hardcode it in the config file.

```bash
export GITHUB_TOKEN=github_pat_...
```

For team use, store it in your secret manager (1Password, AWS Secrets Manager, etc.) and inject at runtime.

### 4. Verify the MCP server is working

Ask the agent to list the open issues on the repo:

```
List the 5 most recent open issues in this repo.
```

If it returns results, the connection is working. If it fails, check:
- Is `GITHUB_TOKEN` set in the environment where Claude Code runs?
- Does the token have the right repository access?
- Is the MCP server entry in `settings.json` valid JSON?

### 5. Add GitHub workflow instructions to AGENTS.md

Tell the agent how to use GitHub access consistently:

```markdown
## GitHub workflow

When working on a feature or fix:
1. Check for an existing issue before starting. Reference it in the PR.
2. Create a branch: `feat/<issue-number>-<short-description>`
3. Open a draft PR early — don't wait until complete
4. Request human review before merging — never self-merge
5. Post a comment on the issue when the PR is ready for review
```

### 6. Restrict what the agent can do with write access

If the agent has `contents: write`, document what it should not do autonomously:

```markdown
## GitHub restrictions

The agent must NOT:
- Merge pull requests
- Push directly to `main` or `develop`
- Delete branches other than ones it created
- Change repository settings, branch protection, or webhooks
```

---

## Exit criteria

- [ ] Fine-grained PAT created with minimum necessary permissions
- [ ] Token scoped to specific repositories (not all repos)
- [ ] Token stored as an environment variable, not in any file
- [ ] MCP server entry in `.claude/settings.json`
- [ ] Agent verified to read issues successfully
- [ ] GitHub workflow instructions added to AGENTS.md
- [ ] Write access restrictions documented (if token has write permissions)

---

## Common shortcuts to avoid

**"I'll use a classic token with repo scope."** Classic tokens can't be scoped to specific repos or fine-grained permissions. Use fine-grained PATs.

**"I'll just put the token in the config file."** Config files end up in git. Even in private repos, this is a bad habit. Always use environment variables or secret managers.

**"The agent can decide when to merge."** Merging is a human decision. PRs exist so humans can review. Letting agents merge removes the review step that catches problems.
