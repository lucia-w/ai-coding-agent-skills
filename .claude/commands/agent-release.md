# /agent-release

Prepare a release with a coding agent: compile the changelog, bump the version, write release notes, add migration notes, and verify CI is green — without publishing autonomously.

**When to use:** Cutting a new version of a library, service, or app. Also useful for preparing a release PR that a human will merge and publish.

**Hard rule:** The agent never publishes, deploys, or pushes a release tag without explicit human approval at each step. Releasing is irreversible.

---

## Steps

### 1. Confirm the release branch and CI status

Before preparing anything:

```bash
git status
git log --oneline main..HEAD
```

Confirm:
- You're on the correct branch (typically `main` or `release/x.y.z`)
- CI is green on the current HEAD — no failing checks
- There are no uncommitted changes

If CI is not green, stop. Do not prepare a release from a failing build.

Checkpoint: Is CI green on the intended release commit?

### 2. Determine the version bump

Decide the new version before the agent touches anything. Use semantic versioning:

| Change type | Version bump | Example |
|-------------|--------------|---------|
| Breaking change (incompatible API change) | Major | 2.0.0 |
| New feature (backward-compatible) | Minor | 1.3.0 |
| Bug fix or patch (backward-compatible) | Patch | 1.2.4 |

Ask the agent to list all changes since the last release and classify each:

```
Read the git log from the last release tag to HEAD.
List every change and classify each as: breaking, new feature, or bug fix.
Based on this, what version bump is required?
```

Review the classification. The agent may miss subtle breaking changes — especially in type signatures, config defaults, or implicit behaviour changes.

### 3. Compile the changelog

From the git log and PR descriptions:

```
Write a changelog entry for version [X.Y.Z] covering all changes since [last-tag].
Format:
- Added: new features
- Changed: modifications to existing behaviour
- Deprecated: features that will be removed in a future version
- Fixed: bug fixes
- Security: security fixes (always include these)
- Breaking: incompatible changes (prominently, at the top)

Use user-facing language — describe impact, not implementation.
```

Review every entry. Changelog language is public — it's often the first thing a user reads when deciding whether to upgrade. The agent writes from commit messages; you know what matters to users.

### 4. Write migration notes for breaking changes

If this is a major version with breaking changes, write a migration guide:

```markdown
## Migrating from v1 to v2

### Breaking: `calculateShippingCost` signature changed

**Before:**
```js
calculateShippingCost(weight, distance)
```

**After:**
```js
calculateShippingCost({ weightKg, distanceKm, express: false })
```

**Why:** Named parameters make call sites more readable and allow future options without another breaking change.
```

Every breaking change needs: what changed, the before/after, and the reason. Users who can't figure out how to upgrade will downgrade instead.

### 5. Bump the version

Ask the agent to bump the version in all relevant files:

```
Bump the version to [X.Y.Z] in:
- package.json (or pyproject.toml / go.mod / Cargo.toml — wherever the version lives)
- Any lockfiles that need updating
- Any hardcoded version references in docs or README
```

Verify the diff. A version bump should only touch version strings — if the agent changed anything else, investigate.

### 6. Create the release commit and tag — with your approval

Present the full release state for review:

```
Here is the release plan:
- Version: [X.Y.Z]
- Commit: "chore: release vX.Y.Z"
- Tag: vX.Y.Z
- Changelog: [attached]
- Migration notes: [attached or N/A]

Awaiting approval before creating the commit and tag.
```

You review, then explicitly say to proceed. The agent does not create tags autonomously.

After approval:

```bash
git commit -m "chore: release vX.Y.Z"
git tag -a vX.Y.Z -m "Release vX.Y.Z"
```

Do not push yet.

### 7. Push and publish — only with explicit approval

Pushing a tag and publishing to a registry are separate steps, each requiring approval:

```
Step 1: Push commit and tag to remote — awaiting approval.
Step 2: Publish to [npm / PyPI / registry] — awaiting approval.
```

Publish commands are never run without typed approval:

```bash
# Only after explicit approval:
git push origin main
git push origin vX.Y.Z
npm publish  # or equivalent
```

If your release process includes a GitHub Release, create it after the tag is pushed — not before.

---

## Exit criteria

- [ ] CI green on the release commit before starting
- [ ] Version bump type decided and justified (major/minor/patch)
- [ ] Changelog compiled from git history — user-facing language, not commit messages
- [ ] Migration guide written for every breaking change
- [ ] Version bumped in all relevant files — diff contains only version string changes
- [ ] Release commit and tag reviewed and explicitly approved before creation
- [ ] Push and publish each require separate explicit approval
- [ ] No autonomous publishing at any step

---

## Common shortcuts to avoid

**"CI was green yesterday, no need to check."** Check at the commit you're releasing. A merged PR since yesterday could have broken it.

**"The agent can publish when it's ready."** Autonomous publishing is never acceptable. Registry publishes are permanent (or difficult to retract). Every publish requires a human to say "go."

**"The changelog can be generated from commit messages."** Commit messages are for engineers. Changelog entries are for users. They describe impact, not implementation. Always review and rewrite the agent's draft.

**"I'll write the migration guide after the release."** Users need migration guidance the moment they see the major version bump. Write it before publishing, not as a follow-up.
