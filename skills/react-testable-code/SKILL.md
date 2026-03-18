# React Code That's Easy to Test with Playwright

> **Core Insight:** React code that's easy to test with E2E tests is React code that's good. Good React code is built on semantic HTML, accessibility, and clear component hierarchies. These same practices make it trivial for Playwright to find and interact with your UI.

## Table of Contents

1. [The Philosophy](#the-philosophy)
2. [Thinking in React for Testability](#thinking-in-react-for-testability)
3. [Semantic HTML & Accessibility First](#semantic-html--accessibility-first)
4. [Component Architecture](#component-architecture)
5. [State Management](#state-management)
6. [Props vs State](#props-vs-state)
7. [Data Flow Patterns](#data-flow-patterns)
8. [Event Handlers & Forms](#event-handlers--forms)
9. [Composition Over Configuration](#composition-over-configuration)
10. [Testing Patterns](#testing-patterns)

---

## The Philosophy

### Why Testability Matters

When you write React code with E2E testing in mind, you naturally:

- **Use semantic HTML** - This makes it possible for Playwright to find elements by role, label, and text
- **Write accessible code** - Proper ARIA attributes support both assistive tech and test queries
- **Think about component boundaries** - Clear components with single responsibilities are easier to test
- **Follow data flow patterns** - Unidirectional data flow makes behavior predictable
- **Avoid implementation details** - You don't rely on CSS classes or test IDs, making refactoring safe

### The Testing Philosophy

Tests should verify what users see and do, not implementation details.

```javascript
// ❌ Tests brittle implementation
test('input has class "search-input"', async () => {
  expect(page.locator('.search-input')).toBeVisible();
});

// ✅ Tests user-facing behavior
test('user can search by product name', async () => {
  await page.getByLabel('Search products').fill('laptop');
  await expect(page.getByRole('button', { name: /search/i })).toBeEnabled();
});
```

The second test passes whether you use CSS Modules, styled-components, Tailwind, or inline styles. It tests what the user does.

---

## Thinking in React for Testability

React's "Thinking in React" approach naturally aligns with E2E testing. Here's how to apply it:

### Step 1: Break the UI Into a Component Hierarchy

Start with mockups or designs. Identify components by asking: "What's a single responsibility?"

```
ProductSearch
├── SearchForm
│   ├── SearchInput
│   └── SearchButton
├── FilterPanel
│   ├── PriceFilter
│   └── CategoryFilter
└── ProductResults
    ├── ProductCard
    │   ├── ProductImage
    │   ├── ProductInfo
    │   └── AddToCartButton
    └── Pagination
```

**Why this helps testing:** Each component has a single, testable purpose.

```javascript
// Test SearchForm independently
test('search form submits with input value', async ({ page }) => {
  await page.getByLabel('Search').fill('laptop');
  await page.getByRole('button', { name: /search/i }).click();
  // Verify behavior...
});

// Test ProductCard independently
test('product card displays price and add to cart button', async ({ page }) => {
  // Mount component with props...
  await expect(page.getByRole('button', { name: /add to cart/i })).toBeVisible();
});
```

### Step 2: Build a Static Version

First, build the UI without interactivity. Think about component props and HTML structure:

```javascript
function ProductCard({ product }) {
  return (
    <article>
      <img src={product.image} alt={product.name} />
      <h2>{product.name}</h2>
      <p>{product.description}</p>
      <span>${product.price}</span>
      <button>Add to Cart</button>
    </article>
  );
}
```

**Why this helps testing:** You're using semantic HTML from the start. Playwright can find elements by role (`<button>`, `<article>`, `<h2>`) and alt text.

### Step 3: Identify State

What data changes based on user interaction?

```javascript
// State needed:
// - Search query (user types in input)
// - Search results (loaded from API)
// - Loading state (while fetching)
// - Error state (if fetch fails)
// - Selected filters (user clicks filter buttons)
// - Current page (user clicks pagination)
```

For each piece of state, ask:
- Is it set by user input or an API?
- Does multiple components need this state?
- Where should it live?

### Step 4: Identify Where State Lives

```javascript
// ❌ Wrong: State in SearchInput, but parent needs results
function SearchForm({ onResultsLoad }) {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  // Parent can't see results
}

// ✅ Right: State in parent that renders both form and results
function ProductSearch() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleSearch = async (q) => {
    setQuery(q);
    const data = await fetchProducts(q);
    setResults(data);
  };

  return (
    <>
      <SearchForm onSearch={handleSearch} />
      <ProductResults results={results} />
    </>
  );
}
```

**Why this helps testing:** When state lives in the right place, your component hierarchy reflects data flow. Tests that verify the hierarchy work automatically.

```javascript
test('search results appear after searching', async ({ page }) => {
  // No need to mock internal state
  await page.getByLabel('Search').fill('laptop');
  await page.getByRole('button', { name: /search/i }).click();
  
  // Wait for results to appear (state updated)
  await expect(page.getByRole('heading', { level: 2 })).toContainText('laptop');
});
```

### Step 5: Add Inverse Data Flow

Connect callbacks to move data back up:

```javascript
function SearchForm({ onSearch }) {
  const [query, setQuery] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    onSearch(query); // Pass data to parent
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        aria-label="Search products"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <button type="submit">Search</button>
    </form>
  );
}
```

**Why this helps testing:** Callbacks are the contract between components. Tests verify the contract works.

```javascript
test('search form calls onSearch with input value', async ({ page }) => {
  // In your test, mount the component and track the callback
  let searchQuery = null;
  await page.evaluate(() => {
    window.testCallbacks = {
      onSearch: (q) => { searchQuery = q; }
    };
  });

  await page.getByLabel('Search products').fill('laptop');
  await page.getByRole('button', { name: /search/i }).click();
  
  // Callback was triggered
  expect(searchQuery).toBe('laptop');
});
```

---

## Semantic HTML & Accessibility First

This is where testability begins. Semantic HTML elements and accessibility attributes are what make your UI discoverable.

### Why Semantic HTML Matters

Semantic elements carry meaning:

```html
<!-- ❌ Not semantic: Playwright can't tell what this is -->
<div onClick={handleSearch} class="search-btn">
  <span>🔍</span>
</div>

<!-- ✅ Semantic: Playwright finds it with getByRole('button') -->
<button aria-label="Search">
  <span aria-hidden="true">🔍</span>
</button>
```

Playwright uses the accessibility tree to find elements. The accessibility tree is built from semantic HTML and ARIA.

### Common Semantic Elements & Roles

```javascript
// Buttons and Links
<button>Click me</button>           // role: button
<a href="/page">Link</a>            // role: link

// Forms
<input type="text" />               // role: textbox
<input type="checkbox" />           // role: checkbox
<select><option>A</option></select> // role: combobox
<textarea></textarea>               // role: textbox
<label htmlFor="input">Label</label> // Connects to input

// Headings
<h1>Main Title</h1>                 // role: heading, level: 1
<h2>Section Title</h2>              // role: heading, level: 2

// Lists
<ul><li>Item</li></ul>              // roles: list, listitem

// Regions
<nav>Navigation</nav>               // role: navigation
<main>Main content</main>            // role: main
<article>Article</article>          // role: article

// Tables
<table><thead><tr><th>Header</th></tr></thead></table> // roles: table, rowheader

// Landmarks
<header>Header</header>             // role: banner (when in main context)
<footer>Footer</footer>             // role: contentinfo
<aside>Sidebar</aside>              // role: complementary
```

### Using Semantic HTML in Components

```javascript
// ❌ Not semantic - uses divs everywhere
function LoginForm({ onSubmit }) {
  return (
    <div className="login-form">
      <div className="form-group">
        <div>Email</div>
        <div className="text-input" />
      </div>
      <div className="form-group">
        <div>Password</div>
        <div className="text-input" />
      </div>
      <div className="button" onClick={onSubmit}>
        Log In
      </div>
    </div>
  );
}

// ✅ Semantic - uses proper elements
function LoginForm({ onSubmit }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(email, password);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
      </div>
      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
        />
      </div>
      <button type="submit">Log In</button>
    </form>
  );
}
```

**Testing the semantic version:**

```javascript
test('user can log in with email and password', async ({ page }) => {
  // Find by label - works because of <label htmlFor>
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password123');
  
  // Find button by role - works because it's a <button>
  await page.getByRole('button', { name: /log in/i }).click();
  
  // Verify callback was called
  expect(onSubmit).toHaveBeenCalledWith('user@example.com', 'password123');
});
```

### ARIA for Enhanced Accessibility (and Testability)

ARIA (Accessible Rich Internet Applications) attributes add semantics to HTML:

```javascript
// aria-label: Provides accessible name for elements without visible text
function IconButton({ icon, action }) {
  return (
    <button aria-label={action}>
      <span aria-hidden="true">{icon}</span>
    </button>
  );
}

// Testable:
test('icon button works', async ({ page }) => {
  await page.getByRole('button', { name: 'close' }).click();
});

// aria-hidden: Hides decorative elements from accessibility tree
// Useful for icons, decorative images, and visual separators

// aria-live: Announces dynamic changes to users
function SearchResults({ results, loading, error }) {
  return (
    <div aria-live="polite" aria-label="Search results">
      {loading && <p>Searching...</p>}
      {error && <p role="alert">{error}</p>}
      {results.length > 0 && (
        <ul>
          {results.map(r => <li key={r.id}>{r.name}</li>)}
        </ul>
      )}
    </div>
  );
}

// Testable:
test('results update after search', async ({ page }) => {
  await page.getByLabel('Search').fill('laptop');
  await page.getByRole('button', { name: /search/i }).click();
  
  // Wait for live region to announce results
  await expect(
    page.getByLabel('Search results').getByRole('listitem')
  ).toHaveCount(5);
});

// aria-describedby: Links elements to descriptions
<input
  id="email-input"
  type="email"
  aria-describedby="email-hint"
/>
<p id="email-hint">We'll use this to send you notifications</p>

// aria-expanded: Shows if something is expanded/collapsed
<button aria-expanded={isOpen} aria-controls="menu">
  Menu
</button>
<nav id="menu" hidden={!isOpen}>
  {/* Navigation items */}
</nav>

// aria-selected: Shows if an item is selected
<button
  aria-selected={isActive}
  role="tab"
  aria-controls={`panel-${id}`}
>
  Tab label
</button>

// aria-disabled: Alternative to disabled attribute for custom elements
<div
  role="button"
  aria-disabled={isDisabled}
  onClick={isDisabled ? undefined : handleClick}
>
  Custom Button
</div>
```

---

## Component Architecture

### Single Responsibility Principle

Each component should have one reason to change:

```javascript
// ❌ Too many responsibilities
function ProductPage({ productId, onAddToCart, userPreferences }) {
  const [product, setProduct] = useState(null);
  const [isInWishlist, setIsInWishlist] = useState(false);
  const [reviews, setReviews] = useState([]);
  const [selectedSize, setSelectedSize] = useState(null);
  const [quantity, setQuantity] = useState(1);
  const [cartError, setCartError] = useState(null);

  useEffect(() => {
    // Fetch product...
    // Fetch wishlist status...
    // Fetch reviews...
  }, [productId]);

  // 50+ lines of JSX mixing layout, forms, and data display
  return (/* ... */);
}

// ✅ Single responsibilities
function ProductPage({ productId }) {
  const product = useProduct(productId);
  
  return (
    <main>
      <ProductHeader product={product} />
      <ProductForm product={product} />
      <ReviewSection productId={productId} />
    </main>
  );
}

function ProductHeader({ product }) {
  return (
    <header>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <WishlistButton productId={product.id} />
    </header>
  );
}

function ProductForm({ product }) {
  const [selectedSize, setSelectedSize] = useState(null);
  const [quantity, setQuantity] = useState(1);
  const [error, setError] = useState(null);
  const { addToCart } = useCart();

  const handleAddToCart = async () => {
    try {
      await addToCart(product.id, { size: selectedSize, quantity });
    } catch (e) {
      setError(e.message);
    }
  };

  return (
    <form onSubmit={(e) => { e.preventDefault(); handleAddToCart(); }}>
      <SizeSelector value={selectedSize} onChange={setSelectedSize} />
      <QuantityInput value={quantity} onChange={setQuantity} />
      {error && <ErrorAlert message={error} />}
      <button type="submit">Add to Cart</button>
    </form>
  );
}
```

**Why this helps testing:** Each component is small and testable in isolation.

```javascript
test('product form adds item to cart', async ({ page }) => {
  // Test ProductForm without needing full ProductPage
  // Mount with props only
  const addToCart = await page.evaluate(() => {
    return window.cartCalls || [];
  });
  
  await page.getByLabel('Size').selectOption('Medium');
  await page.getByLabel('Quantity').fill('2');
  await page.getByRole('button', { name: /add to cart/i }).click();
  
  // Verify behavior
});
```

### Compound Components

Compound components are a set of components that work together with shared state:

```javascript
// ❌ Too many props to configure behavior
function Tabs({ tabs, activeTab, onTabChange, variant, size, disabled }) {
  return (
    <div role="tablist" className={`tabs-${variant}-${size}`}>
      {tabs.map(tab => (
        <button
          key={tab.id}
          role="tab"
          aria-selected={activeTab === tab.id}
          disabled={disabled}
          onClick={() => onTabChange(tab.id)}
        >
          {tab.label}
        </button>
      ))}
    </div>
  );
}

// ✅ Compound components share internal state
function Tabs({ children, defaultValue }) {
  const [value, setValue] = useState(defaultValue);

  return (
    <TabsContext.Provider value={{ value, setValue }}>
      <div role="tablist">{children}</div>
    </TabsContext.Provider>
  );
}

function TabsTrigger({ value, children }) {
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

function TabsContent({ value, children }) {
  const { value: activeValue } = useContext(TabsContext);

  return activeValue === value ? <div role="tabpanel">{children}</div> : null;
}

// Usage is cleaner and more flexible
<Tabs defaultValue="tab1">
  <div role="tablist">
    <TabsTrigger value="tab1">Overview</TabsTrigger>
    <TabsTrigger value="tab2">Details</TabsTrigger>
  </div>
  <TabsContent value="tab1">
    <p>Overview content</p>
  </TabsContent>
  <TabsContent value="tab2">
    <p>Details content</p>
  </TabsContent>
</Tabs>
```

**Testing compound components:**

```javascript
test('user can switch between tabs', async ({ page }) => {
  await page.getByRole('tab', { name: /overview/i }).click();
  await expect(page.getByRole('tabpanel')).toContainText('Overview content');

  await page.getByRole('tab', { name: /details/i }).click();
  await expect(page.getByRole('tabpanel')).toContainText('Details content');
});
```

### Avoiding Prop Drilling

When props pass through multiple component layers, it's hard to test and refactor:

```javascript
// ❌ Props drilling - theme prop passes through many layers
<App theme="dark">
  <Layout theme={theme}>
    <Sidebar theme={theme}>
      <Menu theme={theme}>
        <MenuItem theme={theme} />
      </Menu>
    </Sidebar>
  </Layout>
</App>

// ✅ Use Context to avoid drilling
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('dark');

  return (
    <ThemeContext.Provider value={theme}>
      <Layout>
        <Sidebar>
          <Menu>
            <MenuItem />
          </Menu>
        </Sidebar>
      </Layout>
    </ThemeContext.Provider>
  );
}

function MenuItem() {
  const theme = useContext(ThemeContext);
  return <li className={`item-${theme}`}>Item</li>;
}
```

**Why this helps testing:** Components are decoupled from their ancestors. Test them with different context values.

```javascript
test('menu item uses dark theme', async ({ page }) => {
  // Render with dark theme context
  // Verify styles
});

test('menu item uses light theme', async ({ page }) => {
  // Render with light theme context
  // Verify styles
});
```

---

## State Management

### Where Should State Live?

Ask these questions:

1. **Does multiple components need this state?** → Lift to common parent
2. **Is state derived from other state?** → Don't store it, compute it
3. **Does state exist for API calls?** → Keep it with the component that fetches
4. **Is state user-specific?** → Keep it in user context/session storage
5. **Is state application-wide?** → Use global state management

```javascript
// ❌ State in wrong places
function ProductPage() {
  const [product, setProduct] = useState(null);
  const [inWishlist, setInWishlist] = useState(false);

  return (
    <>
      <ProductDetails product={product} />
      <WishlistButton inWishlist={inWishlist} /> {/* Sibling can't share state */}
    </>
  );
}

// ✅ State in right place
function ProductPage() {
  const [product, setProduct] = useState(null);
  const [inWishlist, setInWishlist] = useState(false);

  return (
    <>
      <ProductDetails product={product} />
      <WishlistButton
        inWishlist={inWishlist}
        onWishlistChange={setInWishlist}
      />
    </>
  );
}

// Even better: Derive wishlist state from product
function ProductPage() {
  const [product, setProduct] = useState(null);

  return (
    <>
      <ProductDetails product={product} />
      <WishlistButton productId={product.id} /> {/* Fetches its own wishlist state */}
    </>
  );
}
```

### State Update Patterns

```javascript
// Updating scalar values
const [count, setCount] = useState(0);
setCount(count + 1); // or
setCount(prev => prev + 1); // Better: doesn't depend on closure

// Updating objects
const [user, setUser] = useState({ name: '', email: '' });
setUser({ ...user, name: 'John' }); // or
setUser(prev => ({ ...prev, name: 'John' }));

// Updating arrays
const [items, setItems] = useState([]);
setItems([...items, newItem]); // Add
setItems(items.filter(i => i.id !== id)); // Remove
setItems(items.map(i => i.id === id ? { ...i, done: true } : i)); // Update

// Complex state: Consider useReducer
function TodoList() {
  const [state, dispatch] = useReducer(
    (state, action) => {
      switch (action.type) {
        case 'ADD_TODO':
          return { ...state, todos: [...state.todos, action.payload] };
        case 'REMOVE_TODO':
          return {
            ...state,
            todos: state.todos.filter(t => t.id !== action.payload)
          };
        case 'TOGGLE_TODO':
          return {
            ...state,
            todos: state.todos.map(t =>
              t.id === action.payload ? { ...t, done: !t.done } : t
            )
          };
        default:
          return state;
      }
    },
    { todos: [] }
  );

  return (
    <div>
      {state.todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} dispatch={dispatch} />
      ))}
    </div>
  );
}
```

### Managing Async State

```javascript
// ❌ Messy with multiple useState
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetchUsers()
      .then(data => {
        setUsers(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, []);

  // Rendering logic handles all three states
}

// ✅ Cleaner with custom hook
function useAsyncData(fetchFn) {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null
  });

  useEffect(() => {
    fetchFn()
      .then(data => setState({ data, loading: false, error: null }))
      .catch(error => setState({ data: null, loading: false, error }));
  }, [fetchFn]);

  return state;
}

function UserList() {
  const { data: users, loading, error } = useAsyncData(fetchUsers);

  if (loading) return <p>Loading...</p>;
  if (error) return <p role="alert">Error: {error.message}</p>;
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

---

## Props vs State

### Decision Tree

```
Is this data rendered?
├─ No → Don't store it (or store in a ref)
└─ Yes → Does it change?
   ├─ No → Pass as prop (static)
   └─ Yes → Who can change it?
      ├─ Only this component → useState
      ├─ This and siblings → useState in parent
      ├─ This and distant ancestors → useContext or global state
      └─ API/external → useState for caching
```

### Examples

```javascript
// Props: Configuration that doesn't change
function Button({ label, variant, size, disabled }) {
  return (
    <button className={`btn-${variant}-${size}`} disabled={disabled}>
      {label}
    </button>
  );
}

// State: Data that changes based on user interaction
function SearchBox() {
  const [query, setQuery] = useState('');
  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      aria-label="Search"
    />
  );
}

// Props + State: A mix
function SearchResults({ limit }) {
  // Props: limit is configuration from parent
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  useEffect(() => {
    const results = search(query).slice(0, limit);
    setResults(results);
  }, [query, limit]);

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {results.map(r => <li key={r.id}>{r.name}</li>)}
      </ul>
    </div>
  );
}
```

### Derived State is Often a Mistake

```javascript
// ❌ Storing derived state - can get out of sync
function UserProfile({ firstName, lastName }) {
  const [fullName, setFullName] = useState(`${firstName} ${lastName}`);

  useEffect(() => {
    setFullName(`${firstName} ${lastName}`);
  }, [firstName, lastName]);

  return <h1>{fullName}</h1>;
}

// ✅ Derive it instead
function UserProfile({ firstName, lastName }) {
  const fullName = `${firstName} ${lastName}`;
  return <h1>{fullName}</h1>;
}

// ❌ Storing total - changes when list changes
function ShoppingCart({ items }) {
  const [total, setTotal] = useState(0);

  useEffect(() => {
    setTotal(items.reduce((sum, item) => sum + item.price * item.qty, 0));
  }, [items]);

  return <p>Total: ${total}</p>;
}

// ✅ Derive it instead
function ShoppingCart({ items }) {
  const total = items.reduce((sum, item) => sum + item.price * item.qty, 0);
  return <p>Total: ${total}</p>;
}
```

---

## Data Flow Patterns

### One-Way Data Flow (Parent to Child)

Data flows down through props. This makes behavior predictable:

```javascript
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <Child count={count} /> {/* Data down */}
    </div>
  );
}

function Child({ count }) {
  return <p>Count: {count}</p>;
}
```

**Testing is straightforward:**

```javascript
test('child displays count from parent', async ({ page }) => {
  // Parent renders with count=5
  await expect(page.getByText('Count: 5')).toBeVisible();
});
```

### Inverse Data Flow (Child to Parent)

Children communicate changes through callbacks:

```javascript
function Parent() {
  const [count, setCount] = useState(0);

  const handleIncrement = () => setCount(prev => prev + 1);

  return (
    <div>
      <p>Count: {count}</p>
      <Child onIncrement={handleIncrement} /> {/* Callback down */}
    </div>
  );
}

function Child({ onIncrement }) {
  return (
    <button onClick={onIncrement}>
      Increment
    </button>
  );
}
```

**Testing:**

```javascript
test('clicking child button updates parent count', async ({ page }) => {
  await expect(page.getByText('Count: 0')).toBeVisible();
  await page.getByRole('button', { name: /increment/i }).click();
  await expect(page.getByText('Count: 1')).toBeVisible();
});
```

### Bidirectional Props

For controlled inputs, prop defines value, callback handles changes:

```javascript
function SearchForm({ query, onQueryChange, onSubmit }) {
  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={query}
        onChange={(e) => onQueryChange(e.target.value)}
        placeholder="Search..."
      />
      <button type="submit">Search</button>
    </form>
  );
}

// Parent controls state
function App() {
  const [query, setQuery] = useState('');

  const handleSearch = () => {
    console.log('Searching for:', query);
  };

  return (
    <SearchForm
      query={query}
      onQueryChange={setQuery}
      onSubmit={handleSearch}
    />
  );
}
```

---

## Event Handlers & Forms

### Form Basics with Semantic HTML

```javascript
function LoginForm({ onSubmit }) {
  const [formData, setFormData] = useState({ email: '', password: '' });
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Validate
    const newErrors = {};
    if (!formData.email) newErrors.email = 'Email is required';
    if (!formData.password) newErrors.password = 'Password is required';

    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

    // Submit
    try {
      await onSubmit(formData);
    } catch (error) {
      setErrors({ submit: error.message });
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          required
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <p id="email-error" role="alert">
            {errors.email}
          </p>
        )}
      </div>

      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
          required
          aria-describedby={errors.password ? 'password-error' : undefined}
        />
        {errors.password && (
          <p id="password-error" role="alert">
            {errors.password}
          </p>
        )}
      </div>

      {errors.submit && (
        <p role="alert">{errors.submit}</p>
      )}

      <button type="submit">Log In</button>
    </form>
  );
}
```

**Testing:**

```javascript
test('user can log in with valid credentials', async ({ page }) => {
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: /log in/i }).click();
  
  await expect(page).toHaveURL('/dashboard');
});

test('shows error for invalid email', async ({ page }) => {
  await page.getByLabel('Email').fill('invalid');
  await page.getByRole('button', { name: /log in/i }).click();
  
  await expect(page.getByRole('alert')).toContainText('Email is required');
});
```

### Controlled vs Uncontrolled Components

```javascript
// Controlled: React state drives the value
function ControlledInput() {
  const [value, setValue] = useState('');

  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}

// Uncontrolled: DOM manages the value
function UncontrolledInput() {
  const inputRef = useRef(null);

  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };

  return (
    <>
      <input ref={inputRef} defaultValue="" />
      <button onClick={handleSubmit}>Submit</button>
    </>
  );
}
```

**For E2E testing, prefer controlled.** Uncontrolled components are harder to test because you can't easily set their value.

### Custom Hooks for Complex Forms

```javascript
// Custom hook for form state
function useForm(initialValues, onSubmit) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    try {
      await onSubmit(values);
    } catch (error) {
      setErrors({ submit: error.message });
    } finally {
      setIsSubmitting(false);
    }
  };

  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
  };
}

// Usage
function MyForm({ onSubmit }) {
  const form = useForm(
    { email: '', password: '' },
    onSubmit
  );

  return (
    <form onSubmit={form.handleSubmit}>
      <input
        name="email"
        type="email"
        value={form.values.email}
        onChange={form.handleChange}
        onBlur={form.handleBlur}
      />
      <button type="submit" disabled={form.isSubmitting}>
        Submit
      </button>
    </form>
  );
}
```

---

## Composition Over Configuration

Rather than boolean props that change behavior, use composition:

```javascript
// ❌ Boolean prop proliferation - hard to understand all combinations
function Button({
  variant,
  size,
  disabled,
  loading,
  fullWidth,
  withIcon,
  iconPosition,
  hasDropdown,
  // 10+ more props to handle variations
}) {
  return (
    <button
      className={`btn-${variant}-${size} ${fullWidth ? 'full-width' : ''}`}
      disabled={disabled || loading}
    >
      {loading && <Spinner />}
      {withIcon && <Icon position={iconPosition} />}
      {/* Complex JSX to handle all combinations */}
    </button>
  );
}

// Usage is confusing - what do these combinations mean?
<Button variant="primary" size="lg" withIcon iconPosition="left" hasDropdown />

// ✅ Composition - clear intent
function Button({ children, disabled, ...props }) {
  return (
    <button disabled={disabled} {...props}>
      {children}
    </button>
  );
}

// Variants are separate components or styles
function PrimaryButton({ children, ...props }) {
  return <Button className="btn-primary" {...props}>{children}</Button>;
}

function LargeButton({ children, ...props }) {
  return <Button className="btn-lg" {...props}>{children}</Button>;
}

// Composition for complex button
<LargeButton disabled>
  <Icon name="search" />
  Search
</LargeButton>

// Or use the base Button directly with className
<Button className="btn-primary btn-lg" disabled>
  <Icon name="search" />
  Search
</Button>
```

### Render Props for Flexible Layouts

```javascript
// ❌ Component tries to handle all layout scenarios
function UserCard({ user, showBio, showStats, showActions }) {
  return (
    <div className="card">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      {showBio && <p>{user.bio}</p>}
      {showStats && (
        <div>
          <span>Followers: {user.followers}</span>
          <span>Posts: {user.posts}</span>
        </div>
      )}
      {showActions && (
        <div>
          <button>Follow</button>
          <button>Message</button>
        </div>
      )}
    </div>
  );
}

// ✅ Children for flexible composition
function UserCard({ user, children }) {
  return (
    <div className="card">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      {children}
    </div>
  );
}

// Usage is flexible
<UserCard user={user}>
  <p>{user.bio}</p>
  <div>
    <span>Followers: {user.followers}</span>
  </div>
  <button>Follow</button>
</UserCard>

// Or render props for more control
function UserCard({ user, renderContent }) {
  return (
    <div className="card">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      {renderContent(user)}
    </div>
  );
}

<UserCard
  user={user}
  renderContent={(u) => (
    <>
      <p>{u.bio}</p>
      <button>Follow</button>
    </>
  )}
/>
```

---

## Testing Patterns

### Testing Philosophy

Write tests that verify user-facing behavior, not implementation:

```javascript
// ❌ Tests implementation
test('search box state is updated', async ({ page }) => {
  const input = page.getByRole('textbox');
  await input.fill('laptop');
  expect(await input.inputValue()).toBe('laptop');
});

// ✅ Tests user behavior
test('user can search for products', async ({ page }) => {
  await page.getByLabel('Search products').fill('laptop');
  await page.getByRole('button', { name: /search/i }).click();
  
  // User sees results
  await expect(
    page.getByRole('heading', { level: 2 })
  ).toContainText('laptop');
});
```

### Testing Guidelines

1. **Use semantic queries** - getByRole, getByLabel, getByText
2. **Avoid implementation details** - Don't test CSS classes or component state
3. **Test user workflows** - Not individual component state
4. **Async operations** - Use waitFor, expect with retry
5. **Accessibility equals testability** - Proper HTML makes tests work

### Common Patterns

```javascript
// Testing button clicks
test('button click works', async ({ page }) => {
  await page.getByRole('button', { name: /submit/i }).click();
  // Assert what happens after click
});

// Testing forms
test('form submits with values', async ({ page }) => {
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('pass123');
  await page.getByRole('button', { name: /log in/i }).click();
  // Assert success
});

// Testing dynamic content
test('list updates when item added', async ({ page }) => {
  await page.getByRole('button', { name: /add item/i }).click();
  
  // Use waitFor for async updates
  await expect(
    page.getByRole('listitem').last()
  ).toContainText('New item');
});

// Testing visibility
test('error message shows on validation', async ({ page }) => {
  await page.getByRole('button', { name: /submit/i }).click();
  
  await expect(page.getByRole('alert')).toBeVisible();
  await expect(page.getByRole('alert')).toContainText('Email is required');
});

// Testing navigation
test('clicking link navigates', async ({ page }) => {
  await page.getByRole('link', { name: /about/i }).click();
  
  await expect(page).toHaveURL('/about');
});

// Testing disabled states
test('submit button is disabled until form is valid', async ({ page }) => {
  const submitBtn = page.getByRole('button', { name: /submit/i });
  
  // Initially disabled
  await expect(submitBtn).toBeDisabled();
  
  // Enable after input
  await page.getByLabel('Email').fill('user@example.com');
  await expect(submitBtn).toBeEnabled();
});
```

---

## Summary: The Testing Mindset

When building React components with E2E testing in mind:

1. **Start with semantic HTML** - Use buttons, labels, forms, headings
2. **Add ARIA where needed** - aria-label, aria-live, roles
3. **Build clear component hierarchy** - Single responsibility
4. **Make data flow unidirectional** - Props down, callbacks up
5. **Use composition over configuration** - Fewer boolean props
6. **Test user behavior** - Not implementation details

This isn't special work for testing. It's just good React architecture. The bonus is that your code becomes trivial to E2E test with Playwright.

---

## Next Steps

- See **EXAMPLES.md** for 30+ production-ready component examples
- See **ANTI-PATTERNS.md** for patterns that break E2E tests
- See **ACCESSIBILITY-FOR-TESTING.md** for the accessibility ↔ testability connection
- See **COMPONENT-ARCHITECTURE.md** for deep dives on component patterns
- See **QUICKSTART.md** for simple templates to get started
