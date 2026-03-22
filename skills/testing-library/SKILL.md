---
name: testing-library
description: Write React component tests using Testing Library. Test user interactions and DOM output, not implementation details.
---

> The more your tests resemble the way your software is used, the more confidence they can give you.

## ⚠️ Check `@testing-library/user-event` version first

Look for it in `package.json` devDependencies:

**v13 and below** — direct synchronous calls:
```js
userEvent.click(screen.getByRole('button', { name: /submit/i }))
userEvent.type(screen.getByLabelText(/email/i), 'user@example.com')
```

**v14+** — setup instance, all async:
```js
const user = userEvent.setup()
await user.click(screen.getByRole('button', { name: /submit/i }))
await user.type(screen.getByLabelText(/email/i), 'user@example.com')
```

## Query priority (always follow this order)

1. **`getByRole()`** — buttons, headings, inputs, links, dialogs
2. **`getByLabelText()`** — form inputs with labels
3. **`getByText()` / `getByAltText()`** — visible text, images
4. **`getByTestId()`** — last resort only (add a comment explaining why)

Query variants: `getBy*` throws if missing · `queryBy*` returns null · `findBy*` async · `getAllBy*` returns array

## Key rules

- Always use `screen.*`, never `container.querySelector()`
- Use regex matchers (`/submit/i`) for resilience to text changes
- For async state changes: use `findBy*` or `waitFor`
- `userEvent` over `fireEvent` for standard interactions
- `fireEvent` IS appropriate for: gesture sequences (mouseDown/Up), custom event data (clientX/Y), transition/media events

## Common jest-dom assertions

```js
expect(el).toBeInTheDocument()
expect(el).toBeVisible()
expect(el).toBeDisabled() / .toBeEnabled()
expect(el).toHaveTextContent('foo')
expect(el).toHaveValue('bar')
expect(el).toHaveAttribute('href', '/x')
```

## Anti-patterns

- ❌ `container.querySelector()` — couples to HTML structure
- ❌ `getByTestId()` as first choice — bypasses accessibility checking
- ❌ `getByTitle()` for non-title elements — not reliably accessible
- ❌ Query by CSS class or ID
- ❌ Test component state or instance methods
- ❌ `fireEvent` for standard clicks/typing
- ❌ Tautological tests (asserting on markup you hardcoded in the test)
