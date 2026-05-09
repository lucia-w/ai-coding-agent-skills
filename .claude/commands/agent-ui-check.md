# /agent-ui-check

Verify frontend changes with a coding agent: run the app, walk through the changed UI, check responsive states, accessibility basics, and empty/error/loading states — before opening a PR.

**When to use:** After any frontend change — new component, layout change, form, data display, or interaction. Use before requesting review to catch visual and interaction issues that tests can't detect.

---

## Steps

### 1. Run the application

Start with a working local environment:

```bash
npm run dev  # or equivalent
```

Confirm:
- The app starts without errors
- No console errors on page load
- The dev server reflects the current branch's changes

Do not check the UI against a deployed environment or a stale build. Changes must be visible in what you're testing.

Checkpoint: Is the app running locally against the current branch?

### 2. Walk the golden path

Test the primary use case for the changed UI end-to-end, as a user would:

```
Walk me through using [the changed feature] from the entry point to completion.
What does a user do, and what do they see at each step?
```

The agent can describe what should happen — you verify it does. Don't rely on the agent's description as verification; use it as a test script.

If the agent hasn't opened the browser: open it. Click through the flow yourself. The agent can guide; you confirm.

### 3. Test empty, loading, and error states

These states are almost always present but often untested. For each changed component:

**Empty state:**
- What does the UI show when there's no data? (empty list, no results, new user with no history)
- Is the empty state informative, or is it a blank space?

**Loading state:**
- Is there a loading indicator while data fetches?
- Does the layout shift when data loads? (causes visual jarring and CLS)
- What happens if loading takes longer than expected?

**Error state:**
- What does the UI show if the API call fails?
- Is the error message useful to the user, or just "Something went wrong"?
- Can the user retry without refreshing the page?

Ask the agent to trigger each state deliberately — disable the network, return empty data, force an API error.

Checkpoint: Does each state display something intentional, or is it broken/blank?

### 4. Check responsive layouts

Test at the breakpoints your app supports:

```
Check the [component] at:
- Mobile: 375px wide
- Tablet: 768px wide
- Desktop: 1280px wide
```

Using browser DevTools responsive mode is sufficient. Look for:
- Content that overflows its container
- Text that becomes unreadable (too small, overlapping)
- Buttons or links that are too small to tap on mobile
- Tables or wide content that breaks the layout

### 5. Check accessibility basics

Accessibility issues are often invisible during development but block users with assistive technology:

**Keyboard navigation:**
- Can you reach all interactive elements using Tab?
- Is the focus indicator visible at each step?
- Can you activate buttons and links using Enter/Space?

**Screen reader basics:**
- Do images have descriptive `alt` text (or `alt=""` if decorative)?
- Do form inputs have visible labels (not just placeholder text)?
- Are interactive elements (`<button>`, `<a>`) using semantic HTML rather than `<div onClick>`?

**Colour and contrast:**
- Is text readable against its background? (aim for at least 4.5:1 contrast ratio)
- Is colour the only way information is conveyed? (a red error that's also a text message is fine; a red-only indicator is not)

Ask the agent to flag any of these it can detect from the code. Run a browser accessibility checker (Axe DevTools, Lighthouse accessibility audit) for a second pass.

### 6. Check interactions and edge cases

Beyond the golden path, test the edges:

- **Long content:** What happens with a very long name, title, or description? Does it truncate gracefully or break the layout?
- **Fast interactions:** What if the user clicks a button multiple times quickly? Does it submit multiple times?
- **Back button:** Does navigation behave correctly after the user goes back?
- **Refresh:** If the user refreshes mid-flow, is the state preserved or does it reset gracefully?

### 7. Take screenshots for the PR

For any visual change, capture screenshots for the PR description:

- The primary state (what most users see)
- Mobile view (if the layout changes responsively)
- Any state that's hard to describe in words (empty state, error state, loading)

Screenshots let reviewers understand what changed without running the app locally. They're especially important for reviewers who aren't in the same timezone.

---

## Exit criteria

- [ ] App running locally on the current branch before testing started
- [ ] Golden path walked end-to-end as a user
- [ ] Empty, loading, and error states all display intentional content
- [ ] Responsive layout checked at mobile, tablet, and desktop breakpoints
- [ ] Keyboard navigation works for all interactive elements
- [ ] Images have alt text; form inputs have labels; semantic HTML used for interactivity
- [ ] Edge cases tested: long content, fast interactions, back/refresh behaviour
- [ ] Screenshots captured for the PR description

---

## Common shortcuts to avoid

**"The component renders correctly — that's enough."** A component rendering without errors doesn't mean the loading state, empty state, or error state are handled. Test all of them.

**"I'll test on desktop only — the design is desktop-first."** Responsive issues are invisible on desktop. Test mobile even for desktop-first designs — users resize, zoom, and use mobile devices unexpectedly.

**"Accessibility is for later."** Accessibility issues found during development take minutes to fix. Accessibility issues found after launch may require architectural changes. Check basics before the PR.

**"The agent can verify the UI."** The agent can read and reason about code. It cannot see the rendered result. UI verification requires a human with a browser. Use the agent to guide what to test; do the testing yourself.
