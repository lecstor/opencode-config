---
name: e2e-testing
description: Write end-to-end tests using Playwright. Test user behavior, not implementation details. Also useful for non-test Playwright scripting.
---

Test what users see and do, never implementation details. Every selector choice should be semantic.

## Locator Priority (always follow this order)

| Priority | Method | Use when |
|---|---|---|
| 1 | `getByRole('button', { name: /submit/i })` | Any semantic element |
| 2 | `getByLabel('Email')` | Form inputs |
| 3 | `getByPlaceholder(...)` | Inputs without labels |
| 4 | `getByAltText(...)` | Images |
| 5 | `getByTitle(...)` | Icon buttons with no text |
| 6 | `getByTestId(...)` | Last resort only |
| 7 | `getByText(...)` | Non-interactive text only |
| ❌ | `locator('.class')` / XPath | Never |

## Assertions — always use web-first (`await expect`)

```js
await expect(page.getByRole('button', { name: 'Save' })).toBeVisible()
await expect(page.getByLabel('Email')).toHaveValue('user@example.com')
await expect(page.getByRole('checkbox')).toBeChecked()
await expect(page.getByRole('listitem')).toHaveCount(5)
await expect(page).toHaveURL('/dashboard')
```

Never use `waitForTimeout()` or `waitForSelector()` — `await expect()` auto-retries.

## Test isolation rules

- Each test sets up its own state (login, data, etc.)
- No test depends on another test's side effects
- Use `beforeEach` for common setup within a suite
- Use `afterEach` for cleanup (prefer API calls over UI)
- Use fixtures for reusable setup (authenticated page, etc.)

## Core rules

- ✅ Test user behavior (what they see and can do)
- ✅ Embrace auto-waiting — Playwright waits for actionability automatically
- ✅ Semantic locators guarantee accessibility compliance as a side effect
- ❌ No CSS/XPath selectors
- ❌ No `waitForTimeout()` / arbitrary sleeps
- ❌ No testing internal state (window.__state, component props, etc.)

## Supporting files

- **[QUICKSTART.md](QUICKSTART.md)** — common test patterns ready to copy
- **[EXAMPLES.md](EXAMPLES.md)** — complete real-world test examples
- **[ANTI-PATTERNS.md](ANTI-PATTERNS.md)** — what not to do and why
- **[DEBUGGING-FAILING-TESTS.md](DEBUGGING-FAILING-TESTS.md)** — systematic workflow for fixing failing tests
