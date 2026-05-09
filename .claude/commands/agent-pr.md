# /agent-pr

Turn completed local changes into a clean, reviewable pull request: inspect the diff, split out unrelated changes, write a useful description, verify checks, and link the originating issue.

**When to use:** After completing a feature, bug fix, or refactor — before asking for human review. Also useful when the agent has accumulated changes across multiple concerns and the diff needs to be cleaned up before it's shareable.

---

## Steps

### 1. Read the full diff before doing anything

Ask the agent to read the complete diff from the base branch:

```
Read the full diff from main.
List every file changed and summarize what changed in each one.
Do not create a PR yet.
```

This surfaces surprises: files changed that weren't intended, leftover debug code, formatting changes mixed with logic changes. Read the summary and compare it against what you expected the change to include.

Checkpoint: Does the list of changed files match what you intended to ship?

### 2. Split out unrelated changes

A PR should have one purpose. If the diff contains unrelated changes, split them now — before the PR is opened.

Common things that don't belong:
- Formatting or whitespace changes in files you passed through
- Opportunistic refactors ("cleaned this up while I was here")
- Dependency upgrades unrelated to the feature
- Config changes for a different purpose

For each unrelated change:

```
These changes to [file] are unrelated to this PR. Revert them to their state on main.
We'll address them separately.
```

If the unrelated change is genuinely valuable, create a separate branch and PR for it.

Checkpoint: Every changed file has a clear reason to be in this PR.

### 3. Check for things that shouldn't ship

Before writing the description, scan for:

- `console.log`, `debugger`, `print`, or temporary debug statements
- Hardcoded values that should be config or env vars
- TODO comments added during development that aren't tracked elsewhere
- Commented-out code
- Test files with `.only` or `.skip` that were added temporarily

Ask the agent to check:

```
Scan the diff for debug statements, hardcoded values, skipped tests, and commented-out code.
List anything found.
```

Remove or address everything flagged before continuing.

### 4. Verify checks pass locally

Don't open a PR with a failing check. Run the same checks CI will run:

```bash
npm test
npm run typecheck
npm run lint
npm run build
```

If any check fails, fix it before creating the PR. A PR opened in a failing state creates noise and delays review.

### 5. Write a useful PR description

A good PR description answers three questions a reviewer has before reading the diff:

```markdown
## What this changes

[One sentence: what does this PR do?]

## Why

[The motivation — link to the issue, or one sentence if no issue exists]

## How to verify

[How a reviewer can confirm this works. Prefer: "run npm test". Add manual steps only if needed.]
```

Optional but useful:
- **Screenshots** for UI changes
- **Before/after** for behaviour changes
- **Out of scope** for anything a reviewer might expect but isn't here

Avoid:
- Restating what the diff already shows
- "Minor changes" or "small fix" — be specific
- Long implementation narratives — the code tells that story

### 6. Link the originating issue

If this PR closes a ticket, say so explicitly in the description:

```
Closes #123
```

GitHub will auto-close the issue on merge. If it partially addresses an issue (not fully closing it), use `Relates to #123` instead.

### 7. Set reviewers, labels, and draft status

- **Draft:** Open as a draft if the PR is for early feedback or CI verification only
- **Reviewers:** Assign at least one human reviewer — never leave this empty
- **Labels:** Apply the relevant label (`bug`, `feature`, `chore`, `agent-generated`, etc.)
- **Milestone:** Assign if the change is tied to a release

---

## Exit criteria

- [ ] Full diff read — list of changed files matches intended scope
- [ ] Unrelated changes split out or reverted
- [ ] No debug statements, skipped tests, or commented-out code
- [ ] All local checks pass: tests, types, lint, build
- [ ] PR description answers: what, why, how to verify
- [ ] Originating issue linked (`Closes #N` or `Relates to #N`)
- [ ] At least one human reviewer assigned

---

## Common shortcuts to avoid

**"CI will catch any issues."** CI catches mechanical failures. It doesn't catch a diff that's 3x bigger than intended because the agent reformatted half the codebase while it was working.

**"The description can be short — reviewers will read the diff."** A PR with no context shifts the entire burden of understanding to the reviewer. A two-sentence description costs you nothing and saves them significant time.

**"I'll clean up the debug statements after review."** Don't open the PR with them in. Review comments about debug statements are noise. Remove them before requesting review.

**"Let the agent write the PR description autonomously."** The agent describes what it did. A good PR description explains why and how to verify — context only you have. Review and rewrite the agent's draft rather than accepting it verbatim.
