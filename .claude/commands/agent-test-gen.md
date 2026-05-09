# /agent-test-gen

Generate tests for existing untested code with a coding agent: characterize current behaviour, identify test cases, and build a test suite that will catch regressions.

**When to use:** Adding coverage to legacy code before refactoring it, filling test gaps before a feature change, or building a safety net for a module an agent is about to modify.

---

## Steps

### 1. Define the scope clearly

Before starting, pick a target. Don't ask the agent to "add tests to the codebase" — that's too broad. Instead:

```
Add unit tests for `src/billing/invoiceCalculator.ts`.
There are currently no tests for this file.
Focus on the exported functions: `calculateTotal`, `applyDiscount`, `roundCurrency`.
```

If you're adding tests ahead of a refactor, name the refactor too — it helps the agent choose which edge cases matter most.

### 2. Read the code before writing any tests

Ask the agent to describe the code before touching the test file:

```
Read `src/billing/invoiceCalculator.ts`.
Before writing any tests, describe in plain language what each exported function does,
what inputs it accepts, and what it returns or mutates.
```

This surfaces misunderstandings early. If the agent describes a function wrong, the tests it writes will test the wrong thing.

Checkpoint: Does the agent's description match your understanding of the code?

### 3. List test cases before writing them

Ask for a test plan first:

```
For each exported function, list the test cases you'd write:
- Happy path (typical inputs, expected output)
- Edge cases (empty, zero, negative, max values)
- Error cases (invalid input, missing required fields)

Don't write any code yet — just list the cases.
```

Review the list. Add cases the agent missed; remove cases that are trivial or already impossible given the types. This is faster than reviewing bad tests after they're written.

### 4. Write tests one function at a time

Don't generate the whole test file at once. Write tests for one function, run them, then move on:

```
Write the tests for `calculateTotal` from the list we agreed on.
Run them and confirm they all pass before moving to `applyDiscount`.
```

Tests that pass immediately on first run are worth checking — they may be testing nothing. See step 5.

### 5. Verify tests actually fail when they should

This is the step most agents skip. A test that always passes isn't a test — it's a comment.

For each test, ask:

```
For the test covering [case], what would have to be true in the code for this test to fail?
Is it possible for this test to pass even if the function is completely broken?
```

If a test can't fail, delete it or rewrite it. A hollow test suite is worse than no test suite — it creates false confidence.

Checkpoint: If you introduced a deliberate bug into each function, would at least one test catch it?

### 6. Check test quality before committing

Read the test file before marking the work done:

- Test names describe the behaviour being tested, not the implementation (`"returns zero for empty cart"` not `"test calculateTotal"`)
- Each test has one assertion focus (multiple unrelated assertions in one test make failure diagnosis harder)
- Setup is minimal — tests don't build more state than they need
- No copy-pasted blocks that could be a `describe` group or shared fixture

---

## Exit criteria

- [ ] Target scope defined before starting (which files and which functions)
- [ ] Agent described the code before writing tests — description reviewed and correct
- [ ] Test cases listed and agreed on before any code written
- [ ] Tests written one function at a time, each verified to pass
- [ ] At least one test per function will fail if the function is broken
- [ ] Test names describe behaviour, not implementation
- [ ] Full test suite passes with no regressions to existing tests

---

## Common shortcuts to avoid

**"Write tests for this whole file."** Broad prompts produce broad tests — mostly happy-path assertions that pass even on broken code. Scope to one function at a time.

**"The tests all pass, we're done."** Passing tests prove the agent ran the tests. They don't prove the tests are correct. Always verify that at least one test would catch a real bug.

**"I'll review the tests later."** Test quality degrades fast if you don't review it at write time. A bad test is harder to remove than to not write in the first place — it feels like you're deleting coverage.
