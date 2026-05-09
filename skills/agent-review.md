# /agent-review

Run a structured code review with a coding agent: catch mechanical issues automatically, surface judgment calls for human review, and produce a clear review comment.

**When to use:** Before posting a human review on a PR, as a pre-review pass on your own code, or to get a second opinion on a diff.

---

## Steps

### 1. Scope the review

Before starting, define what kind of review this is:

| Review type | Focus |
|-------------|-------|
| Correctness | Does it do what it claims? Are there logic errors or edge cases? |
| Security | Are there injection risks, auth gaps, or exposed secrets? |
| Performance | Are there N+1 queries, unnecessary re-renders, or blocking calls? |
| Style | Does it follow conventions? Is it readable? |
| Full | All of the above |

Agents are good at correctness, security, and style. They are less reliable on performance (requires system-level knowledge) and product judgment ("is this the right feature?").

### 2. Provide the diff and context

Give the agent everything it needs:

```
Review this PR diff. The PR is: [one-line description of what it does].

Context:
- This code runs on every API request, so performance matters
- The auth model uses JWTs stored in httpOnly cookies
- We use Zod for all input validation

Diff:
[paste diff here or reference the PR]
```

The more context you provide, the more accurate the review. "Review this" with no context produces generic comments.

### 3. Run the mechanical checks first

Ask the agent to flag:
- Missing input validation at API boundaries
- Unhandled promise rejections or missing error handling
- Secrets, tokens, or credentials in the code
- Missing or incorrect types (TypeScript)
- Dead code or unused variables
- Functions that are too long or too complex to read

These are the things a good linter would catch but often doesn't. The agent should comment on each finding with file and line reference.

Checkpoint: Are there any blocking issues from the mechanical pass?

### 4. Check for correctness

Ask the agent to verify the logic:

```
Read the PR description and the diff. Does the implementation match the intended behaviour?
Are there edge cases the implementation doesn't handle?
```

Common correctness gaps:
- Off-by-one errors in loops or pagination
- Incorrect operator precedence
- Race conditions in async code
- Missing null/undefined checks
- Wrong comparison (reference vs value equality)

### 5. Check for security issues

For any PR that touches auth, data handling, or external input:

```
Review this diff for security issues. Specifically:
- Is any user input used without sanitization or validation?
- Are there SQL queries built with string concatenation?
- Are there any new API endpoints that don't check authentication?
- Are any secrets or tokens logged or returned in responses?
```

### 6. Identify what needs human judgment

The agent should flag, not decide, on:
- **Product decisions:** "This changes the error message users see — is that intentional?"
- **Architecture decisions:** "This introduces a new caching layer — has the team aligned on this?"
- **Risk assessment:** "This migrates the auth flow — needs QA and possibly a feature flag"

These are escalation flags for you, not for the agent to resolve.

### 7. Write the review comment

Ask the agent to produce a structured review:

```markdown
## Review summary

[1-2 sentences: overall assessment]

## Blocking issues

[Must fix before merge]
- line X: [issue and suggested fix]

## Non-blocking suggestions

[Nice to have, but not required]
- line Y: [suggestion]

## Questions for the author

[Things that need clarification or human judgment]
- [question]
```

---

## Exit criteria

- [ ] Review scope defined (correctness, security, style, or full)
- [ ] Mechanical checks complete (validation, error handling, types, secrets)
- [ ] Logic correctness checked against PR description
- [ ] Security pass done for auth/data-handling changes
- [ ] Judgment calls flagged for human review (not decided by agent)
- [ ] Review comment structured as: blocking issues / suggestions / questions

---

## Common shortcuts to avoid

**"The agent approved it so it's fine."** Agent approval is a signal, not a guarantee. The agent cannot assess product correctness, team conventions from memory, or cross-repo implications.

**"I'll skip the security pass — it's just a UI change."** UI changes often include new API calls, new data displayed, or new input fields. Always run the security pass.

**"I'll let the agent decide if this should be merged."** Merging is a human decision. The agent surfaces issues; you decide what to do with them.
