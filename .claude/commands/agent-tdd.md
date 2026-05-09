# /agent-tdd

Drive new code with unit tests using a coding agent: write the test first, implement the minimum code to pass it, then refactor — keeping the red-green-refactor loop intact.

**When to use:** Writing new functions or modules where the behaviour can be specified upfront, or when you want the tests to drive the design of the implementation rather than follow it.

**How this differs from `/agent-test-gen`:** `/agent-test-gen` adds tests to existing code. This skill uses tests to create new code that doesn't exist yet.

---

## Steps

### 1. Specify the behaviour before writing anything

Before any code or tests, write a plain-language description of what the unit should do:

```
Write a function `calculateShippingCost(weightKg, distanceKm, express)` that:
- Returns cost in cents (integer)
- Base rate: 50 cents per kg plus 10 cents per km
- Express multiplier: 1.5x
- Minimum charge: 500 cents
- Throws if weight or distance is negative
```

The more specific you are here, the less you'll need to correct later. Ambiguity in the spec becomes ambiguity in the tests.

Checkpoint: Does your spec cover the happy path, edge cases, and error cases?

### 2. Write the failing tests — no implementation yet

Ask the agent to write the tests only:

```
Write unit tests for `calculateShippingCost` based on this spec.
Do not write the implementation yet.
The tests should fail because the function doesn't exist.
```

Review the tests before running them. Check that:
- Each test case maps to a clause in the spec
- Edge cases from the spec are represented (minimum charge, negative inputs)
- Test names describe the behaviour (`"applies express multiplier"`, not `"test express"`)
- No test cheats by hardcoding the expected output of a not-yet-written function

Run the tests. They should all fail. If any pass, the test is wrong — a test for non-existent code cannot legitimately pass.

Checkpoint: All tests fail. If a test passes, investigate and fix it before continuing.

### 3. Write the minimum implementation to pass the tests

Now implement:

```
Write the implementation of `calculateShippingCost` to make the failing tests pass.
Write the minimum code that satisfies the tests — do not add behaviour the tests don't cover.
```

"Minimum code" is a constraint, not a suggestion. If the agent adds error handling, caching, or logic not required by a test, ask it to remove it. Untested code added "while we're here" is untested code.

Run the tests. They should all pass.

Checkpoint: All tests pass. Implementation does not contain logic that no test covers.

### 4. Refactor — separately from making tests pass

With tests green, refactor the implementation for readability and structure:

```
The tests pass. Now refactor the implementation for clarity.
Do not change any behaviour — if a test breaks, the refactor went too far.
Run the tests after each change.
```

This is the step most agents skip — they write passing code and stop. The refactor step is where good design emerges. Common refactors at this stage:
- Extract a named constant for magic numbers
- Extract a helper function that clarifies intent
- Rename variables to match the domain language

Run the tests after refactoring. All should still pass.

### 5. Add cases the first test pass missed

After the initial red-green-refactor cycle, ask:

```
Are there any cases in the spec that aren't covered by the tests we have?
Are there any inputs where the behaviour is unspecified or surprising?
```

Write one more test for any gap found, then make it pass, then refactor again. One cycle per gap.

Stop when every clause in your spec has a corresponding test.

### 6. Review the final test file

Before committing:

- Each test name reads as a statement of behaviour
- No test duplicates another (different assertion, not just different variable name)
- Tests are independent — no test depends on another running first
- No implementation detail leaks into the test (testing the function's contract, not its internals)

---

## Exit criteria

- [ ] Behaviour specified in plain language before any code written
- [ ] Tests written and verified to fail before implementation started
- [ ] Implementation written to pass tests — no untested logic added
- [ ] Tests pass after implementation
- [ ] Refactor done as a separate step with tests staying green
- [ ] Every clause in the original spec has a corresponding test
- [ ] Tests are independent and describe behaviour, not implementation

---

## Common shortcuts to avoid

**"Let the agent write the tests and implementation together."** This collapses TDD into regular development. The agent will write implementation it already has in mind and write tests to match. You lose the design pressure that comes from writing the test first.

**"Some tests passed before the implementation — that's fine."** It's not. A test that passes before the code exists either tests something trivial or is incorrect. Investigate every premature pass.

**"Skip the refactor — the tests pass, we're done."** Passing tests mean the code is correct. They say nothing about readability. The refactor step is where the code becomes maintainable. Skip it and you accumulate the kind of debt that makes future features harder.

**"Add the extra error handling while we're implementing."** Only implement what a test requires. If the error handling matters, write the test for it first. Untested logic is a liability, not a feature.
