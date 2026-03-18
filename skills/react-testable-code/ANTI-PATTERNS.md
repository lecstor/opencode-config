# Anti-Patterns: What Breaks E2E Tests

This document covers 8 common React patterns that make E2E testing difficult or impossible. Each pattern shows why it breaks tests and how to fix it.

---

## Anti-Pattern 1: Divs with onClick Instead of Buttons

### The Problem

```javascript
// ❌ Not semantic, not testable
function DeleteButton({ onDelete }) {
  return (
    <div
      className="delete-btn"
      onClick={onDelete}
      role="button"  // Adding role doesn't fix it - divs aren't buttons
    >
      Delete
    </div>
  );
}
```

**Why it breaks tests:**
- `getByRole('button')` won't find it (it's a div, not a button)
- It's not keyboard accessible (no Enter/Space support)
- Screen readers don't announce it properly
- It looks like a button but doesn't act like one

**Playwright test fails:**
```javascript
test('delete button works', async ({ page }) => {
  // This fails - no button found
  await page.getByRole('button', { name: /delete/i }).click();
});
```

### The Fix

```javascript
// ✅ Semantic button, fully testable
function DeleteButton({ onDelete }) {
  return (
    <button onClick={onDelete} className="delete-btn">
      Delete
    </button>
  );
}
```

**Test passes:**
```javascript
test('delete button works', async ({ page }) => {
  await page.getByRole('button', { name: /delete/i }).click();
  
  // Assert what happens
  await expect(page.getByText(/item deleted/i)).toBeVisible();
});
```

**Keyboard navigation also works automatically:**
- User can Tab to the button
- User can press Enter or Space to activate it
- Screen readers announce it correctly

---

## Anti-Pattern 2: Missing Labels on Form Inputs

### The Problem

```javascript
// ❌ No label, input is unfindable
function SearchForm() {
  const [query, setQuery] = useState('');

  return (
    <form>
      <input
        type="text"
        placeholder="Search..."
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <button type="submit">Go</button>
    </form>
  );
}
```

**Why it breaks tests:**
- `getByLabel()` can't find the input (no label exists)
- `getByPlaceholderText()` works but is fragile (placeholders change with design)
- Visually impaired users can't understand what the input is for
- No programmatic association between label and input

**Playwright test is fragile:**
```javascript
test('user can search', async ({ page }) => {
  // Relying on placeholder is brittle
  await page.getByPlaceholderText('Search...').fill('laptop');
  
  // What if designer changes placeholder text? Test breaks.
});
```

### The Fix

```javascript
// ✅ Proper label, fully testable
function SearchForm() {
  const [query, setQuery] = useState('');

  return (
    <form>
      <label htmlFor="search-input">Search</label>
      <input
        id="search-input"
        type="text"
        placeholder="e.g., laptop"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <button type="submit">Go</button>
    </form>
  );
}
```

**Test is robust:**
```javascript
test('user can search', async ({ page }) => {
  // This is resilient to placeholder changes
  await page.getByLabel('Search').fill('laptop');
  await page.getByRole('button', { name: /go/i }).click();
  
  // Test passes regardless of placeholder text
});
```

---

## Anti-Pattern 3: Using Test IDs Instead of Semantic HTML

### The Problem

```javascript
// ❌ Tied to test infrastructure, brittle refactoring
function ProductCard({ product }) {
  return (
    <div data-testid="product-card">
      <div data-testid="product-name">{product.name}</div>
      <div data-testid="product-price">${product.price}</div>
      <div
        data-testid="add-to-cart-btn"
        onClick={() => addToCart(product)}
        className="btn"
      >
        Add to Cart
      </div>
    </div>
  );
}
```

**Why it breaks tests:**
- Tests depend on implementation details (data-testid values)
- Renaming data-testid breaks tests
- You can refactor CSS but not the component structure
- Tests read like implementation, not user behavior
- Developers who don't write tests don't know what tests depend on

**Tests are fragile:**
```javascript
test('can add product to cart', async ({ page }) => {
  // If you rename data-testid="add-to-cart-btn", test breaks
  // This test reads like code, not user behavior
  await page.getByTestId('add-to-cart-btn').click();
});
```

### The Fix

```javascript
// ✅ Semantic HTML, resilient to refactoring
function ProductCard({ product }) {
  return (
    <article>
      <h2>{product.name}</h2>
      <p>${product.price}</p>
      <button onClick={() => addToCart(product)}>
        Add to Cart
      </button>
    </article>
  );
}
```

**Tests are resilient:**
```javascript
test('can add product to cart', async ({ page }) => {
  // Finds button by role and name - works with any styling/refactoring
  await page.getByRole('button', { name: /add to cart/i }).click();
  
  // This test reads like user behavior
  await expect(page.getByText(/added to cart/i)).toBeVisible();
});
```

---

## Anti-Pattern 4: Missing Accessible Names on Icon Buttons

### The Problem

```javascript
// ❌ Icon-only button with no accessible name
function IconButton({ icon, onClick }) {
  return (
    <button onClick={onClick} className="icon-btn">
      {icon}
    </button>
  );
}

// Usage
<IconButton icon="🔍" onClick={handleSearch} />
<IconButton icon="❌" onClick={handleClose} />
<IconButton icon="⋮" onClick={handleMenu} />
```

**Why it breaks tests:**
- `getByRole('button', { name: /search/i })` can't find it (no accessible name)
- Screen readers read it as just "button" (confusing)
- You don't know which button is which in tests
- Emoji are not reliable accessible names

**Playwright test fails:**
```javascript
test('user can search by clicking icon', async ({ page }) => {
  // This fails - button has no accessible name
  await page.getByRole('button', { name: /search/i }).click();
});
```

### The Fix

```javascript
// ✅ Icon button with aria-label
function IconButton({ icon, label, onClick }) {
  return (
    <button onClick={onClick} aria-label={label} className="icon-btn">
      <span aria-hidden="true">{icon}</span>
    </button>
  );
}

// Usage
<IconButton icon="🔍" label="Search" onClick={handleSearch} />
<IconButton icon="❌" label="Close" onClick={handleClose} />
<IconButton icon="⋮" label="Menu" onClick={handleMenu} />
```

**Test works:**
```javascript
test('user can search by clicking icon', async ({ page }) => {
  await page.getByRole('button', { name: /search/i }).click();
  await expect(page.getByText(/results/i)).toBeVisible();
});

test('user can close by clicking icon', async ({ page }) => {
  await page.getByRole('button', { name: /close/i }).click();
  await expect(page.getByRole('dialog')).not.toBeVisible();
});
```

---

## Anti-Pattern 5: State in the Wrong Component

### The Problem

```javascript
// ❌ State in wrong place - parent can't see child's state
function ProductPage() {
  return (
    <div>
      <ProductDetails />
      <Reviews /> {/* Can't share state with ProductDetails */}
    </div>
  );
}

function ProductDetails({ productId }) {
  const [isFavorited, setIsFavorited] = useState(false);
  
  return (
    <div>
      <button onClick={() => setIsFavorited(!isFavorited)}>
        {isFavorited ? '❤️' : '🤍'} Favorite
      </button>
    </div>
  );
}

function Reviews({ productId }) {
  const [isFavorited, setIsFavorited] = useState(false);
  // Different state! Can get out of sync
}
```

**Why it breaks tests:**
- Tests can't verify consistent state across components
- State changes in one component don't affect the other
- Data is duplicated and can diverge
- Impossible to verify that clicking favorite updates related UI

**Test fails:**
```javascript
test('favoriting product updates all sections', async ({ page }) => {
  const favoriteBtn = page.getByRole('button', { name: /favorite/i }).first();
  await favoriteBtn.click();

  // This won't work - Reviews component has its own state
  await expect(
    page.getByRole('button', { name: /favorite/i }).last()
  ).toHaveAttribute('aria-pressed', 'true');
});
```

### The Fix

```javascript
// ✅ State in parent (or shared context)
function ProductPage({ productId }) {
  const [isFavorited, setIsFavorited] = useState(false);

  return (
    <div>
      <ProductDetails
        productId={productId}
        isFavorited={isFavorited}
        onFavoriteChange={setIsFavorited}
      />
      <Reviews
        productId={productId}
        isFavorited={isFavorited}
        onFavoriteChange={setIsFavorited}
      />
    </div>
  );
}

function ProductDetails({ isFavorited, onFavoriteChange }) {
  return (
    <button
      onClick={() => onFavoriteChange(!isFavorited)}
      aria-pressed={isFavorited}
    >
      {isFavorited ? '❤️' : '🤍'} Favorite
    </button>
  );
}

function Reviews({ isFavorited, onFavoriteChange }) {
  return (
    <div>
      {isFavorited && <p>You favorited this!</p>}
    </div>
  );
}
```

**Test works:**
```javascript
test('favoriting product updates all sections', async ({ page }) => {
  const favoriteBtn = page.getByRole('button', { name: /favorite/i });
  await favoriteBtn.click();

  // Both components see the updated state
  await expect(favoriteBtn).toHaveAttribute('aria-pressed', 'true');
  await expect(page.getByText(/you favorited this/i)).toBeVisible();
});
```

---

## Anti-Pattern 6: Props Drilling Too Deep

### The Problem

```javascript
// ❌ Props drilling through many components
<App>
  <Layout>
    <Header>
      <Navigation>
        <UserMenu user={user} />
      </Navigation>
    </Header>
  </Layout>
</App>

// Each component just passes props down
function Header({ user }) {
  return <Navigation user={user} />;
}

function Navigation({ user }) {
  return <UserMenu user={user} />;
}

function UserMenu({ user }) {
  return <button>{user.name}</button>;
}
```

**Why it breaks tests:**
- Hard to understand data flow
- Refactoring structure requires changing many components
- Tests can't verify intermediate components
- Easy to forget to pass props when refactoring

**Tests become tightly coupled to structure:**
```javascript
test('user menu shows username', async ({ page }) => {
  // Have to verify through App > Layout > Header > Navigation > UserMenu
  // If structure changes, test fails even if behavior is the same
});
```

### The Fix

```javascript
// ✅ Use Context to avoid drilling
const UserContext = createContext();

function App() {
  const [user, setUser] = useState(null);

  return (
    <UserContext.Provider value={user}>
      <Layout>
        <Header>
          <Navigation />
        </Header>
      </Layout>
    </UserContext.Provider>
  );
}

// Components consume directly, no drilling
function Header() {
  return <Navigation />;
}

function Navigation() {
  return <UserMenu />;
}

function UserMenu() {
  const user = useContext(UserContext);
  return <button>{user.name}</button>;
}
```

**Tests verify behavior, not structure:**
```javascript
test('user menu shows username', async ({ page }) => {
  // Works regardless of component structure
  await expect(page.getByRole('button', { name: /john/i })).toBeVisible();
});
```

---

## Anti-Pattern 7: Boolean Prop Proliferation

### The Problem

```javascript
// ❌ Too many boolean props - hard to understand combinations
function Button({
  variant,
  size,
  disabled,
  loading,
  fullWidth,
  outline,
  withIcon,
  iconPosition,
  hasDropdown,
  // 10 more props...
}) {
  return (
    <button
      className={`btn btn-${variant} btn-${size} ${fullWidth ? 'full' : ''} ${
        outline ? 'outline' : ''
      }`}
      disabled={disabled || loading}
    >
      {loading && <Spinner />}
      {/* Complex logic to handle all combinations */}
    </button>
  );
}

// Usage is unclear
<Button variant="primary" size="lg" outline loading fullWidth withIcon />
```

**Why it breaks tests:**
- Hard to test all combinations
- Props have interdependencies (loading disables button, but so does disabled)
- Interface is confusing - developers don't know what's valid
- Brittle - changes to one prop affect multiple tests

**Tests are hard to write:**
```javascript
test('button shows loading state', async ({ page }) => {
  // Which props combine to show loading? variant + loading?
  // variant + loading + size? variant + loading + outline?
  // Tests multiply - 2^10 combinations of 10 boolean props
});
```

### The Fix

```javascript
// ✅ Composition - clear intent
function Button({ children, disabled, className, ...props }) {
  return (
    <button disabled={disabled} className={className} {...props}>
      {children}
    </button>
  );
}

// Separate components for variants
function PrimaryButton({ children, ...props }) {
  return <Button className="btn-primary" {...props}>{children}</Button>;
}

function OutlineButton({ children, ...props }) {
  return <Button className="btn-outline" {...props}>{children}</Button>;
}

// Size variants
function LargeButton({ children, ...props }) {
  return <Button className="btn-lg" {...props}>{children}</Button>;
}

// Usage is clear
<PrimaryButton onClick={handleClick}>
  <Spinner aria-hidden="true" />
  Loading...
</PrimaryButton>

<OutlineButton disabled>Cancel</OutlineButton>

<LargeButton className="full-width">Click me</LargeButton>
```

**Tests are simple and clear:**
```javascript
test('primary button works', async ({ page }) => {
  await page.getByRole('button', { name: /click me/i }).click();
});

test('disabled button cannot be clicked', async ({ page }) => {
  await expect(page.getByRole('button', { name: /cancel/i })).toBeDisabled();
});
```

---

## Anti-Pattern 8: Missing Error Boundaries and No Error States

### The Problem

```javascript
// ❌ No error handling, crashes silently
function UserList() {
  const [users, setUsers] = useState([]);
  const [error, setError] = useState(null); // Created but not used

  useEffect(() => {
    // No error handling
    fetch('/api/users')
      .then(r => r.json())
      .then(data => setUsers(data));
    // If fetch fails, nothing happens
  }, []);

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

**Why it breaks tests:**
- Tests don't know if loading failed
- No way to verify error states
- Network errors silently fail
- User sees a blank page with no indication why

**Tests fail mysteriously:**
```javascript
test('shows user list', async ({ page }) => {
  // If API fails, page stays blank - test times out
  await expect(page.getByRole('listitem')).not.toBeEmpty();
  // Test fails but not for clear reasons
});
```

### The Fix

```javascript
// ✅ Proper error handling with clear states
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUsers = async () => {
      setLoading(true);
      setError(null);
      try {
        const response = await fetch('/api/users');
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  if (loading) {
    return <p aria-busy="true">Loading users...</p>;
  }

  if (error) {
    return (
      <div role="alert">
        <p>Error loading users: {error}</p>
        <button onClick={() => window.location.reload()}>Try Again</button>
      </div>
    );
  }

  if (users.length === 0) {
    return <p>No users found.</p>;
  }

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

**Tests are clear:**
```javascript
test('shows loading state', async ({ page }) => {
  await expect(page.getByText(/loading users/i)).toBeVisible();
});

test('shows error state on fetch failure', async ({ page }) => {
  // Mock API to fail...
  await expect(page.getByRole('alert')).toBeVisible();
  await expect(page.getByRole('alert')).toContainText(/error loading users/i);
});

test('shows user list on success', async ({ page }) => {
  await expect(page.getByRole('listitem')).not.toBeEmpty();
});
```

---

## Summary: Why These Anti-Patterns Break Tests

| Anti-Pattern | Why It Breaks | How to Fix |
|---|---|---|
| Divs with onClick | Not semantic, not keyboard accessible | Use `<button>` |
| Missing labels | Inputs unfindable via getByLabel | Use `<label htmlFor>` |
| Test IDs only | Brittle, tied to implementation | Use semantic HTML |
| Icon buttons no label | Can't find button by name | Add `aria-label` |
| State in wrong place | Sibling state gets out of sync | Lift state to parent |
| Props drilling | Hard to refactor, tight coupling | Use Context |
| Boolean props | Combinatorial explosion, unclear API | Use composition |
| No error handling | Silent failures, unclear states | Add error boundaries & states |

**The core principle:** These anti-patterns break tests because they break the contract between the component interface and E2E testing. When you use semantic HTML, proper accessibility, and clear component patterns, tests work automatically.

---

## Next Steps

- See **SKILL.md** for how to build components right from the start
- See **ACCESSIBILITY-FOR-TESTING.md** for why accessibility = testability
- See **EXAMPLES.md** for correct patterns
