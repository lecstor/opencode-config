# Component Architecture: How to Think About Component Design

> When your component architecture is clear, testing becomes trivial. This guide covers how to think about component boundaries, composition, and data flow.

---

## Component Boundaries: The Single Responsibility Principle

Each component should have one reason to change.

### Good Component Boundaries

```javascript
// ✅ Each component has one responsibility
function ProductList({ products, onProductSelect }) {
  return (
    <ul>
      {products.map(product => (
        <ProductListItem
          key={product.id}
          product={product}
          onSelect={onProductSelect}
        />
      ))}
    </ul>
  );
}

function ProductListItem({ product, onSelect }) {
  return (
    <li>
      <button onClick={() => onSelect(product.id)}>
        {product.name}
      </button>
    </li>
  );
}

function ProductDetails({ productId, product }) {
  return (
    <div>
      <h2>{product.name}</h2>
      <p>{product.description}</p>
      <Price price={product.price} />
    </div>
  );
}

function Price({ price }) {
  return <span>${price.toFixed(2)}</span>;
}
```

### Bad Component Boundaries

```javascript
// ❌ Component does too much
function ProductPage({ productId, onAddToCart, userPreferences, theme }) {
  // 50+ lines mixing list, details, forms, data fetching, styling...
  return (
    <div className={`page-${theme}`}>
      {/* List section */}
      <ul>
        {/* List items here */}
      </ul>

      {/* Details section */}
      <div>
        {/* Details here */}
      </div>

      {/* Add to cart form */}
      <form>
        {/* Form fields */}
      </form>

      {/* Wishlist section */}
      <div>
        {/* Wishlist content */}
      </div>
    </div>
  );
}
```

**Why good boundaries help testing:**

```javascript
test('user can select product', async ({ page }) => {
  // With good boundaries, test is focused:
  // ProductList is responsible for rendering items
  // ProductListItem is responsible for click handling
  // ProductDetails is responsible for showing details
  
  const productBtn = page.getByRole('button', { name: /first product/i });
  await productBtn.click();
  
  // Only test what ProductListItem does
  // Not the entire page
});

test('product details shows correct information', async ({ page }) => {
  // Test ProductDetails independently
  // Verify it displays the right data
  
  await expect(page.getByRole('heading', { level: 2 })).toContainText('Product Name');
  await expect(page.getByText(/\$99\.99/)).toBeVisible();
});
```

---

## Composition Over Inheritance

React doesn't use inheritance. Instead, use composition: combining components together.

### Component Composition Example

```javascript
// Base button component
function Button({ children, disabled, onClick, ...props }) {
  return (
    <button onClick={onClick} disabled={disabled} {...props}>
      {children}
    </button>
  );
}

// Variant: Primary button
function PrimaryButton({ children, ...props }) {
  return <Button className="btn-primary" {...props}>{children}</Button>;
}

// Variant: Danger button
function DangerButton({ children, ...props }) {
  return <Button className="btn-danger" {...props}>{children}</Button>;
}

// Composition: Button with icon
function IconButton({ icon, label, ...props }) {
  return (
    <Button aria-label={label} {...props}>
      <span aria-hidden="true">{icon}</span>
      <span>{label}</span>
    </Button>
  );
}

// Composition: Button with loading state
function LoadingButton({ isLoading, children, ...props }) {
  return (
    <Button disabled={isLoading} {...props}>
      {isLoading && <Spinner />}
      {children}
    </Button>
  );
}

// Complex: Combine multiple
<PrimaryButton onClick={handleSubmit}>
  <IconButton icon="✓" label="Save" />
  Save
</PrimaryButton>

// Actually better - simpler composition:
<PrimaryButton onClick={handleSubmit}>
  Save
</PrimaryButton>
```

---

## Thinking About State: Where Should It Live?

State should live as close as possible to where it's used, but it must be accessible to all components that need it.

### Decision Tree for State Placement

```
Does this state change? 
├─ No → It's a constant, pass it as a prop
└─ Yes → Who needs to know when it changes?
   ├─ Only this component → useState in this component
   ├─ This component and children → useState here, pass down as props
   ├─ This component and siblings → useState in parent
   ├─ Many unrelated components → Consider Context or global state
   └─ Persisted data (API) → useState in root or Context
```

### Example: Filter State

```javascript
// ❌ Wrong: Filter state in the filter component, but parent needs to filter
function ProductSearch() {
  return (
    <>
      <FilterPanel /> {/* Has its own state */}
      <ProductResults /> {/* Can't see filter state */}
    </>
  );
}

function FilterPanel() {
  const [category, setCategory] = useState('all');
  // Parent has no way to know the filter changed
}

// ✅ Right: Filter state in parent
function ProductSearch() {
  const [filters, setFilters] = useState({ category: 'all' });

  const handleFilterChange = (newFilters) => {
    setFilters(newFilters);
  };

  const filteredProducts = applyFilters(allProducts, filters);

  return (
    <>
      <FilterPanel filters={filters} onFilterChange={handleFilterChange} />
      <ProductResults products={filteredProducts} />
    </>
  );
}

function FilterPanel({ filters, onFilterChange }) {
  return (
    <select
      value={filters.category}
      onChange={(e) => onFilterChange({ ...filters, category: e.target.value })}
    >
      {/* Options */}
    </select>
  );
}
```

**Testing shows why state placement matters:**

```javascript
test('filtering updates results', async ({ page }) => {
  // With state in parent:
  // When user changes filter, parent state updates
  // Results component re-renders with new data
  // Test can verify entire workflow
  
  await page.getByLabel('Category').selectOption('electronics');
  
  // Results update
  await expect(page.getByRole('heading')).toContainText('electronics');
});
```

---

## Context vs Props: When to Use Each

### Props: Direct Relationships

Use props when there's a direct parent-child relationship:

```javascript
// Props work well for direct relationships
function UserProfile({ user, onFollowClick }) {
  return (
    <div>
      <h1>{user.name}</h1>
      <UserBio bio={user.bio} />
      <FollowButton onFollowClick={onFollowClick} />
    </div>
  );
}

function UserBio({ bio }) {
  return <p>{bio}</p>;
}

function FollowButton({ onFollowClick }) {
  return <button onClick={onFollowClick}>Follow</button>;
}
```

**Testing is straightforward:**

```javascript
test('can follow user', async ({ page }) => {
  await page.getByRole('button', { name: /follow/i }).click();
  // Verify follow happened
});
```

### Context: Distant Relationships

Use Context when many components need the same data without passing it through every level:

```javascript
// Problem: Props drilling through many layers
<App user={user}>
  <Layout user={user}>
    <Header user={user}>
      <Navigation user={user}>
        <UserMenu user={user} />
      </Navigation>
    </Header>
  </Layout>
</App>

// Solution: Context
const UserContext = createContext();

function App({ user }) {
  return (
    <UserContext.Provider value={user}>
      <Layout>
        <Header>
          <Navigation>
            <UserMenu />
          </Navigation>
        </Header>
      </Layout>
    </UserContext.Provider>
  );
}

function UserMenu() {
  const user = useContext(UserContext);
  return <button>{user.name}</button>;
}
```

**Testing with Context:**

```javascript
// Create a test wrapper that provides context
function TestUserProvider({ children, user }) {
  return (
    <UserContext.Provider value={user}>
      {children}
    </UserContext.Provider>
  );
}

test('user menu shows username', async ({ page }) => {
  // With wrapper, test is simple
  render(
    <TestUserProvider user={{ name: 'John' }}>
      <UserMenu />
    </TestUserProvider>
  );
  
  await expect(page.getByRole('button')).toContainText('John');
});
```

### Multiple Contexts

```javascript
// Theme context
const ThemeContext = createContext('light');

// User context
const UserContext = createContext(null);

// Combine them
function App() {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState(null);

  return (
    <ThemeContext.Provider value={theme}>
      <UserContext.Provider value={user}>
        <Layout>
          {/* Deeply nested components can use either context */}
        </Layout>
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}
```

---

## Custom Hooks: Extracting Logic

Custom hooks let you extract state and logic into reusable units:

```javascript
// ✅ Extract fetching logic into a hook
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const json = await response.json();
        setData(json);
      } catch (err) {
        setError(err);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// Now use it in many components
function UserList() {
  const { data: users, loading, error } = useFetch('/api/users');

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name}</li>)}
    </ul>
  );
}

// ✅ Extract form logic into a hook
function useForm(initialValues, onSubmit) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
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

  return { values, errors, isSubmitting, handleChange, handleSubmit };
}

// Use in any form
function LoginForm({ onSubmit }) {
  const form = useForm({ email: '', password: '' }, onSubmit);

  return (
    <form onSubmit={form.handleSubmit}>
      <input
        name="email"
        value={form.values.email}
        onChange={form.handleChange}
      />
      <button type="submit" disabled={form.isSubmitting}>
        Log In
      </button>
    </form>
  );
}
```

---

## Compound Components: Flexible APIs

Compound components are a set of components designed to work together while giving users control over structure:

```javascript
// ✅ Compound Accordion
const AccordionContext = createContext();

export function Accordion({ children, defaultValue }) {
  const [value, setValue] = useState(defaultValue);

  return (
    <AccordionContext.Provider value={{ value, setValue }}>
      <div>{children}</div>
    </AccordionContext.Provider>
  );
}

export function AccordionItem({ value, children }) {
  // Just wraps children, doesn't impose structure
  return <div>{children}</div>;
}

export function AccordionTrigger({ value, children }) {
  const { value: activeValue, setValue } = useContext(AccordionContext);

  return (
    <button
      onClick={() => setValue(value)}
      aria-expanded={activeValue === value}
    >
      {children}
    </button>
  );
}

export function AccordionContent({ value, children }) {
  const { value: activeValue } = useContext(AccordionContext);

  return activeValue === value ? <div>{children}</div> : null;
}

// Users can structure it however they want:
<Accordion defaultValue="section1">
  <AccordionItem value="section1">
    <AccordionTrigger value="section1">
      Section 1
    </AccordionTrigger>
    <AccordionContent value="section1">
      Content 1
    </AccordionContent>
  </AccordionItem>

  <AccordionItem value="section2">
    <AccordionTrigger value="section2">
      Section 2
    </AccordionTrigger>
    <AccordionContent value="section2">
      Content 2
    </AccordionContent>
  </AccordionItem>
</Accordion>

// Or with custom styling:
<Accordion defaultValue="section1">
  {sections.map(section => (
    <div key={section.id} className="custom-item">
      <AccordionTrigger value={section.id}>
        <h3>{section.title}</h3>
      </AccordionTrigger>
      <AccordionContent value={section.id}>
        <article>{section.content}</article>
      </AccordionContent>
    </div>
  ))}
</Accordion>
```

**Testing compound components:**

```javascript
test('user can open accordion sections', async ({ page }) => {
  const trigger1 = page.getByRole('button', { name: /section 1/i });
  const trigger2 = page.getByRole('button', { name: /section 2/i });

  // Initially section 1 is open
  await expect(trigger1).toHaveAttribute('aria-expanded', 'true');
  await expect(trigger2).toHaveAttribute('aria-expanded', 'false');

  // Click section 2
  await trigger2.click();

  // Section 1 closes, section 2 opens
  await expect(trigger1).toHaveAttribute('aria-expanded', 'false');
  await expect(trigger2).toHaveAttribute('aria-expanded', 'true');
});
```

---

## Avoiding Prop Drilling

### The Problem

```javascript
// ❌ Props drill through many layers
<App settings={settings}>
  <Layout settings={settings}>
    <Content settings={settings}>
      <Component settings={settings} />
    </Content>
  </Layout>
</App>
```

### Solutions

**1. Context**

```javascript
const SettingsContext = createContext();

function App({ settings }) {
  return (
    <SettingsContext.Provider value={settings}>
      <Layout>
        <Content>
          <Component />
        </Content>
      </Layout>
    </SettingsContext.Provider>
  );
}

function Component() {
  const settings = useContext(SettingsContext);
  return <div>{settings.theme}</div>;
}
```

**2. Composition (Better)**

```javascript
// Don't drill, compose instead
function App({ settings }) {
  const styled = applySettings(settings);

  return (
    <Layout className={styled}>
      <Content className={styled}>
        <Component className={styled} />
      </Content>
    </Layout>
  );
}
```

**3. Children (Best for Structure)**

```javascript
// Pass structure as children, not configuration
function Layout({ children }) {
  return (
    <div className="layout">
      {children}
    </div>
  );
}

function App({ settings }) {
  return (
    <Layout>
      <Content>
        <Component settings={settings} />
      </Content>
    </Layout>
  );
}
```

---

## Performance Optimization (When Needed)

> Only optimize after measuring actual performance issues.

### useMemo: Prevent Expensive Recalculations

```javascript
// ❌ Recalculates every render even if items haven't changed
function ProductList({ items }) {
  // This calculation runs on every render
  const sorted = [...items].sort((a, b) => a.name.localeCompare(b.name));

  return (
    <ul>
      {sorted.map(item => <ProductCard key={item.id} item={item} />)}
    </ul>
  );
}

// ✅ Only recalculate when items changes
function ProductList({ items }) {
  const sorted = useMemo(
    () => [...items].sort((a, b) => a.name.localeCompare(b.name)),
    [items]
  );

  return (
    <ul>
      {sorted.map(item => <ProductCard key={item.id} item={item} />)}
    </ul>
  );
}
```

### useCallback: Prevent Unnecessary Re-renders

```javascript
// ❌ New callback function every render
function Parent() {
  const handleClick = () => {
    doSomething();
  };

  // Child re-renders even if items haven't changed
  return <Child items={items} onItemClick={handleClick} />;
}

// ✅ Same callback unless dependencies change
function Parent() {
  const handleClick = useCallback(() => {
    doSomething();
  }, []); // Empty deps - callback never changes

  return <Child items={items} onItemClick={handleClick} />;
}
```

### React.memo: Prevent Child Re-renders

```javascript
// ❌ Child re-renders even if props haven't changed
function ProductCard({ item }) {
  return <div>{item.name}</div>;
}

// ✅ Only re-render if props change
const ProductCard = React.memo(function ProductCard({ item }) {
  return <div>{item.name}</div>;
});
```

---

## Summary: Good Component Architecture

- **Single Responsibility** - Each component has one reason to change
- **Composition** - Combine simple components together
- **Props for Direct Relationships** - Pass data down naturally
- **Context for Distant Relationships** - Avoid prop drilling
- **Custom Hooks** - Extract reusable logic
- **Compound Components** - Flexible APIs that users can structure
- **Optimize Only When Needed** - Measure first, optimize second
- **Test Components Independently** - Each component is testable in isolation

---

## Next Steps

- See **SKILL.md** for complete implementation patterns
- See **EXAMPLES.md** for 30+ production-ready components
- See **ANTI-PATTERNS.md** for what not to do
