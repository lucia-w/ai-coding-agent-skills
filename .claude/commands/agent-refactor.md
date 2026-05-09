# /agent-refactor

Refactor code with a coding agent while keeping behaviour identical: characterize existing behaviour with tests, make changes in small atomic steps, and verify at each step.

**When to use:** Cleaning up technical debt, renaming or reorganizing code, extracting modules, or improving readability before a feature build.

---

## Steps

### 1. Define the scope and goal precisely

"Refactor the auth module" is not a goal. Before starting:

- What specifically will change? (rename, extract, restructure, inline, split)
- What will not change? (the public API, the database schema, the external behaviour)
- Why now? (before a feature, to fix a performance issue, to meet a convention)

Write this down:

```
Goal: Extract the JWT validation logic from `middleware/auth.ts` into `lib/jwt.ts`.
The public API of `middleware/auth.ts` must remain identical — no changes to how it's called.
No behaviour changes — only structural changes.
```

If you can't write this down precisely, the scope isn't defined yet.

### 2. Ensure test coverage before touching anything

A refactor without tests is a change without a safety net. Before the agent writes a single line:

```bash
npm test -- --coverage
```

Check the coverage report for the files you're about to change. If coverage is below 70% on the affected code, add characterization tests first (see `/agent-test-gen`).

Characterization tests capture current behaviour without judging it:

```
Write tests that capture the current behaviour of `middleware/auth.ts`.
Don't test what it should do — test what it actually does right now.
These tests will fail if we accidentally change behaviour during the refactor.
```

Checkpoint: Do you have enough tests to catch a behaviour change in the affected code?

### 3. Break the refactor into atomic steps

Each step must:
- Leave the code in a runnable, passing state
- Be independently understandable in isolation
- Not depend on a future step to make sense

Ask the agent to plan the steps:

```
Break this refactor into the smallest steps where each step compiles and all tests pass.
List them in order. We'll execute one at a time.
```

Example step sequence for extracting a module:
1. Create `lib/jwt.ts` with the extracted logic, keeping the original in place
2. Import and re-export from `middleware/auth.ts` so callers are unaffected
3. Update callers one file at a time to import from `lib/jwt.ts` directly
4. Remove the re-export from `middleware/auth.ts` once all callers are updated

This is slower than "just move it all at once" — it's also recoverable if something goes wrong.

### 4. Execute one step at a time and verify

For each step:

```
Execute step 1: [description].
After making changes, run the full test suite.
Do not proceed to step 2 until I confirm.
```

After each step:
- Run the full test suite — no regressions
- Run type checking — no new type errors
- Read the diff — does it match what the step said it would do?

If any check fails, fix it before moving to the next step. A failing intermediate state is not acceptable — it means a future step is now harder to evaluate.

Checkpoint after each step: All tests pass, all types check, diff matches the plan.

### 5. Separate refactoring from behaviour changes

This rule is strict: **a refactor commit contains no behaviour changes**. If the agent discovers a bug while refactoring, do not fix it in the same change.

When a bug surfaces:

```
Don't fix this now. Create a note or a separate branch for this bug.
We'll address it after the refactor is complete.
Continue with step [N] as planned.
```

Mixing refactoring with bug fixes makes both harder to review and harder to revert selectively.

### 6. Review the final diff

Before creating a PR, read the full diff from the base branch:

```
Read the complete diff from main.
For every changed line, confirm it's a structural change only.
Flag any line that changes what the code does, not just how it's organized.
```

Look specifically for:
- Logic changes hiding inside a rename ("I also improved this condition")
- New error handling that wasn't there before (that's a behaviour change)
- Deleted code that wasn't just moved (confirm it's unreachable before deleting)

---

## Exit criteria

- [ ] Refactor scope written down: what changes, what doesn't, why now
- [ ] Test coverage verified for affected files before any changes made
- [ ] Refactor broken into atomic steps — each leaves tests passing
- [ ] One step at a time — tests passed after each step before proceeding
- [ ] No bug fixes or feature changes mixed into the refactor
- [ ] Final diff reviewed — only structural changes, no behaviour changes

---

## Common shortcuts to avoid

**"Tests are slow, I'll run them at the end."** Run them after every step. If a test fails three steps later, you don't know which step broke it. Tests after each step keep the failure surface small.

**"The tests still pass, so the refactor is safe."** Passing tests mean the tested behaviour is unchanged. They don't cover untested code paths. Know your coverage before trusting the green check.

**"I found a bug — I'll fix it while I'm in here."** This is the most tempting shortcut and the most dangerous. A combined refactor-plus-bugfix PR is harder to review, harder to revert, and harder to blame. Create a separate ticket.

**"Let the agent do the whole refactor in one pass."** A 1,000-line refactor diff is unreviable. A sequence of 5-step atomic commits is. The agent's speed is not a reason to skip the structure.
