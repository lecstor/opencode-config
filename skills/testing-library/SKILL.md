---
name: testing-library
description: Write effective React component tests using testing-library by testing user interactions instead of implementation details
---

## What I do

I help you write React component tests that resemble how users actually interact with your application. Testing Library encourages querying elements by their accessibility attributes (role, label, text) rather than implementation details, resulting in tests that give you real confidence and don't break on refactors.

## Core Philosophy

Testing Library's guiding principle is simple:

> **The more your tests resemble the way your software is used, the more confidence they can give you.**

This translates into three core practices:

1. **Test the DOM, not component instances** - Query and interact with DOM nodes the way users do, not via component refs or state inspection
2. **Query by accessibility** - Prefer queries that use accessibility attributes (role, labels, text) because they mirror how users navigate
3. **Avoid testing implementation** - Your tests should verify behavior and output, not internal state or function calls

## Essential Concepts

### Query Priority Order (Always Follow This Hierarchy)

Testing Library provides multiple query methods. **Always follow this priority order** — skip to the next option only when the previous one won't work:

1. **`getByRole()`** ⭐ **FIRST CHOICE**
   - Query by accessibility role (button, heading, link, img, form, alert, etc.)
   - Most recommended because it aligns with how assistive technologies (screen readers) work
   - Encourages semantic HTML and accessible components
   - Mirrors real user interaction
   - **Examples:** `getByRole('button')`, `getByRole('heading', { level: 1 })`, `getByRole('textbox', { name: /email/i })`

2. **`getByLabelText()`** ⭐ **SECOND CHOICE (Form Inputs)**
   - Query form inputs by their associated `<label>` element
   - Guarantees your inputs are properly labeled for accessibility
   - **Examples:** `getByLabelText(/username/i)`, `getByLabelText(/password/i)`

3. **`getByText()` / `getByAltText()`** ⭐ **THIRD CHOICE (Content)**
   - Query by visible text content or alt text for images
   - Use `getByText()` for interactive elements with no other semantic role
   - Use `getByAltText()` specifically for images with proper alt attributes
   - **Examples:** `getByText('Submit')`, `getByAltText('User profile picture')`

4. **`getByTestId()`** ⚠️ **LAST RESORT ONLY**
   - Query by `data-testid` attribute
   - Only use when no semantic role or label exists
   - If you're tempted to use this first, your component may not be accessible
   - Add a comment explaining why role-based queries won't work
   - **Examples:** `getByTestId('custom-svg-icon')` (when SVG has no alt text)

### Query Variants

Each query type has variants for different scenarios:

- **getBy*** - Returns element or throws error (use when element must exist)
- **queryBy*** - Returns element or null (use for asserting absence)
- **findBy*** - Returns Promise, useful for async elements (use with await)
- **getAllBy*** - Returns array of elements instead of single element

### Async Operations

When testing interactions that trigger state changes or API calls:

```javascript
// Wait for element to appear
const element = await screen.findByRole('alert')

// Or use waitFor for more complex async scenarios
await waitFor(() => {
  expect(screen.getByText('Success')).toBeInTheDocument()
})
```

### User Interactions

Simulate user actions like clicks, typing, and form submissions using **`userEvent`** (recommended) instead of `fireEvent`:

> **⚠️ userEvent version matters!** The API changed significantly between v13 and v14.
> - **v14+**: Uses `const user = userEvent.setup()` then `await user.click(...)` (async)
> - **v13.x** (e.g., 13.5.0): Call methods directly — `userEvent.click(...)`, `userEvent.type(...)` (synchronous, no `.setup()`)
>
> **Always check the project's version** in `package.json` before writing tests. Using the wrong API will cause TypeScript errors or test failures.

#### userEvent v14+ (with `.setup()`)

```javascript
import userEvent from '@testing-library/user-event'
import { render, screen } from '@testing-library/react'

test('user interactions', async () => {
  const user = userEvent.setup()
  render(<MyComponent />)
  
  // Click elements
  await user.click(screen.getByRole('button', { name: /submit/i }))
  
  // Type into inputs
  await user.type(screen.getByLabelText(/username/i), 'john')
  
  // Clear and type
  await user.clear(screen.getByLabelText(/password/i))
  await user.type(screen.getByLabelText(/password/i), 'secret123')
})
```

#### userEvent v13.x (direct calls, no `.setup()`)

```javascript
import userEvent from '@testing-library/user-event'
import { render, screen } from '@testing-library/react'

test('user interactions', () => {
  render(<MyComponent />)
  
  // Click elements (synchronous — no await)
  userEvent.click(screen.getByRole('button', { name: /submit/i }))
  
  // Type into inputs
  userEvent.type(screen.getByLabelText(/username/i), 'john')
  
  // Clear and type
  userEvent.clear(screen.getByLabelText(/password/i))
  userEvent.type(screen.getByLabelText(/password/i), 'secret123')
})
```

**Why `userEvent` instead of `fireEvent`?**
- `fireEvent`: Dispatches low-level DOM events only
- `userEvent`: Simulates **full browser interactions** (recommended)
- `userEvent` includes visibility & interactivity checks
- `userEvent` catches more bugs (e.g., clicking hidden elements)
- `userEvent` is the modern testing-library standard

### jest-dom Matchers

Use these assertions from `@testing-library/jest-dom` for better error messages:

- `toBeInTheDocument()` - Element exists in DOM
- `toHaveTextContent(text)` - Element contains text
- `toBeVisible()` - Element is visible to users
- `toBeDisabled()` / `toBeEnabled()` - Button/input state
- `toHaveAttribute(name, value)` - Element has attribute
- `toHaveClass(className)` - Element has CSS class

## Common Workflows

### 1. Basic Setup and Render

```javascript
import '@testing-library/jest-dom'
import { render, screen } from '@testing-library/react'
import MyComponent from '../MyComponent'

test('renders greeting', () => {
  render(<MyComponent />)
  
  // Query by text
  expect(screen.getByText('Hello')).toBeInTheDocument()
})
```

### 2. Testing Form Interaction

```javascript
import userEvent from '@testing-library/user-event'

// v14+ (with setup)
test('submits form with user input', async () => {
  const user = userEvent.setup()
  render(<LoginForm />)
  await user.type(screen.getByLabelText(/username/i), 'alice')
  await user.type(screen.getByLabelText(/password/i), 'secret')
  await user.click(screen.getByRole('button', { name: /submit/i }))
  expect(screen.getByText(/welcome alice/i)).toBeInTheDocument()
})

// v13.x (direct calls)
test('submits form with user input', () => {
  render(<LoginForm />)
  userEvent.type(screen.getByLabelText(/username/i), 'alice')
  userEvent.type(screen.getByLabelText(/password/i), 'secret')
  userEvent.click(screen.getByRole('button', { name: /submit/i }))
  expect(screen.getByText(/welcome alice/i)).toBeInTheDocument()
})
```

### 3. Testing Visibility Toggle

```javascript
import userEvent from '@testing-library/user-event'

// v14+ (with setup)
test('shows message when checkbox is checked', async () => {
  const user = userEvent.setup()
  render(<HiddenMessage>Secret content</HiddenMessage>)
  expect(screen.queryByText('Secret content')).not.toBeInTheDocument()
  await user.click(screen.getByLabelText(/show message/i))
  expect(screen.getByText('Secret content')).toBeInTheDocument()
})

// v13.x (direct calls)
test('shows message when checkbox is checked', () => {
  render(<HiddenMessage>Secret content</HiddenMessage>)
  expect(screen.queryByText('Secret content')).not.toBeInTheDocument()
  userEvent.click(screen.getByLabelText(/show message/i))
  expect(screen.getByText('Secret content')).toBeInTheDocument()
})
```

### 4. Testing Async Operations

```javascript
import userEvent from '@testing-library/user-event'

// v14+ (with setup)
test('displays results after API call', async () => {
  const user = userEvent.setup()
  render(<SearchResults />)
  await user.type(screen.getByLabelText(/search/i), 'react')
  await user.click(screen.getByRole('button', { name: /search/i }))
  const results = await screen.findByRole('list')
  expect(results).toBeInTheDocument()
  expect(screen.getAllByRole('listitem')).toHaveLength(3)
})

// v13.x (direct calls — use findBy/waitFor for async state changes)
test('displays results after API call', async () => {
  render(<SearchResults />)
  userEvent.type(screen.getByLabelText(/search/i), 'react')
  userEvent.click(screen.getByRole('button', { name: /search/i }))
  const results = await screen.findByRole('list')
  expect(results).toBeInTheDocument()
  expect(screen.getAllByRole('listitem')).toHaveLength(3)
})
```

### 5. Mocking External APIs

```javascript
import { rest } from 'msw'
import { setupServer } from 'msw/node'

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json({ name: 'John' }))
  })
)

beforeAll(() => server.listen())
afterEach(() => server.resetHandlers())
afterAll(() => server.close())

test('displays user data from API', async () => {
  render(<UserProfile />)
  
  const name = await screen.findByText('John')
  expect(name).toBeInTheDocument()
})
```

## Query Reference

| Query | Use Case | Returns |
|-------|----------|---------|
| `getByRole(role, options)` | Button, heading, link, etc. by accessibility role | Element or throws |
| `getByLabelText(text)` | Form inputs by their label | Element or throws |
| `getByText(text/regex)` | Elements by visible text | Element or throws |
| `getByTestId(id)` | Last resort when others won't work | Element or throws |
| `getByPlaceholderText(text)` | Input by placeholder attribute | Element or throws |
| `getByAltText(text)` | Images by alt text | Element or throws |
| `queryBy*` | Same as above but returns null if not found | Element or null |
| `findBy*` | Same as above but waits for element (async) | Promise<Element> |
| `getAllBy*` | Returns array instead of single element | Array or throws |

### Common Role Examples

```javascript
// Buttons
screen.getByRole('button', { name: /submit/i })

// Links
screen.getByRole('link', { name: /home/i })

// Headings
screen.getByRole('heading', { level: 1 })

// Form inputs
screen.getByRole('textbox', { name: /username/i })
screen.getByRole('checkbox', { name: /agree/i })

// Lists
screen.getByRole('list')
screen.getAllByRole('listitem')

// Alerts
screen.getByRole('alert')

// SVG icons (if properly marked with role and name)
screen.getByRole('img', { name: /success/i })
```

## Real-World Query Examples

### Example 1: Button with Text (Use getByRole)
```javascript
// Component: <button>Delete Account</button>

// ✅ GOOD - Query by role
screen.getByRole('button', { name: /delete account/i })

// ❌ BAD - Query by text alone (less precise)
screen.getByText(/delete account/i)
```

### Example 2: Form Input with Label (Use getByLabelText)
```javascript
// Component:
// <label>Email: <input type="email" /></label>

// ✅ GOOD - Query by label
screen.getByLabelText(/email/i)

// ❌ BAD - Query by placeholder (less reliable for accessibility)
screen.getByPlaceholderText(/enter email/i)
```

### Example 3: Image with Alt Text (Use getByAltText)
```javascript
// Component: <img src="avatar.png" alt="User profile" />

// ✅ GOOD - Query by alt text
screen.getByAltText(/user profile/i)

// ❌ BAD - Query by src (not accessible)
container.querySelector('img[src="avatar.png"]')
```

### Example 4: SVG Icon (Use getByText on title, or getByRole if available)
```javascript
// Component:
// <svg>
//   <title>Check Circle</title>
//   <path ... />
// </svg>

// ✅ GOOD (option 1) - Query by title text
screen.getByText('Check Circle').closest('svg')

// ✅ GOOD (option 2) - If icon has role and name
screen.getByRole('img', { name: /check/i })

// ❌ BAD - Query by selector (not accessible)
container.querySelector('svg')
```

### Example 5: Last Resort - Custom Component (Use getByTestId)
```javascript
// Component: <CustomChart data-testid="revenue-chart" />
// (No semantic role available for custom visualizations)

// ✅ ACCEPTABLE - With explanation comment
// Revenue chart is a custom visualization with no semantic role
const chart = screen.getByTestId('revenue-chart')
```

## Common Role Examples

```javascript
// Buttons
screen.getByRole('button', { name: /submit/i })

// Links
screen.getByRole('link', { name: /home/i })

// Headings
screen.getByRole('heading', { level: 1 })

// Form inputs
screen.getByRole('textbox', { name: /username/i })
screen.getByRole('checkbox', { name: /agree/i })

// Lists
screen.getByRole('list')
screen.getAllByRole('listitem')

// Alerts
screen.getByRole('alert')
```

## Anti-Patterns to Avoid ❌

These patterns violate testing-library principles and will make your tests fragile:

### 1. Using `container.querySelector()` (DOM Implementation Details)

**❌ BAD:**
```javascript
const { container } = render(<MyButton />)
const button = container.querySelector('button')  // Couples to HTML structure
expect(button).toBeInTheDocument()
```

**✅ GOOD:**
```javascript
render(<MyButton />)
const button = screen.getByRole('button')  // Queries by accessibility role
expect(button).toBeInTheDocument()
```

**Why:** `querySelector()` tightly couples tests to HTML structure. If you refactor the component markup, the test breaks even if the button still works. Role-based queries are resilient to refactoring.

---

### 2. Using `getByTitle()` for Non-Title Elements

**❌ BAD:**
```javascript
// Querying SVG icon by title attribute (not a proper semantic query)
const icon = screen.getByTitle('Check Circle')
```

**✅ GOOD:**
```javascript
// For SVG with <title> element, use getByText() on the title
const icon = screen.getByText('Check Circle').closest('svg')

// OR add alt text and use getByAltText()
const icon = screen.getByAltText('Check')

// OR use getByTestId() as last resort with explanation
const icon = screen.getByTestId('check-icon')  // SVG has no alt text, using testid
```

**Why:** `getByTitle()` is not part of testing-library's API because title attributes aren't a reliable accessibility mechanism. The title attribute is often hidden from screen readers and not available on touch devices. Use `getByText()`, `getByAltText()`, or `getByTestId()` instead.

---

### 3. Querying by CSS Class or ID

**❌ BAD:**
```javascript
const input = container.querySelector('.username-input')  // Couples to CSS class
expect(input).toHaveValue('john')
```

**✅ GOOD:**
```javascript
const input = screen.getByLabelText(/username/i)  // Queries by label
expect(input).toHaveValue('john')
```

**Why:** CSS classes and IDs are styling/targeting concerns, not semantic meaning. Your tests will break whenever CSS changes, even if the component's functionality is identical.

---

### 4. Using `getByTestId()` as First Choice

**❌ BAD:**
```javascript
// Why not use a role?
const button = screen.getByTestId('submit-button')
```

**✅ GOOD:**
```javascript
// Button has semantic meaning - use role!
const button = screen.getByRole('button', { name: /submit/i })
```

**Why:** `getByTestId()` bypasses accessibility checking. If you can't use a role-based query, your component might not be properly accessible. Always try role first; only use `testid` when truly necessary.

---

### 5. Querying SVG Icons Without Proper Accessibility

**❌ BAD:**
```javascript
// Raw SVG with no alt text or role
const icon = container.querySelector('svg')
```

**✅ GOOD:**
```javascript
// Option 1: SVG icon with alt text
const icon = screen.getByAltText('success')

// Option 2: SVG icon with title and accessible role
const icon = screen.getByRole('img', { name: /success/i })

// Option 3: SVG icon with only <title> - use getByText on title content
const icon = screen.getByText('Success').closest('svg')

// Option 4: As absolute last resort
const icon = screen.getByTestId('success-icon')
```

**Why:** SVGs need explicit accessibility markup. Using `querySelector()` on SVGs means your tests can't verify accessibility attributes exist.

---

### 6. Testing Implementation Details Instead of Behavior

**❌ BAD:**
```javascript
// Testing internal state, not user-visible behavior
const component = render(<Counter />)
expect(component.instance().state.count).toBe(1)
```

**✅ GOOD (v14+):**
```javascript
render(<Counter />)
const user = userEvent.setup()
const button = screen.getByRole('button', { name: /increment/i })
await user.click(button)
expect(screen.getByText(/count: 1/i)).toBeInTheDocument()
```

**✅ GOOD (v13.x):**
```javascript
render(<Counter />)
const button = screen.getByRole('button', { name: /increment/i })
userEvent.click(button)
expect(screen.getByText(/count: 1/i)).toBeInTheDocument()
```

**Why:** Internal state is an implementation detail. When you refactor the component to use a hook instead of a class, the bad test breaks. The good test still works because it only verifies user-visible behavior.

---

### 7. Using `fireEvent` When `userEvent` Should Be Used

**❌ BAD:**
```javascript
fireEvent.click(screen.getByRole('button'))
fireEvent.change(input, { target: { value: 'hello' } })
```

**✅ GOOD (v14+):**
```javascript
const user = userEvent.setup()
await user.click(screen.getByRole('button'))
await user.type(input, 'hello')
```

**✅ GOOD (v13.x):**
```javascript
userEvent.click(screen.getByRole('button'))
userEvent.type(input, 'hello')
```

**Why:** `userEvent` simulates real browser interactions including focus, hover, and interactability checks. `fireEvent` just dispatches a raw DOM event.

#### ⚠️ Legitimate `fireEvent` Exceptions (TIER 3)

Some interactions **cannot** be simulated with `userEvent` and require `fireEvent`. Do NOT flag these as anti-patterns:

1. **Gesture simulation** (mouseDown/mouseUp/touchStart/touchEnd sequences) — e.g., press-and-hold buttons that track timing between mouseDown and mouseUp events
2. **Custom event data** — e.g., `fireEvent.mouseEnter(element, { clientX: 100, clientY: 200 })` for zoom/hover position tracking
3. **Transition events** — `fireEvent.transitionEnd(element)` for CSS transition callbacks
4. **Media events** — `fireEvent.ended(audioElement)` for audio/video playback completion
5. **Low-level DOM events** with no userEvent equivalent — e.g., `wheel`, `animationEnd`, `resize`

```javascript
// ✅ LEGITIMATE fireEvent: gesture simulation (press-and-hold)
fireEvent.mouseDown(button)
fireEvent.mouseUp(button)  // component measures time between events

// ✅ LEGITIMATE fireEvent: custom event data for zoom positioning
fireEvent.mouseEnter(image, { clientX: 150, clientY: 200 })

// ✅ LEGITIMATE fireEvent: CSS transition callback
fireEvent.transitionEnd(element)
```

---

## Query Method Decision Tree 🌳

Use this flowchart to choose the right query method:

```
START: Need to find an element?
│
├─→ Is it a button, link, heading, form input, or other semantic role?
│   ├─→ YES: Use getByRole('button'), getByRole('heading'), etc. ✅
│   └─→ NO: Continue...
│
├─→ Is it a form input with a label?
│   ├─→ YES: Use getByLabelText(/label text/i) ✅
│   └─→ NO: Continue...
│
├─→ Is it an image with alt text?
│   ├─→ YES: Use getByAltText('alt text') ✅
│   └─→ NO: Continue...
│
├─→ Is it text content visible to the user?
│   ├─→ YES: Use getByText(/content/i) ✅
│   └─→ NO: Continue...
│
├─→ Is it an SVG with accessible role/name?
│   ├─→ YES: Use getByRole('img', { name: /name/i }) ✅
│   └─→ NO: Continue...
│
├─→ Is it content inside a <title> tag (like SVG <title>)?
│   ├─→ YES: Use getByText('title content').closest('svg') ✅
│   └─→ NO: Continue...
│
└─→ No semantic way to query it?
    └─→ Use getByTestId('unique-id') ⚠️
        (Add data-testid attribute and consider improving component accessibility)
```

## Best Practices

### ✅ Do

- **Always use `screen` object, not `container.querySelector()`** - The `screen` object forces you to query by accessibility-first methods. `container` bypasses accessibility checks.
  ```javascript
  // ✅ Good
  render(<MyComponent />)
  const button = screen.getByRole('button')
  
  // ❌ Avoid
  const { container } = render(<MyComponent />)
  const button = container.querySelector('button')
  ```

- **Query by role first** - It's most accessible and mirrors real user behavior. Use the query priority order: role → label → text/alt → testid.

- **Use regex matchers** (e.g., `/submit/i`) for resilience to content changes
  ```javascript
  // Better: Case-insensitive, partial match
  screen.getByRole('button', { name: /submit/i })
  // vs
  screen.getByRole('button', { name: 'Submit Button' })
  ```

- **Test complete workflows** - User fills form → submits → sees result. This tests realistic behavior.

- **Import jest-dom** at the top of your test file for better assertions

- **Wait for async changes** - Use `findBy` or `waitFor` for API calls and state updates

- **Mock external APIs** - Use MSW to stub API responses

- **Use semantic HTML** - Makes querying by role work naturally. If role queries are hard, your component may not be accessible.

- **Add explanatory comments when using `getByTestId()`** - Explain why semantic queries won't work:
  ```javascript
  // SVG icon has no alt text or role, using testid for now
  // TODO: Add alt text or role to make this accessible
  const icon = screen.getByTestId('success-icon')
  ```

### ❌ Don't

- **Don't use `container.querySelector()`** - Couples tests to HTML structure. Use `screen` instead with role/label/text queries.

- **Don't use `getByTitle()` for querying elements** - Title attributes aren't reliably exposed to screen readers. Use `getByText()` on `<title>` content, `getByAltText()`, or `getByRole()` instead.

- **Don't query by CSS class or ID** - Couples tests to styling. Use semantic queries (role, label, text) instead.

- **Don't use `getByTestId()` as your first choice** - It's a last resort escape hatch. Always try role-based queries first. If testid is necessary, add a comment explaining why.

- **Don't test implementation details** - Don't assert on component state or instance methods. Test what users see instead.

- **Don't use `getByTestId` as first choice** - It's a last resort, not primary strategy

- **Don't test component props directly** - Test what users see instead

- **Don't ignore accessibility** - If role queries are hard, your component may not be accessible

- **Don't fire events without cleanup** - Reset mocks and handlers between tests

- **Don't write tautological tests** - Never render trivial markup (e.g., `render(<div>hello</div>)`) and assert on it. Tests must exercise real component logic — if the assertion only checks something you hardcoded in the test itself, delete it.

- **Don't use `fireEvent` for standard user interactions** - Use `userEvent` instead for clicks, typing, etc. However, `fireEvent` IS appropriate for gesture simulation (mouseDown/mouseUp sequences), custom event data (clientX/clientY), transition/animation events, and media events — see "Legitimate `fireEvent` Exceptions" above.

## Using Context7 for Testing-Library Docs

When you need deeper guidance beyond this skill, use Context7 to query the official testing-library documentation:

### For General Guidance and Examples

```
Query `/websites/testing-library` with your question, for example:
- "How to query elements by role with multiple options?"
- "Best practices for testing async components"
- "Examples of testing error boundaries"
```

**Library ID:** `/websites/testing-library`

### For React-Specific Patterns

```
Query `/testing-library/react-testing-library` for React-focused examples, such as:
- "How to test hooks like useEffect?"
- "Testing controlled vs uncontrolled inputs"
- "Setup and cleanup patterns"
```

**Library ID:** `/testing-library/react-testing-library`

### When to Query Docs

- **Setup requirements** - Initial configuration and test environment setup
- **Framework-specific patterns** - Testing hooks, context, Redux integration
- **Advanced queries** - Complex role-based queries with multiple options
- **Specific assertions** - jest-dom matchers and custom assertions
- **Debugging failing tests** - Understanding why a query isn't finding elements

## When to Use This Skill

- You're writing or reviewing React component tests
- You want to understand testing-library's philosophy and approach
- You need quick reference for query types and common patterns
- You're migrating from Enzyme or older testing approaches
- You want to ensure your tests are testing user behavior, not implementation

## What This Skill Is Not

- Not a test runner guide (use Jest, Vitest, or Playwright docs for that)
- Not a mocking library reference (check MSW docs for detailed mock setup)
- Not a performance testing guide (testing-library focuses on user interaction)
- Not comprehensive API documentation (use Context7 to query official docs for detailed specs)

## Resources

- **Official Docs:** https://testing-library.com/docs/
- **Guiding Principles:** https://testing-library.com/docs/guiding-principles
- **React Testing Library GitHub:** https://github.com/testing-library/react-testing-library
- **jest-dom Matchers:** https://github.com/testing-library/jest-dom
- **Mock Service Worker (MSW):** https://mswjs.io/
- **Common Setup Examples:** Query Context7 with `/testing-library/react-testing-library` for framework-specific patterns
