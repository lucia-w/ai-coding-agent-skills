# /agent-docs

Update documentation after code changes with a coding agent: identify what changed, find every affected doc location, update each one, and verify examples still work.

**When to use:** After shipping a feature, bugfix, or API change — before closing the ticket. Also use when an agent shipped code without updating docs and you're catching up.

---

## Steps

### 1. Identify what changed that affects documentation

Start from the diff, not from memory:

```
Read the diff from main (or the PR diff).
List every change that could affect documentation:
- New or removed public API methods, endpoints, or props
- Changed behaviour (different defaults, different error messages, different return shapes)
- New required configuration or environment variables
- Deprecated or removed features
- Changed installation or setup steps
```

Don't assume you know what changed. Read the diff and derive the doc impact from it.

Checkpoint: Do you have a list of specific changes that need to be documented?

### 2. Find every doc location that needs updating

Code changes can impact multiple places. Ask the agent to find them all:

```
Given these changes, find every file that might need updating:
- README.md (setup, usage examples, feature list)
- API reference docs (if separate from code)
- Inline code comments and JSDoc/docstrings on changed functions
- CHANGELOG.md
- Migration guides or upgrade notes
- Any example apps or `examples/` directory
```

Different projects have different doc locations. Make sure the agent checks all of yours, not just the obvious ones.

Checkpoint: Is there any doc location the agent missed that you know exists?

### 3. Update each location — one at a time

Don't update all docs in a single prompt. Do them one by one so each update can be reviewed:

```
Update the README to reflect [specific change].
Show me the diff before we move to the next doc location.
```

For each location, verify:
- The updated text accurately reflects the new behaviour (not the old one)
- No outdated information was left alongside the new information
- The tone and style matches the surrounding docs

### 4. Verify code examples still work

Docs with broken examples are worse than no docs — they erode trust and waste reader time.

For every code example in the updated docs, verify it:

```
For each code example in the updated docs, confirm it works against the current version of the code.
If an example would fail or produce wrong output, update it.
```

Treat broken examples as bugs. Fix them in this PR, not in a follow-up.

### 5. Update the changelog

Every user-visible change should have a changelog entry:

```markdown
## [Unreleased]

### Added
- `calculateShippingCost` now accepts an `express` flag for expedited shipping rates

### Changed
- Default timeout increased from 5s to 10s

### Deprecated
- `address` field on Order — use `shipping_address` instead. Removed in v3.

### Fixed
- Shipping cost calculation no longer rounds incorrectly for small weights
```

Use [Keep a Changelog](https://keepachangelog.com) format if your project doesn't already have a convention. The agent can draft entries, but review them — changelog language is user-facing and should be clear to someone who didn't write the code.

### 6. Check for docs that should be deleted

Code removal often leaves orphaned docs behind. If a feature was removed:

```
Was any documentation written for the feature or API that was removed in this diff?
If so, find and remove it.
```

Outdated docs that aren't deleted actively mislead users. Removal is as important as addition.

---

## Exit criteria

- [ ] Diff read to identify all doc-impacting changes (not recalled from memory)
- [ ] All affected doc locations identified: README, API docs, inline comments, CHANGELOG, examples
- [ ] Each location updated with accurate information — no outdated text left alongside new
- [ ] Code examples verified to work against the current codebase
- [ ] Changelog entry written for every user-visible change
- [ ] Orphaned docs for removed features deleted

---

## Common shortcuts to avoid

**"The code is self-documenting."** For internal code, maybe. For public APIs, configuration, and setup steps, it isn't. If a user has to read your source code to use your project, the docs are missing.

**"I'll update the docs in a follow-up PR."** Follow-up doc PRs have a low completion rate. Update the docs in the same PR as the code, while the context is fresh.

**"The agent can write the docs autonomously."** The agent can draft them. Changelog entries, user-facing descriptions, and example code all need human review — you know your users better than the agent does.

**"I only need to update the README."** Code changes ripple into API references, inline comments, examples, and changelogs. Start from a full audit, not an assumption about where the doc impact is.
