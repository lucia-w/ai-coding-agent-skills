# /ci-guardrail

Set up CI as the primary safety net for agent-generated code — the check that catches what code review and local testing missed, and the gate that prevents autonomous agents from merging broken or risky changes.

**When to use:** Setting up CI for a repo where agents will contribute code, auditing existing CI for agent readiness, or after an agent ships a regression.

---

## Steps

### 1. Audit your current CI coverage

List every check your CI currently runs. Score it against what agents typically miss:

| Check | Do you have it? | What it catches |
|-------|-----------------|-----------------|
| Unit tests | | Logic errors, regressions |
| Integration tests | | Cross-component breakage |
| Type checking | | Type mismatches agents introduce |
| Linting | | Style and pattern violations |
| Secret scanning | | Credentials committed by mistake |
| Dependency audit | | Malicious or vulnerable packages added |
| Build check | | Code that doesn't compile |
| Test coverage threshold | | Tests added but hollow |

Checkpoint: Which of these are missing? Prioritise adding type checking and secret scanning if they're absent — agents trigger both frequently.

### 2. Add secret scanning

Agents occasionally write secrets into code when given examples. Add scanning:

**GitHub Actions:**
```yaml
- name: Scan for secrets
  uses: trufflesecurity/trufflehog-actions-scan@main
  with:
    path: ./
    base: ${{ github.event.repository.default_branch }}
    head: HEAD
```

Or use `gitleaks`:
```yaml
- name: Detect secrets
  uses: gitleaks/gitleaks-action@v2
```

This runs on every PR and fails the build if credentials are found.

### 3. Enforce type checking in CI

Agents introduce more type errors than humans because they don't always track the full type graph. Make type checking a required check:

**TypeScript:**
```yaml
- name: Type check
  run: npx tsc --noEmit
```

**Python (with mypy or pyright):**
```yaml
- name: Type check
  run: mypy src/ --strict
```

Mark this as a required status check in your branch protection rules so it blocks merge.

### 4. Set a test coverage floor

An agent may write tests that pass but don't actually test the changed behaviour. A coverage threshold catches hollow test additions:

```yaml
- name: Test with coverage
  run: npm test -- --coverage --coverageThreshold='{"global":{"lines":80}}'
```

Set the threshold at your current coverage level — the goal is to prevent coverage drops, not to immediately reach 100%.

### 5. Configure branch protection

In GitHub (Settings → Branches → Branch protection rules), require:

- [ ] Pull request before merging
- [ ] Minimum 1 human approving review
- [ ] All status checks must pass (select each CI job)
- [ ] No bypassing for administrators
- [ ] Dismiss stale reviews when new commits are pushed

The "no bypassing for admins" rule is critical. Agents may have admin tokens. If admins can bypass, so can the agent.

### 6. Add a required human reviewer step

For agent-generated PRs, consider a mandatory reviewer assignment:

**GitHub Actions — auto-assign reviewer:**
```yaml
name: Assign reviewer to agent PRs
on:
  pull_request:
    types: [opened]

jobs:
  assign:
    if: contains(github.event.pull_request.labels.*.name, 'agent-generated')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/github-script@v7
        with:
          script: |
            github.rest.pulls.requestReviewers({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number,
              reviewers: ['your-username']
            })
```

Label agent PRs consistently so you can track them and apply different rules.

### 7. Monitor what agents are touching

Add a step that reports which files changed in agent PRs. Large or unexpected diffs warrant closer review:

```yaml
- name: Summarise diff
  run: |
    echo "Files changed:"
    git diff --name-only origin/${{ github.base_ref }}...HEAD
    echo ""
    echo "Lines changed:"
    git diff --stat origin/${{ github.base_ref }}...HEAD
```

If an agent changes 47 files in a PR that should touch 3, that's a signal to read the diff carefully.

---

## Exit criteria

- [ ] CI runs on every PR (not just on merge)
- [ ] Secret scanning is a required check
- [ ] Type checking is a required check
- [ ] Full test suite runs and a coverage threshold is enforced
- [ ] Branch protection requires passing checks before merge
- [ ] At least one human review required — no auto-merge
- [ ] Admin bypass is disabled on protected branches

---

## Common shortcuts to avoid

**"CI is too slow, agents will wait too long."** Speed up CI by running jobs in parallel and caching dependencies. Don't remove checks to make it faster — that defeats the purpose.

**"The agent can merge if all checks pass."** Checks verify mechanical correctness. They don't verify intent. A human reviewer asks: "Is this the right change?" An agent cannot answer that question about its own work.

**"We'll add secret scanning later."** Later is when the agent commits your AWS key. Add it before connecting the agent to your repo.
