# /agent-bugfix

Run a structured bug-fix workflow with a coding agent: reproduce the bug, locate the cause, implement the fix, verify it, and hand back a clean PR.

**When to use:** Assigning a reported bug to an agent, or pairing with an agent to debug something you're stuck on.

---

## Steps

### 1. Provide a complete bug report before starting

The agent cannot fix what it cannot reproduce. Before invoking this skill, confirm the bug report includes:

- **What happened:** the actual behaviour
- **What was expected:** the correct behaviour
- **How to reproduce:** exact steps, inputs, or a failing test
- **Environment:** OS, browser, version, runtime — whatever is relevant
- **Error output:** full stack trace or log output if available

If any of these are missing, gather them before continuing. A bug you can't reproduce is a bug you can't fix.

Checkpoint: Can you make the bug happen right now? If not, find a reproduction first.

### 2. Reproduce the bug before touching code

Ask the agent to reproduce the issue, not fix it:

```
Read the bug report. Do not change any code yet.
First, tell me: which files are most likely involved, and what would a failing test for this bug look like?
```

This surfaces the agent's understanding before it starts editing. Misdiagnosis at this stage is common and expensive.

Checkpoint: Does the agent's diagnosis match your understanding of the bug?

### 3. Write a failing test (if one doesn't exist)

Before the fix, add a test that captures the bug:

```
Write a test that fails in the current state because of this bug.
Do not fix the bug yet — just write the failing test.
```

This has two benefits:
- It proves you understand the bug well enough to specify it
- The fix is verified by making the test pass — not by manual checking

If a test is impractical (flaky environment, visual bug, etc.), document why and note what manual check will substitute.

### 4. Implement the fix

Now let the agent implement:

```
Fix the bug so the failing test passes.
Change only what is necessary. Do not refactor surrounding code.
```

The scope constraint matters. Bug fixes that include unrelated cleanups are harder to review and introduce unintended regressions.

### 5. Verify the fix

Run the full test suite, not just the new test:

```bash
npm test
npm run typecheck
npm run lint
```

Confirm:
- The new test passes
- No previously passing tests now fail
- No type errors or lint errors introduced

If any check fails, the bug fix is not done — continue in this step until all checks pass.

### 6. Review the diff before creating a PR

Read every changed file. For each change, ask:
- Is this change necessary for the fix?
- Does it introduce any new assumptions or side effects?
- Is it clear from the code what it does and why?

If any change feels unrelated to the bug, remove it. Keep the diff small and focused.

### 7. Write a clear PR description

The PR description should include:

```markdown
## What this fixes

[One sentence: what was broken and for whom]

## Root cause

[What actually caused it]

## Fix

[What changed and why this approach]

## How to verify

[Steps to confirm the fix works — ideally just "run npm test"]
```

---

## Exit criteria

- [ ] Bug reproduced before any code was changed
- [ ] Failing test written that captures the bug
- [ ] Fix makes the failing test pass
- [ ] Full test suite passes with no regressions
- [ ] Diff contains only changes necessary for the fix
- [ ] PR description includes root cause and verification steps

---

## Common shortcuts to avoid

**"The test suite takes too long, I'll skip it."** Run the relevant subset if needed, but don't skip all tests. CI will catch it — but now you've created review cycle delay instead of catching it locally.

**"The fix is obvious, no need to reproduce first."** Bugs that seem obvious often aren't. A five-minute reproduction step prevents a multi-hour wrong-direction fix.

**"I'll clean up some adjacent code while I'm in here."** Don't. Unrelated changes in a bug fix PR obscure the actual fix, complicate revert if needed, and make blame history harder to read.
