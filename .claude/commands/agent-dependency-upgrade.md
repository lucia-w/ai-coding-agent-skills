# /agent-dependency-upgrade

Upgrade dependencies safely with a coding agent: read the changelog, update one package at a time, verify tests after each, and avoid drive-by upgrades of unrelated packages.

**When to use:** Routine dependency maintenance, addressing a security advisory, or upgrading a dependency to access a new feature you need.

---

## Steps

### 1. Define the scope before touching anything

Name the packages you're upgrading and why. Do not let the agent upgrade everything it finds:

```
Upgrade [package-name] from [current-version] to [target-version].
Reason: [security advisory / need feature X / routine maintenance]

Do not upgrade any other packages.
```

Drive-by upgrades of unrelated packages mix unrelated risk into a single change, making it harder to diagnose regressions and harder to revert a specific upgrade if needed.

### 2. Read the changelog before upgrading

Ask the agent to read the package's changelog between the current version and the target:

```
Read the changelog for [package-name] from [current-version] to [target-version].
List:
- Breaking changes
- Deprecated APIs we might be using
- Any behaviour changes relevant to how we use this package
```

If the package doesn't publish a changelog, read the release notes or GitHub tags. If you can't find what changed, be more cautious about the upgrade — run the full test suite and review the diff carefully.

Checkpoint: Do you know what changed and whether any breaking changes affect your usage?

### 3. Check your current usage before upgrading

Before making any change:

```
Search the codebase for all usages of [package-name].
List the APIs, methods, and configuration options we use.
Cross-reference against the breaking changes you found in the changelog.
```

This identifies exactly what needs updating — before you're in the middle of a broken state.

### 4. Update one package at a time

Install the upgrade:

```bash
npm install [package-name]@[target-version]
# or: pip install [package-name]==[target-version]
# or: go get [module]@[version]
```

Do not run `npm update`, `pip install --upgrade`, or any command that upgrades multiple packages. Update the named package only.

After upgrading, check what else changed in the lockfile:

```bash
git diff package-lock.json  # or yarn.lock, requirements.txt, go.sum
```

If the lockfile shows unexpected packages being upgraded (transitive dependencies changing version), note them. They're usually benign but worth knowing.

### 5. Fix compilation and type errors

After the upgrade, check types and compilation before running tests:

```bash
npm run typecheck
npm run build
```

Fix any errors introduced by the upgrade. Refer to the breaking changes you noted in step 2.

For TypeScript: breaking changes often show up as type errors before tests fail. Fix type errors first — they'll point you to the call sites that need updating.

### 6. Run the full test suite

```bash
npm test
```

Run all tests, not just the tests related to the upgraded package. Dependency upgrades can have unexpected effects on code you didn't expect to be affected.

If tests fail:
- Is the failure in code that uses the upgraded package? Likely a breaking change you need to handle.
- Is the failure in seemingly unrelated code? A transitive dependency may have changed — investigate.

Checkpoint: All tests pass after the upgrade.

### 7. Verify the upgrade addresses the original reason

If the upgrade was for a security advisory:
```bash
npm audit  # or equivalent
```
Confirm the advisory is resolved.

If the upgrade was for a new feature: verify the feature works as expected.

### 8. Write a focused PR description

```markdown
## What

Upgrades [package] from [old] to [new].

## Why

[security advisory link / feature needed / routine maintenance]

## Breaking changes addressed

- [change] — updated [file] to use new API
- (none)

## How to verify

Run `npm test`. Run `npm audit` to confirm advisory is cleared.
```

---

## Exit criteria

- [ ] Scope defined: named packages only, no drive-by upgrades
- [ ] Changelog read: breaking changes and relevant behaviour changes identified
- [ ] Current usage audited before upgrading
- [ ] One package upgraded at a time — lockfile changes reviewed
- [ ] Type errors and compilation errors fixed before running tests
- [ ] Full test suite passes after upgrade
- [ ] Original reason for upgrade verified (advisory resolved, feature works)
- [ ] PR describes what was upgraded, why, and what breaking changes were handled

---

## Common shortcuts to avoid

**"Upgrade everything that's outdated while we're at it."** Every additional upgrade is additional risk. Keep upgrades focused — one concern per PR.

**"The tests pass so the upgrade is safe."** Tests verify tested behaviour. They don't verify your test coverage. If coverage is low in the affected areas, read the diff carefully even if tests pass.

**"Skip the changelog — it'll just work."** Sometimes it does. When it doesn't, having read the changelog beforehand saves significant debugging time. The changelog read is cheap.

**"Lock file changes are just noise."** Unexpected lockfile changes (transitive dependency upgrades) occasionally introduce regressions. Note them; don't ignore them.
