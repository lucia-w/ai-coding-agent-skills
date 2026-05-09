# /agent-feature

Build a new feature with a coding agent: plan the implementation, break it into verifiable slices, and ship a focused PR without scope creep or broken intermediate states.

**When to use:** Starting a new feature from a ticket, extending existing functionality, or any greenfield work where the agent will touch multiple files.

---

## Steps

### 1. Start from a complete ticket

Don't hand the agent an idea — hand it a ticket. Run `/agent-ticket` first if you don't have one. A complete ticket includes:
- The outcome (what done looks like)
- Acceptance criteria (verifiable checklist)
- Relevant files or areas
- Explicit constraints (what not to change)

If the ticket is missing any of these, write them before continuing.

Checkpoint: Would a skilled contractor be able to execute this ticket with no back-and-forth?

### 2. Ask for an implementation plan before any code

Before the agent writes a single line:

```
Read the ticket and the relevant files.
Describe your implementation plan:
- What files will you create or modify?
- What's the order of changes?
- Are there any risks or ambiguities you want to flag before starting?
Don't write any code yet.
```

Review the plan. Look for:
- Files the agent missed that it will likely need
- A risky change early in the sequence (should it be later?)
- Any assumption that contradicts a constraint

Agree on the plan before continuing. A plan disagreement at this stage costs 5 minutes; discovering the same disagreement mid-implementation costs much more.

### 3. Break the work into vertical slices

A vertical slice is a change that leaves the codebase in a runnable, testable state. It's not a layer ("do all the database work first") — it's an end-to-end thin slice of functionality.

Good slice sequence for a feature:
1. Data model change + migration (if needed)
2. Business logic + unit tests
3. API endpoint + integration test
4. UI component + wired to API
5. Edge cases and error states

Ask the agent to identify slices:

```
Break this feature into the smallest slices where each slice leaves the code in a working state.
List them in order. We'll implement and verify one at a time.
```

### 4. Implement one slice at a time

For each slice:

```
Implement slice 1: [description].
Run tests and lint after.
Don't start slice 2 until I confirm this one looks right.
```

After each slice:
- Review the diff (is it what you expected?)
- Run the test suite (no regressions?)
- If it's a UI slice, open it in a browser

Don't let the agent queue up multiple slices and implement them together. This defeats the purpose of slicing.

Checkpoint after each slice: Do the tests pass? Does the diff match the plan?

### 5. Write tests alongside the code, not after

Tests written after the fact are harder to write well and easier to skip. For each slice:

```
For this slice, write the tests before or alongside the implementation.
Tests should cover the happy path and at least one error or edge case.
```

If a slice is genuinely untestable (a UI layout change, a config tweak), note why and specify what manual check substitutes.

### 6. Keep the diff reviewable

Before creating the PR:

- The total diff should reflect the planned scope — no more, no less
- No speculative additions ("I added X while I was in here — seemed useful")
- No reformatting of unrelated files
- No commented-out code

Ask the agent to review the diff:

```
Read the full diff from main.
List every file changed and why.
Flag anything that wasn't in the original plan.
```

Remove anything that wasn't in the plan. If you want it, create a follow-up ticket.

### 7. Write the PR description

```markdown
## What this adds

[One sentence: what the feature does and for whom]

## Implementation

[How it was built — approach and key decisions]

## How to verify

[Steps to test the feature manually, if needed, plus: "run npm test"]

## Out of scope

[What you explicitly chose not to include and why]
```

The "out of scope" section prevents reviewers from asking "why didn't you also do X?" — a question that derails review.

---

## Exit criteria

- [ ] Complete ticket existed before starting
- [ ] Implementation plan reviewed and agreed on before any code written
- [ ] Work broken into vertical slices — each slice left the codebase in a runnable state
- [ ] Tests written alongside the implementation for each slice
- [ ] Full test suite passes with no regressions
- [ ] Diff reviewed — no out-of-plan changes
- [ ] PR description includes what was added, how, and what was explicitly left out

---

## Common shortcuts to avoid

**"Skip the plan, just start."** Planning feels slow. Discovering the agent's mental model is wrong after 30 minutes of implementation is slower. Five minutes of planning prevents that.

**"Do all the slices at once and I'll review at the end."** A 600-line diff from an agent is hard to review. Six 100-line diffs are not. The slice boundary is the review opportunity — use it.

**"I'll add tests in a follow-up PR."** Tests added after the fact are written to pass, not to catch bugs. Write them during the feature so they inform the implementation.

**"The feature works, so the diff is fine."** Working code can still contain unrelated changes that complicate future debugging or revert. Review the diff, not just the outcome.
