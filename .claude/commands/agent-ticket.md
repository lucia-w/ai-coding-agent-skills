# /agent-ticket

Write an issue or ticket that a coding agent can execute without ambiguity — clear enough that the agent produces a correct first attempt rather than needing repeated clarification.

**When to use:** Before assigning a ticket to an agent, when a teammate asks how to write better agent tickets, or when an agent keeps asking clarifying questions on simple tasks.

---

## Steps

### 1. State the outcome, not the approach

Describe what done looks like, not how to get there. The agent decides the approach; your job is to specify the result.

Good:
```
Users with the "viewer" role should not be able to access /admin routes.
Attempting to access /admin should redirect to /dashboard with a 403 toast message.
```

Vague:
```
Fix the admin permissions.
```

Checkpoint: If you read the ticket without knowing anything else about the codebase, is it clear what success looks like?

### 2. Include explicit acceptance criteria

Write a checklist of verifiable conditions. The agent uses this to know when to stop and to self-verify before handing back.

```markdown
## Acceptance criteria

- [ ] GET /admin returns 403 for users with role "viewer"
- [ ] Viewer users are redirected to /dashboard after the 403
- [ ] A toast notification shows "Access denied" on redirect
- [ ] Admin users are unaffected and can still access /admin
- [ ] Existing tests pass
- [ ] New test added for the viewer redirect case
```

Vague acceptance criteria like "it works correctly" or "no regressions" are not checkable — be specific.

### 3. Point to relevant files and locations

Don't make the agent search the whole codebase. Link to what it needs:

```markdown
## Relevant files

- Auth middleware: `middleware/auth.ts`
- Role definitions: `lib/roles.ts`
- Admin route config: `app/admin/layout.tsx`
- Existing auth tests: `__tests__/auth.test.ts`
```

If you don't know the exact files, mention the area: "Somewhere in the auth middleware" is still useful.

### 4. Specify constraints and what NOT to do

Coding agents often solve problems in unexpected ways. If certain approaches are off-limits, say so:

```markdown
## Constraints

- Do not change the User schema or any database migrations
- Use the existing `useToast` hook — don't add a new toast library
- The fix should work without changing the /admin page components directly
```

### 5. Include the why (briefly)

Context helps the agent make better tradeoffs. One sentence is enough:

```markdown
## Context

Viewer accounts are shared with external contractors. Admin routes expose internal pricing data that contractors must not see.
```

### 6. Specify the expected deliverable

Be explicit about what you expect back:

```markdown
## Deliverable

A pull request with:
- The fix
- At least one new test covering the redirect behaviour
- A brief explanation in the PR description of the approach taken
```

### 7. Test your ticket before assigning it

Read it back and ask: could a skilled contractor execute this ticket with no back-and-forth? If not, it needs more work.

Things that reliably cause confusion:
- Jargon specific to your team without explanation
- References to Slack conversations or meetings ("as discussed")
- Conflicting requirements in different parts of the ticket
- Missing information about the current broken state

---

## Exit criteria

- [ ] Outcome is described, not the approach
- [ ] Acceptance criteria is a verifiable checklist (not "works correctly")
- [ ] Relevant files or areas of the codebase are named
- [ ] Constraints are explicit (what not to change)
- [ ] Context (the why) is included in one sentence
- [ ] Expected deliverable is specified (PR, test, explanation)
- [ ] No references to out-of-band context (Slack, meetings)

---

## Common shortcuts to avoid

**"The agent will figure out the details."** It will. But it may figure them out wrong. Specificity in the ticket is cheaper than correction cycles after.

**"We write tickets for humans, not agents."** A ticket good enough for an agent is also better for a human. The clarity requirements are the same; agents are just less forgiving.

**"The acceptance criteria will be obvious from the description."** Write them anyway. Explicit criteria are the agent's stopping signal — without them, it may over-engineer or stop too early.
