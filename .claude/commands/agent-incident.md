# /agent-incident

Respond to a production incident with a coding agent: stabilize first, gather a timeline, narrow scope carefully, avoid risky broad fixes under pressure, and capture postmortem notes.

**When to use:** Something is broken in production right now. Users are affected. You need to diagnose and stabilize quickly without making things worse.

**How this differs from `/agent-bugfix`:** Incident response prioritizes stabilization over a clean fix. Speed and reversibility matter more than code quality. The permanent fix comes later.

---

## Steps

### 1. Stabilize before investigating

The first priority is stopping the bleeding — not finding the root cause. Ask:

- Can you roll back the last deploy? If yes, do it immediately.
- Can you disable the broken feature flag? If yes, do it.
- Can you route around the broken service temporarily?

```
Before any investigation: is there a rollback or feature flag that can stabilize the situation right now?
Do not investigate root cause until the immediate user impact is reduced.
```

A rollback that takes 5 minutes is almost always better than a 45-minute investigation and hotfix. Stabilize first, understand later.

Checkpoint: Is user impact reduced or contained? Only proceed to investigation once it is.

### 2. Gather facts — do not theorize yet

Once stable (or while stabilizing, if the incident is ongoing), collect the timeline:

```
Build a timeline of events. Do not speculate on cause yet — only record facts.

- When did the incident start? (first alert, first user report)
- What was the last deploy before the incident?
- What changed in the last 24 hours? (deploys, config changes, traffic patterns, upstream dependencies)
- What are the current error rates and which services are affected?
- What do the logs show? (errors, unusual patterns, timeouts)
```

Fact collection and hypothesis formation are separate phases. Mixing them leads to premature root cause assumptions and missed evidence.

### 3. Narrow the blast radius

Before searching for a fix, establish the scope:

- Which users are affected? (all, a subset, a specific region, a specific feature)
- Which requests are failing? (all endpoints, one endpoint, one operation type)
- Is it getting worse, stable, or improving?

```
Based on the logs and error rates, narrow down:
- What percentage of requests are affected?
- Is the failure deterministic (always) or probabilistic (sometimes)?
- Are there any requests that are succeeding that shouldn't be (or vice versa)?
```

Narrower scope = narrower fix = lower risk of making things worse.

### 4. Form a hypothesis — and challenge it

Once you have facts and scope:

```
Based on what we've collected, what is your current hypothesis for the root cause?
What evidence supports it?
What evidence would disprove it?
What else could explain the same symptoms?
```

Pressure during incidents drives premature conclusions. The agent (and you) will want to act. Make sure the hypothesis holds up before moving to a fix.

### 5. Apply the smallest possible fix

An incident hotfix is not the time for refactoring, cleanup, or addressing adjacent issues. The fix should be:

- **Minimal:** The smallest change that stops the bleeding
- **Reversible:** Possible to roll back quickly if the fix makes things worse
- **Surgical:** Affects only the failing code path

```
Implement the minimum change to address [specific root cause].
Do not touch anything outside the identified failure path.
This fix does not need to be elegant — it needs to be safe and reversible.
```

If the minimum fix is still large or risky, prefer a rollback or feature flag over the code change.

### 6. Verify before and after

Before deploying a hotfix:
- Run the test suite (even quickly, even just the relevant subset)
- Have a second person read the diff if possible
- Confirm the rollback plan if this fix also fails

After deploying:
- Watch error rates and logs for at least 10 minutes
- Confirm affected users are no longer experiencing the issue
- Don't declare resolved until metrics confirm it

### 7. Capture postmortem notes while fresh

Within an hour of resolution, write down:

```markdown
## Incident notes [date]

**What happened:** [one paragraph: symptoms and user impact]
**Timeline:** [bulleted list of timestamped events]
**Root cause:** [what actually caused it]
**Fix applied:** [what was changed and why]
**Detection gap:** [why didn't we catch this before it hit production?]
**Follow-up actions:** [permanent fix ticket, alerting improvement, test to add]
```

These notes are written now, while the details are fresh. The formal postmortem can be written later from these notes. Do not skip this step because the incident is resolved — the learnings are the point.

---

## Exit criteria

- [ ] Stabilization attempted before investigation (rollback, feature flag, or routing change)
- [ ] Timeline of facts gathered before hypothesis formed
- [ ] Blast radius determined: which users and requests are affected
- [ ] Hypothesis challenged — what evidence would disprove it?
- [ ] Fix is minimal, surgical, and reversible
- [ ] Error rates and logs verified to confirm resolution
- [ ] Postmortem notes captured within one hour of resolution
- [ ] Follow-up tickets created for permanent fix and detection improvements

---

## Common shortcuts to avoid

**"Let's find the root cause before rolling back."** Every minute of investigation is a minute users are experiencing the incident. Stabilize first — root cause analysis can happen after.

**"The fix is obvious, I'll skip the hypothesis step."** Incidents have many potential causes that look similar on the surface. "Obvious" root causes are often wrong. The hypothesis step takes five minutes and prevents a wrong fix that makes things worse.

**"I'll fix several things at once while I'm in here."** One change at a time. If the fix makes things worse, you need to know which change caused it. Multiple simultaneous changes make that impossible.

**"I'll write the postmortem later."** Later is never. Incident details fade within hours. Write the notes immediately after resolution while context is fresh — the formal document can be formatted later.
