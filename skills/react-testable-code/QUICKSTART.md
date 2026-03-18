# Quick Start: 5 Simple Templates to Get Started

This guide gives you 5 simple templates you can copy-paste to start building testable React components immediately.

---

## Template 1: Form Component

This is your go-to template for any form:

```javascript
function MyForm({ onSubmit }) {
  const [formData, setFormData] = useState({
    field1: '',
    field2: ''
  });
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Validate
    const newErrors = {};
    if (!formData.field1) newErrors.field1 = 'This field is required';
    if (!formData.field2) newErrors.field2 = 'This field is required';

    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

    // Submit
    setIsSubmitting(true);
    try {
      await onSubmit(formData);
    } catch (error) {
      setErrors({ submit: error.message });
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} noValidate>
      <div>
        <label htmlFor="field1">Field 1</label>
        <input
          id="field1"
          name="field1"
          type="text"
          value={formData.field1}
          onChange={handleChange}
          disabled={isSubmitting}
          aria-describedby={errors.field1 ? 'field1-error' : undefined}
          required
        />
        {errors.field1 && (
          <p id="field1-error" role="alert">
            {errors.field1}
          </p>
        )}
      </div>

      <div>
        <label htmlFor="field2">Field 2</label>
        <input
          id="field2"
          name="field2"
          type="text"
          value={formData.field2}
          onChange={handleChange}
          disabled={isSubmitting}
          aria-describedby={errors.field2 ? 'field2-error' : undefined}
          required
        />
        {errors.field2 && (
          <p id="field2-error" role="alert">
            {errors.field2}
          </p>
        )}
      </div>

      {errors.submit && (
        <p role="alert">{errors.submit}</p>
      )}

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

**Test template:**

```javascript
test('form validation works', async ({ page }) => {
  await page.getByRole('button', { name: /submit/i }).click();
  
  await expect(page.getByRole('alert')).toContainText('This field is required');
});

test('form submission works', async ({ page }) => {
  await page.getByLabel('Field 1').fill('Value 1');
  await page.getByLabel('Field 2').fill('Value 2');
  await page.getByRole('button', { name: /submit/i }).click();
  
  // Assert what happens after submit
});
```

---

## Template 2: Data Fetching with Loading and Error States

```javascript
function DataDisplay({ url }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);
      setError(null);
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const json = await response.json();
        setData(json);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  if (loading) {
    return <p aria-busy="true">Loading...</p>;
  }

  if (error) {
    return (
      <div role="alert">
        <p>Error: {error}</p>
        <button onClick={() => window.location.reload()}>
          Try Again
        </button>
      </div>
    );
  }

  if (!data) {
    return <p>No data found.</p>;
  }

  return (
    <div>
      {/* Render your data here */}
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}
```

**Test template:**

```javascript
test('shows loading state', async ({ page }) => {
  await expect(page.getByText(/loading/i)).toBeVisible();
});

test('shows data after loading', async ({ page }) => {
  await expect(page.getByText(/data/i)).toBeVisible();
});

test('shows error on failure', async ({ page }) => {
  // Mock API failure...
  await expect(page.getByRole('alert')).toBeVisible();
});
```

---

## Template 3: List Component with Add/Remove

```javascript
function ItemList() {
  const [items, setItems] = useState([]);
  const [input, setInput] = useState('');

  const addItem = () => {
    if (input.trim()) {
      setItems([...items, { id: Date.now(), text: input }]);
      setInput('');
    }
  };

  const removeItem = (id) => {
    setItems(items.filter(item => item.id !== id));
  };

  const handleKeyPress = (e) => {
    if (e.key === 'Enter') {
      addItem();
    }
  };

  return (
    <div>
      <h1>Items</h1>

      <div>
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={handleKeyPress}
          placeholder="Add an item"
          aria-label="New item"
        />
        <button onClick={addItem}>Add</button>
      </div>

      {items.length === 0 ? (
        <p>No items yet.</p>
      ) : (
        <ul>
          {items.map(item => (
            <li key={item.id}>
              <span>{item.text}</span>
              <button
                onClick={() => removeItem(item.id)}
                aria-label={`Delete "${item.text}"`}
              >
                Delete
              </button>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

**Test template:**

```javascript
test('can add and remove items', async ({ page }) => {
  await page.getByLabel('New item').fill('Item 1');
  await page.getByRole('button', { name: /add/i }).click();

  await expect(page.getByText('Item 1')).toBeVisible();

  await page.getByLabel(/delete.*item 1/i).click();

  await expect(page.getByText('Item 1')).not.toBeVisible();
});
```

---

## Template 4: Modal/Dialog Component

```javascript
function MyModal({ isOpen, onClose, title, children }) {
  const dialogRef = useRef(null);

  useEffect(() => {
    if (isOpen && dialogRef.current) {
      dialogRef.current.showModal();
    } else if (dialogRef.current) {
      dialogRef.current.close();
    }
  }, [isOpen]);

  const handleBackdropClick = (e) => {
    if (e.target === dialogRef.current) {
      onClose();
    }
  };

  return (
    <dialog
      ref={dialogRef}
      onCancel={onClose}
      onClick={handleBackdropClick}
      aria-labelledby="modal-title"
    >
      <h2 id="modal-title">{title}</h2>

      {children}

      <button onClick={onClose}>Close</button>
    </dialog>
  );
}

// Usage
function App() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(true)}>Open Modal</button>

      <MyModal
        isOpen={isOpen}
        onClose={() => setIsOpen(false)}
        title="Modal Title"
      >
        <p>Modal content goes here</p>
      </MyModal>
    </>
  );
}
```

**Test template:**

```javascript
test('can open and close modal', async ({ page }) => {
  await page.getByRole('button', { name: /open modal/i }).click();

  await expect(page.getByRole('dialog')).toBeVisible();

  await page.getByRole('button', { name: /close/i }).click();

  await expect(page.getByRole('dialog')).not.toBeVisible();
});
```

---

## Template 5: Tabs Component

```javascript
const TabsContext = createContext();

function Tabs({ children, defaultValue }) {
  const [value, setValue] = useState(defaultValue);

  return (
    <TabsContext.Provider value={{ value, setValue }}>
      <div>{children}</div>
    </TabsContext.Provider>
  );
}

function TabsList({ children }) {
  return <div role="tablist">{children}</div>;
}

function TabsTrigger({ value, children }) {
  const { value: activeValue, setValue } = useContext(TabsContext);

  return (
    <button
      role="tab"
      aria-selected={activeValue === value}
      aria-controls={`panel-${value}`}
      onClick={() => setValue(value)}
    >
      {children}
    </button>
  );
}

function TabsContent({ value, children }) {
  const { value: activeValue } = useContext(TabsContext);

  return activeValue === value ? (
    <div role="tabpanel" id={`panel-${value}`}>
      {children}
    </div>
  ) : null;
}

// Usage
function App() {
  return (
    <Tabs defaultValue="tab1">
      <TabsList>
        <TabsTrigger value="tab1">Tab 1</TabsTrigger>
        <TabsTrigger value="tab2">Tab 2</TabsTrigger>
      </TabsList>

      <TabsContent value="tab1">
        <p>Content 1</p>
      </TabsContent>

      <TabsContent value="tab2">
        <p>Content 2</p>
      </TabsContent>
    </Tabs>
  );
}
```

**Test template:**

```javascript
test('can switch between tabs', async ({ page }) => {
  const tab1 = page.getByRole('tab', { name: /tab 1/i });
  const tab2 = page.getByRole('tab', { name: /tab 2/i });

  await expect(tab1).toHaveAttribute('aria-selected', 'true');

  await tab2.click();

  await expect(tab2).toHaveAttribute('aria-selected', 'true');
  await expect(page.getByRole('tabpanel')).toContainText('Content 2');
});
```

---

## Converting Styled Components to Semantic HTML

### Before: Div-based Component

```javascript
function Button() {
  return (
    <div
      className="button-wrapper"
      onClick={handleClick}
      role="button"
      tabIndex={0}
    >
      <span className="button-icon">🔍</span>
      <span className="button-text">Search</span>
    </div>
  );
}
```

### After: Semantic HTML

```javascript
function Button() {
  return (
    <button onClick={handleClick}>
      <span aria-hidden="true">🔍</span>
      <span>Search</span>
    </button>
  );
}
```

**The CSS might stay the same:**

```css
/* .button-wrapper becomes button */
button {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  border: 1px solid currentColor;
  background: white;
  cursor: pointer;
  border-radius: 0.25rem;
}

button:hover {
  background: #f0f0f0;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

---

## Testing Your Components with Playwright

### Basic Test File Structure

```javascript
// mycomponent.test.js
import { test, expect } from '@playwright/test';

test.describe('MyComponent', () => {
  test.beforeEach(async ({ page }) => {
    // Navigate to component page
    await page.goto('http://localhost:3000/my-component');
  });

  test('renders correctly', async ({ page }) => {
    // Assertions
    await expect(page.getByRole('heading')).toContainText('My Component');
  });

  test('handles user interaction', async ({ page }) => {
    // Interact
    await page.getByRole('button', { name: /click me/i }).click();

    // Assert
    await expect(page.getByText(/clicked/i)).toBeVisible();
  });

  test('shows error state', async ({ page }) => {
    // Trigger error
    await page.getByRole('button', { name: /invalid/i }).click();

    // Assert error
    await expect(page.getByRole('alert')).toBeVisible();
  });
});
```

### Running Tests

```bash
# Run all tests
npx playwright test

# Run tests in a specific file
npx playwright test mycomponent.test.js

# Run tests in UI mode (watch changes)
npx playwright test --ui

# Run tests for a specific test
npx playwright test -g "renders correctly"
```

---

## Common Playwright Queries

```javascript
// By Role (Preferred)
page.getByRole('button', { name: /submit/i })
page.getByRole('textbox', { name: /email/i })
page.getByRole('heading', { level: 1 })
page.getByRole('tab', { name: /details/i })

// By Label (For inputs)
page.getByLabel('Email Address')
page.getByLabel(/search products/i)

// By Text
page.getByText('Click me')
page.getByText(/welcome.*/i)

// By Placeholder (Last resort)
page.getByPlaceholderText('Enter your name')

// By Alt Text (For images)
page.getByAltText('Product image')

// For IDs (If necessary)
page.locator('#my-id')
```

---

## Checklist for Testable Components

Before you submit a component, verify:

- [ ] Uses semantic HTML (`<button>`, `<label>`, `<input>`, etc.)
- [ ] Form inputs have associated `<label>` elements
- [ ] Icon buttons have `aria-label`
- [ ] Interactive elements are keyboard accessible
- [ ] Error messages have `role="alert"`
- [ ] Forms have proper `onSubmit` and don't use `onClick` on divs
- [ ] Loading states are clear (aria-busy or descriptive text)
- [ ] Async operations have proper error handling
- [ ] Tests use `getByRole`, `getByLabel`, `getByText` (not data-testid)
- [ ] All interactions work from tests without implementation knowledge

---

## Next Steps

- Copy a template above and adapt it to your needs
- See **EXAMPLES.md** for 30+ real-world components
- See **SKILL.md** for deep dives on each pattern
- See **ANTI-PATTERNS.md** for what to avoid
