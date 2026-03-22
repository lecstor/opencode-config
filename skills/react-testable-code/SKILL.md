---
name: react-testable-code
description: Write React components designed to be easily tested with Playwright and Testing Library — semantic HTML, clear boundaries, no test-specific hacks
---

> React code that's easy to test is React code that's good. Semantic HTML and clear component boundaries make Playwright trivially able to find and interact with your UI.

## The core principle

Tests should verify what users see and do:

```js
// ❌ Brittle — tests implementation
expect(page.locator('.search-input')).toBeVisible()

// ✅ Resilient — tests user-facing behaviour
await page.getByLabel('Search products').fill('laptop')
await expect(page.getByRole('button', { name: /search/i })).toBeEnabled()
```

## Semantic HTML first

Use the right HTML elements — they carry roles that Playwright can query:

```html
<!-- ❌ Playwright can't tell what this is -->
<div onClick={handleSearch} class="btn">Search</div>

<!-- ✅ Playwright finds this with getByRole('button') -->
<button aria-label="Search"><span aria-hidden="true">🔍</span></button>
```

Always use:
- `<button>` not `<div onClick>`
- `<label>` associated with every input
- `<article>`, `<section>`, `<nav>` for structure
- `aria-label` on icon buttons with no visible text
- `aria-hidden="true"` on decorative icons

## Component architecture for testability

- Single responsibility — one component, one thing to test
- Props are the contract — test that the contract works
- State lives at the lowest common ancestor — no internal state leaking into tests
- Callbacks for child→parent communication — easy to verify in tests

## State patterns that help testing

```js
// ✅ State in the right place — test verifies the full flow
function ProductSearch() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])

  return (
    <>
      <SearchForm onSearch={setQuery} />
      <ProductResults results={results} />
    </>
  )
}
```

When state is in the right place, you don't need to mock internal state — just interact through the UI.

## Forms

```js
// ✅ Every input has an accessible label
<label htmlFor="email">Email address</label>
<input id="email" type="email" value={email} onChange={...} />

// or with implicit label
<label>
  Email address
  <input type="email" value={email} onChange={...} />
</label>
```

Use `aria-describedby` for error messages so screen readers and tests can both find them.

## What NOT to do

- ❌ `data-testid` as primary locator strategy — signals poor accessibility
- ❌ Divs as buttons — no keyboard access, no role, no testable name
- ❌ State stored in refs when it affects rendered output
- ❌ Logic in render — extract to hooks or helpers so components stay simple
- ❌ God components — split when a component does more than one thing

## Supporting files

- **[COMPONENT-ARCHITECTURE.md](COMPONENT-ARCHITECTURE.md)** — patterns for structuring components for testability
- **[ACCESSIBILITY-FOR-TESTING.md](ACCESSIBILITY-FOR-TESTING.md)** — ARIA patterns, semantic HTML reference
- **[ANTI-PATTERNS.md](ANTI-PATTERNS.md)** — what breaks tests and how to fix it
- **[EXAMPLES.md](EXAMPLES.md)** — full examples of testable component patterns
