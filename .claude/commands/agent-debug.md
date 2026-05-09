# /agent-debug

Investigate an unknown problem with a coding agent: reproduce it reliably, gather evidence, bisect the cause, and produce a diagnosis — before any fix is written.

**When to use:** Something is broken but you don't know why. The cause isn't obvious from the report. Use this skill to find the root cause; use `/agent-bugfix` once you know what to fix.

**How this differs from `/agent-bugfix`:** `/agent-bugfix` assumes you know the cause and are implementing a fix. This skill is the investigation that happens before that.

---

## Steps

### 1. Write down what you know and what you don't

Before the agent touches anything, structure your unknowns:

```
What we know:
- [symptom, when it started, who reported it]

What we don't know:
- Root cause
- Whether it's deterministic or intermittent
- What changed recently that could explain it

What we've already ruled out:
- [anything you've checked]
```

This prevents the agent from re-investigating things you've already ruled out, and keeps the investigation focused.

### 2. Reproduce the problem before doing anything else

Ask the agent to reproduce — not investigate, not fix:

```
Read the bug report. Do not change any code.
Tell me: can you reproduce this locally? What exact steps trigger it?
```

If the agent cannot reproduce it, stop. An unreproduced bug cannot be debugged. Gather more information — logs, environment details, steps — and try again.

Checkpoint: Is the problem reproducible on demand? If not, is it reproducible with known frequency?

### 3. Gather evidence — logs, traces, state

With reproduction in hand, collect everything relevant:

```
Reproduce the issue and capture:
- Full error output and stack trace
- Relevant log lines (expand the window — the error line is often not the cause)
- Application state at the time of failure (which user, which input, which request)
- Any network requests or database queries involved
```

Ask the agent to read logs literally, not interpret them yet. Interpretation comes after collection. Agents often jump to conclusions from partial evidence.

### 4. Narrow the scope

Use elimination to shrink the problem surface:

```
Given the stack trace, which layer is the failure in: data access, business logic, or presentation?
What's the smallest code path that triggers the problem?
```

Bisection techniques depending on context:

- **Git bisect** — if it worked before and is broken now, find the commit that introduced it:
  ```bash
  git bisect start
  git bisect bad HEAD
  git bisect good <last-known-good-commit>
  ```
- **Comment out / stub** — temporarily replace components to isolate which one is responsible
- **Minimal reproduction** — reduce to the smallest possible input that still triggers the failure

Checkpoint: Can you point to a specific function, query, or commit as the likely cause?

### 5. Form and test a hypothesis

Once the scope is narrow, form a hypothesis:

```
State your hypothesis: what do you think is causing this and why?
Then test it: what would be true if your hypothesis is correct, and how can we verify that?
```

A hypothesis without a test is a guess. The agent should be able to say: "If I'm right, then X will be true — let me check." If a hypothesis can't be tested, form a more specific one.

Do not accept "it might be X" — ask for a concrete test of the hypothesis.

### 6. Document the diagnosis

Once the cause is confirmed:

```markdown
## Diagnosis

**Symptom:** [what the user sees]
**Root cause:** [what's actually wrong and why]
**Evidence:** [log lines, commit, test that confirms it]
**Scope:** [how many users/requests/records are affected]
**Introduced by:** [commit or PR if known]
```

### 7. Decide the next step

A debug session ends with a decision, not a fix:

- **Open a fix ticket:** If the fix is non-trivial, hand off the diagnosis to `/agent-bugfix` with the full diagnosis doc attached
- **Fix immediately:** Only if the fix is one or two lines and the risk of regression is very low
- **Escalate:** If the cause is outside your codebase (upstream library, infrastructure, third party)

Do not let the agent start fixing during the debug session. Investigation and implementation are separate contexts.

---

## Exit criteria

- [ ] Problem reproduced reliably before investigation started
- [ ] Evidence collected: stack trace, logs, state at time of failure
- [ ] Scope narrowed to a specific function, module, or commit
- [ ] Hypothesis formed and tested — not just stated
- [ ] Diagnosis documented: symptom, root cause, evidence, scope
- [ ] Next step decided: fix ticket opened, or fix deferred to `/agent-bugfix`

---

## Common shortcuts to avoid

**"The stack trace points to X, so X is the cause."** The stack trace shows where the error surfaced, not where it originated. Treat it as a starting point, not a conclusion.

**"I'll fix it while I'm investigating."** Investigation context and fix context are different. Fixing during debugging mixes two different tasks, produces rushed fixes, and leaves no documentation of what was found. Finish the diagnosis first.

**"I can't reproduce it, but I have a theory."** An unreproduced bug leads to a guessed fix. Guessed fixes either don't work or fix something adjacent. Reproduction is not optional.

**"Let the agent search the whole codebase for the cause."** Broad codebase searches produce noisy results. Narrow the scope first — layer, module, commit range — then search within that scope.
