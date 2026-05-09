# /agent-security-review

Run a focused security review of a diff or codebase with a coding agent: check authentication and authorization, secrets and credential handling, injection points, dependency risk, and sensitive data exposure.

**When to use:** Before merging any PR that touches auth, data handling, external inputs, or dependencies. Also use as a standalone audit pass on an existing codebase.

**How this differs from `/agent-review`:** `/agent-review` covers correctness, style, and general quality. This skill is focused exclusively on security — it goes deeper on each security category and applies to code that `/agent-review` wouldn't flag.

---

## Steps

### 1. Scope the review

Security review has different depth requirements depending on what changed. Before starting:

| Change type | Security focus |
|-------------|----------------|
| Auth flow (login, logout, token refresh) | Full pass on all categories |
| New API endpoint | AuthN/AuthZ, input validation, output sanitization |
| New dependency added | Dependency risk, known CVEs |
| Data access layer | SQL injection, over-fetching, logging of sensitive fields |
| Frontend change with user input | XSS, CSRF, input handling |
| Config or env var change | Exposed secrets, insecure defaults |

Tell the agent what changed and which categories apply.

### 2. Check authentication and authorization

The most common class of security bug is: an action that should be restricted isn't.

```
Review [the diff / these files] for authentication and authorization issues:
- Are there any new API endpoints, routes, or actions that don't verify the caller is authenticated?
- Are there any endpoints that authenticate (know who the caller is) but don't authorize (check what they're allowed to do)?
- Are there any admin or privileged actions that check role only on the client side?
- Are there any authentication bypass conditions (unauthenticated paths, fallback behaviours, error states that skip checks)?
```

Pay special attention to:
- Middleware that only applies to some routes
- Error handling that bypasses auth checks
- Admin features protected by UI only (no server-side check)

### 3. Check for secrets and credential handling

```
Scan for secrets and credential handling issues:
- Are any secrets, API keys, tokens, or passwords hardcoded in source files?
- Are any secrets written to logs?
- Are any secrets returned in API responses?
- Are environment variables used correctly (read at startup, not committed as values)?
- Are there any new files that should be in .gitignore (e.g., .env, credential files)?
```

Look at log statements carefully. Logging `user` objects or `request` headers is a common way to accidentally log auth tokens or session data.

### 4. Check for injection vulnerabilities

```
Check for injection vulnerabilities:
- Are there SQL queries built with string concatenation or interpolation rather than parameterized queries?
- Are there shell commands constructed from user input?
- Is user input rendered in HTML without sanitization (XSS)?
- Are there HTTP requests made to URLs derived from user input without validation (SSRF)?
- Is `eval()` or equivalent used on any user-controlled data?
```

For each finding, the agent should identify the specific line and the user-controlled input that reaches it.

### 5. Check input validation at trust boundaries

User input should be validated at the point it enters your system — not deep in business logic:

```
Identify every place where external input enters the system:
- API request bodies, query params, headers
- File uploads
- Webhook payloads
- Data from third-party APIs

For each entry point: is the input validated for type, shape, and allowed values before it's used?
```

Validation deep in the call stack (after the data has been stored, processed, or logged) is too late.

### 6. Assess dependency risk

For any PR adding or upgrading dependencies:

```
For each new or upgraded dependency:
- Does it have known CVEs? Check: npm audit, pip-audit, or Snyk
- Is it actively maintained? (last commit, number of open issues)
- Does it request unusual permissions or access? (for native extensions, browser extensions)
- Is it from a trusted publisher? (typosquatting is common — verify the exact package name)
```

A dependency is code you didn't write that runs in your process. Treat it with proportionate skepticism.

### 7. Check for sensitive data in logs and responses

Data exposure is often subtle — not a dramatic leak, but a field that shouldn't be included:

```
Check for sensitive data exposure:
- Are any of these fields returned in API responses: passwords, tokens, full PII, internal IDs not meant for clients?
- Are any of these fields written to logs: auth tokens, session IDs, credit card numbers, SSNs?
- Are error messages returned to clients detailed enough to aid an attacker? (stack traces, internal paths, DB errors)
- Are responses filtered by the caller's permission level, or do they return the full object and rely on the client to hide fields?
```

### 8. Write a structured security report

```markdown
## Security review: [PR title or scope]

### Critical (block merge)
- [finding]: [file:line] — [what the issue is and why it's risky]

### High (fix before merge)
- [finding]: [file:line] — [issue and recommendation]

### Medium (fix soon)
- [finding] — [issue and recommendation]

### No issues found
- Authentication and authorization: ✓
- Secrets and credential handling: ✓
- Injection: ✓
- Input validation: ✓
- Dependency risk: ✓
- Sensitive data exposure: ✓
```

---

## Exit criteria

- [ ] Review scoped to the relevant security categories for this change
- [ ] Authentication checked: all new endpoints verify caller identity
- [ ] Authorization checked: privilege checks are server-side and can't be bypassed
- [ ] No secrets in source, logs, or responses
- [ ] Injection points checked: SQL, shell, HTML, SSRF
- [ ] Input validated at trust boundaries — not deep in business logic
- [ ] New dependencies checked for known CVEs and trustworthiness
- [ ] Sensitive data fields audited in logs and API responses
- [ ] Findings categorized: critical / high / medium — with file and line references

---

## Common shortcuts to avoid

**"It's just a UI change — skip the security pass."** UI changes often include new API calls, new user inputs, and new data displayed. Run at minimum the input validation and data exposure checks.

**"The agent found nothing — it's clean."** The agent cannot assess business logic authorization (is this user allowed to see this record?), only structural auth patterns. Review privilege checks that involve business rules manually.

**"We'll fix the medium findings later."** Medium findings have a way of becoming high findings when combined with other changes. Log them, create tickets, and resolve them — don't let them accumulate.

**"The dependency is popular, so it's safe."** Popular packages are high-value targets. Always run an audit check, regardless of how well-known the package is.
