# React Testable Code Skill

A comprehensive OpenCode skill for writing React components that are trivial to test with Playwright E2E tests.

## Core Philosophy

> **Good React code is code that's easy to test.**

When you design React components with E2E testing in mind, you naturally:

- Use **semantic HTML** (`<button>`, `<label>`, `<form>`, etc.)
- Write **accessible code** (ARIA attributes, proper roles)
- Create **clear component boundaries** (single responsibility)
- Make **data flow explicit** (props down, callbacks up)
- **Avoid implementation details** (no test IDs, CSS class selectors)

As a bonus, Playwright testing becomes trivial. You query the accessibility tree using `getByRole()`, `getByLabel()`, `getByText()` — and it just works.

## Key Insight

Playwright doesn't look at CSS classes or test IDs. It queries the **accessibility tree**, the same tree that assistive technologies use. When you build accessible React code, E2E tests work automatically.

```javascript
// ❌ Brittle test tied to implementation
test('searching works', async ({ page }) => {
  await page.locator('.search-input').fill('laptop');
  await page.locator('.btn-primary').click();
});

// ✅ Resilient test focused on behavior
test('user can search for products', async ({ page }) => {
  // These queries find elements by their semantic meaning
  await page.getByLabel('Search products').fill('laptop');
  await page.getByRole('button', { name: /search/i }).click();
  
  // Verify user sees results
  await expect(page.getByText(/laptop/i)).toBeVisible();
});
```

The second test passes whether you refactor CSS, rename classes, or change frameworks. It tests what the user does, not implementation details.

## What's Included

### 1. **SKILL.md** (~1500 lines)
The comprehensive guide covering:
- Testing philosophy and why it matters
- Thinking in React for testability (5-step process)
- Semantic HTML & accessibility best practices
- Component architecture and boundaries
- State management patterns
- Props vs state decision-making
- One-way and inverse data flow
- Event handlers and forms
- Composition over configuration
- Testing patterns with Playwright

### 2. **EXAMPLES.md** (~1550 lines)
30+ production-ready component examples:
- **Form Components**: Search, login, contact form, filters, real-time validation
- **List & Table Components**: Todo list, data tables with sorting/pagination, expandable rows
- **Modal & Dialog Components**: Confirmation dialogs, modal with form, alerts
- **Compound Components**: Tabs, accordion, menu, breadcrumbs
- **Async & Loading**: Data fetching patterns, form submission, loading states
- **Error Handling**: Error boundaries, error states, validation feedback

Each example includes working code, why it's testable, and Playwright test examples.

### 3. **ANTI-PATTERNS.md** (~700 lines)
8 common anti-patterns that break E2E tests:
1. Divs with onClick instead of buttons
2. Missing labels on form inputs
3. Using test IDs instead of semantic HTML
4. Missing accessible names on icon buttons
5. State in the wrong component
6. Props drilling too deep
7. Boolean prop proliferation
8. Missing error boundaries

Each includes why it breaks tests and how to fix it.

### 4. **ACCESSIBILITY-FOR-TESTING.md** (~680 lines)
The critical connection between accessibility and testability:
- The accessibility tree: what Playwright actually sees
- How semantic HTML builds the right tree
- ARIA attributes (aria-label, aria-live, aria-expanded, etc.)
- Semantic HTML elements and their roles
- Building accessible forms
- Keyboard navigation testing
- Checklist for accessible (and testable) components

### 5. **COMPONENT-ARCHITECTURE.md** (~750 lines)
Design patterns for component structure:
- Single responsibility principle
- Composition over inheritance
- Where state should live (decision tree)
- Context vs props (when to use each)
- Custom hooks for logic extraction
- Compound components for flexible APIs
- Avoiding prop drilling
- Performance optimization patterns

### 6. **QUICKSTART.md** (~600 lines)
Get started immediately with:
- 5 copy-paste component templates (form, data fetching, list, modal, tabs)
- Converting styled components to semantic HTML
- Playwright test structure and patterns
- Common Playwright queries
- Testable components checklist

### 7. **INDEX.md**
Navigation guide with:
- Quick navigation by learning path
- Document overviews
- Key concepts across documents
- How to use the skill for different scenarios

## Learning Paths

### Fast Track (75 minutes)
1. **QUICKSTART.md** (20 min) - Get 5 templates working
2. **ANTI-PATTERNS.md** (15 min) - Learn what not to do
3. **EXAMPLES.md** (40 min) - Study real implementations

### Thorough Path (195 minutes)
1. **SKILL.md** (60 min) - Understand principles
2. **ACCESSIBILITY-FOR-TESTING.md** (30 min) - Why it matters
3. **COMPONENT-ARCHITECTURE.md** (40 min) - Design patterns
4. **EXAMPLES.md** (45 min) - See everything in practice
5. **ANTI-PATTERNS.md** (20 min) - Learn from mistakes

### Reference Lookups
- Need a specific pattern? → **EXAMPLES.md**
- What breaks tests? → **ANTI-PATTERNS.md**
- Accessibility questions? → **ACCESSIBILITY-FOR-TESTING.md**
- Design questions? → **COMPONENT-ARCHITECTURE.md**
- Quick templates? → **QUICKSTART.md**

## Key Concepts

### Semantic HTML is Testable HTML

```javascript
// ❌ Not semantic, not testable
<div onClick={handleSearch} role="button">Search</div>

// ✅ Semantic, fully testable
<button onClick={handleSearch}>Search</button>
```

Playwright finds the button with `getByRole('button', { name: /search/i })`. The div won't be found. Semantic HTML = testable HTML.

### Labels Make Inputs Findable

```javascript
// ❌ No label, input is unfindable
<input type="email" placeholder="Email" />

// ✅ Proper label, fully testable
<label htmlFor="email">Email</label>
<input id="email" type="email" />
```

With a label, you find the input via `getByLabel('Email')`. Without one, you're stuck with fragile placeholder or test ID selectors.

### Accessibility Tree = Testing Tree

Playwright queries the accessibility tree (same as screen readers). When you build accessible code, tests work automatically.

```javascript
// Icons need accessible names
<button aria-label="Search">🔍</button>  // Testable
<button>🔍</button>                       // Not testable

// Error messages need roles
<p role="alert">Error message</p>       // Testable
<p class="error">Error message</p>      // Not testable

// Loading states need aria-busy or clear text
<p aria-busy="true">Loading...</p>      // Testable
// (implicit from text content)          // Testable
```

### State Lives Where Data Flow Requires

```javascript
// ❌ State in wrong place
function ProductSearch() {
  return (
    <>
      <FilterPanel /> {/* Has filter state */}
      <Results /> {/* Can't see filter */}
    </>
  );
}

// ✅ State in parent
function ProductSearch() {
  const [filters, setFilters] = useState({});
  
  return (
    <>
      <FilterPanel filters={filters} onChange={setFilters} />
      <Results filters={filters} />
    </>
  );
}
```

The parent needs to see both filter state and results. State lives where it's accessed.

## Common Patterns

### Form Component

```javascript
function MyForm({ onSubmit }) {
  const [data, setData] = useState({});
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setData(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    // Validate, submit...
  };

  return (
    <form onSubmit={handleSubmit} noValidate>
      <label htmlFor="field">Field</label>
      <input
        id="field"
        name="field"
        value={data.field}
        onChange={handleChange}
        aria-describedby={errors.field ? 'field-error' : undefined}
      />
      {errors.field && (
        <p id="field-error" role="alert">{errors.field}</p>
      )}
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

### Async Data Fetching

```javascript
function DataDisplay({ url }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        setData(await response.json());
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  if (loading) return <p aria-busy="true">Loading...</p>;
  if (error) return <p role="alert">Error: {error}</p>;
  if (!data) return <p>No data found.</p>;

  return <div>{/* Render data */}</div>;
}
```

### Tabs Component

```javascript
const TabsContext = createContext();

export function Tabs({ children, defaultValue }) {
  const [value, setValue] = useState(defaultValue);
  return (
    <TabsContext.Provider value={{ value, setValue }}>
      {children}
    </TabsContext.Provider>
  );
}

export function TabsTrigger({ value, children }) {
  const { value: activeValue, setValue } = useContext(TabsContext);
  return (
    <button
      role="tab"
      aria-selected={activeValue === value}
      onClick={() => setValue(value)}
    >
      {children}
    </button>
  );
}

export function TabsContent({ value, children }) {
  const { value: activeValue } = useContext(TabsContext);
  return activeValue === value ? (
    <div role="tabpanel">{children}</div>
  ) : null;
}
```

## Testing Examples

### Finding Elements

```javascript
// By role (preferred)
page.getByRole('button', { name: /submit/i })
page.getByRole('textbox', { name: /email/i })
page.getByRole('heading', { level: 1 })
page.getByRole('tab', { name: /details/i })

// By label
page.getByLabel('Email Address')
page.getByLabel(/search products/i)

// By text
page.getByText('Click me')
page.getByText(/welcome/i)

// Avoid these (fragile)
page.getByPlaceholderText('...')  // Brittle
page.getByTestId('...')            // Implementation detail
page.locator('.btn-primary')       // CSS specific
```

### Testing User Workflows

```javascript
test('user can search for products', async ({ page }) => {
  await page.getByLabel('Search products').fill('laptop');
  await page.getByRole('button', { name: /search/i }).click();
  
  await expect(page.getByText(/laptop/i)).toBeVisible();
});

test('form shows validation errors', async ({ page }) => {
  await page.getByRole('button', { name: /submit/i }).click();
  
  await expect(page.getByRole('alert')).toBeVisible();
  await expect(page.getByRole('alert')).toContainText('Email is required');
});

test('user can switch between tabs', async ({ page }) => {
  await page.getByRole('tab', { name: /details/i }).click();
  
  await expect(page.getByRole('tabpanel')).toContainText('Details');
});

test('loading state shows and hides', async ({ page }) => {
  await expect(page.getByText(/loading/i)).toBeVisible();
  
  // Wait for loading to finish
  await expect(page.getByText(/loading/i)).not.toBeVisible();
  
  // Data is now visible
  await expect(page.getByText(/results/i)).toBeVisible();
});
```

## Summary

This skill teaches that **testability is not an afterthought** — it's a natural consequence of good React architecture.

When you build components with:
- ✅ Semantic HTML
- ✅ Proper accessibility
- ✅ Clear component boundaries
- ✅ Explicit data flow
- ✅ No reliance on implementation details

...then Playwright tests that verify user behavior work perfectly.

**6,088 lines** of guides, patterns, examples, and anti-patterns to help you build testable React code.

---

## Files

- `INDEX.md` - Navigation guide
- `SKILL.md` - Comprehensive guide (1,500+ lines)
- `EXAMPLES.md` - 30+ component examples (1,550+ lines)
- `ANTI-PATTERNS.md` - 8 anti-patterns and fixes (700 lines)
- `ACCESSIBILITY-FOR-TESTING.md` - Why accessibility = testability (680 lines)
- `COMPONENT-ARCHITECTURE.md` - Design patterns (750 lines)
- `QUICKSTART.md` - 5 templates and quick reference (600 lines)

---

**Start with INDEX.md or QUICKSTART.md**
