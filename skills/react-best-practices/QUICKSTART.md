# React Best Practices - Quick Start Templates

Copy-paste templates for common React patterns. All templates are fully functional and follow best practices.

## 1. Simple Component with State

```javascript
import { useState } from 'react';

/**
 * A simple counter component with state management
 * 
 * Key concepts:
 * - useState for managing component state
 * - setState function for updates
 * - Re-render on state change
 */
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div className="counter">
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <button onClick={() => setCount(count - 1)}>
        Decrement
      </button>
      <button onClick={() => setCount(0)}>
        Reset
      </button>
    </div>
  );
}

export default Counter;
```

## 2. Component with Side Effects

```javascript
import { useState, useEffect } from 'react';

/**
 * Fetches data from an API and displays it
 * 
 * Key concepts:
 * - useEffect for side effects
 * - Dependency array to control when effect runs
 * - Cleanup function to prevent memory leaks
 * - Loading and error states
 */
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Fetch data when component mounts
    let isMounted = true;

    const fetchUsers = async () => {
      try {
        const response = await fetch('https://api.example.com/users');
        if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
        const data = await response.json();
        
        if (isMounted) {
          setUsers(data);
        }
      } catch (error) {
        if (isMounted) {
          setError(error.message);
        }
      } finally {
        if (isMounted) {
          setLoading(false);
        }
      }
    };

    fetchUsers();

    // Cleanup: mark component as unmounted
    return () => {
      isMounted = false;
    };
  }, []); // Empty dependency array: run only once on mount

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

export default UserList;
```

## 3. Form Component (Controlled)

```javascript
import { useState } from 'react';

/**
 * A form with controlled components and validation
 * 
 * Key concepts:
 * - Controlled inputs (value from state)
 * - Form submission handling
 * - Input validation
 * - Error messages
 */
function ContactForm() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    message: '',
  });

  const [errors, setErrors] = useState({});
  const [submitted, setSubmitted] = useState(false);

  // Validate form fields
  const validate = () => {
    const newErrors = {};

    if (!form.name.trim()) {
      newErrors.name = 'Name is required';
    }

    if (!form.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email)) {
      newErrors.email = 'Email is invalid';
    }

    if (!form.message.trim()) {
      newErrors.message = 'Message is required';
    }

    return newErrors;
  };

  // Handle input changes
  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
    // Clear error for this field when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  // Handle form submission
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const newErrors = validate();
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

    // Form is valid, submit it
    console.log('Submitting:', form);
    setSubmitted(true);

    // Reset form after 2 seconds
    setTimeout(() => {
      setForm({ name: '', email: '', message: '' });
      setSubmitted(false);
    }, 2000);
  };

  if (submitted) {
    return <div className="success">✓ Thank you! Your message has been sent.</div>;
  }

  return (
    <form onSubmit={handleSubmit} className="form">
      <div className="form-group">
        <label htmlFor="name">Name</label>
        <input
          id="name"
          name="name"
          value={form.name}
          onChange={handleChange}
          className={errors.name ? 'error' : ''}
          placeholder="Your name"
        />
        {errors.name && <span className="error-message">{errors.name}</span>}
      </div>

      <div className="form-group">
        <label htmlFor="email">Email</label>
        <input
          id="email"
          name="email"
          type="email"
          value={form.email}
          onChange={handleChange}
          className={errors.email ? 'error' : ''}
          placeholder="your@email.com"
        />
        {errors.email && <span className="error-message">{errors.email}</span>}
      </div>

      <div className="form-group">
        <label htmlFor="message">Message</label>
        <textarea
          id="message"
          name="message"
          value={form.message}
          onChange={handleChange}
          className={errors.message ? 'error' : ''}
          placeholder="Your message..."
          rows="5"
        />
        {errors.message && <span className="error-message">{errors.message}</span>}
      </div>

      <button type="submit">Send</button>
    </form>
  );
}

export default ContactForm;
```

## 4. List with Filtering and Sorting

```javascript
import { useState, useMemo } from 'react';

/**
 * A list with filtering and sorting capabilities
 * 
 * Key concepts:
 * - useMemo for expensive computations
 * - Derived state instead of storing filtered results
 * - Proper list keys for React reconciliation
 */
function ProductList() {
  const [products] = useState([
    { id: 1, name: 'Laptop', category: 'Electronics', price: 1200 },
    { id: 2, name: 'Mouse', category: 'Electronics', price: 30 },
    { id: 3, name: 'Desk', category: 'Furniture', price: 300 },
    { id: 4, name: 'Chair', category: 'Furniture', price: 150 },
  ]);

  const [filter, setFilter] = useState('');
  const [sortBy, setSortBy] = useState('name');

  // Memoize filtered and sorted products
  const filteredAndSorted = useMemo(() => {
    return products
      .filter(product =>
        product.name.toLowerCase().includes(filter.toLowerCase()) ||
        product.category.toLowerCase().includes(filter.toLowerCase())
      )
      .sort((a, b) => {
        if (sortBy === 'name') {
          return a.name.localeCompare(b.name);
        } else if (sortBy === 'price') {
          return a.price - b.price;
        }
        return 0;
      });
  }, [products, filter, sortBy]);

  return (
    <div className="product-list">
      <div className="controls">
        <input
          type="text"
          placeholder="Search products..."
          value={filter}
          onChange={(e) => setFilter(e.target.value)}
          className="search-input"
        />

        <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
          <option value="name">Sort by Name</option>
          <option value="price">Sort by Price</option>
        </select>
      </div>

      <ul className="products">
        {filteredAndSorted.map(product => (
          <li key={product.id} className="product-item">
            <span className="name">{product.name}</span>
            <span className="category">{product.category}</span>
            <span className="price">${product.price}</span>
          </li>
        ))}
      </ul>

      {filteredAndSorted.length === 0 && (
        <div className="no-results">No products found</div>
      )}
    </div>
  );
}

export default ProductList;
```

## 5. Custom Hook

```javascript
import { useState, useCallback, useEffect } from 'react';

/**
 * Custom hook: useLocalStorage
 * Syncs state with browser localStorage
 * 
 * Key concepts:
 * - Custom hooks for reusable logic
 * - Hook rules: top level, in components/hooks
 * - useCallback for memoized setter
 */
function useLocalStorage(key, initialValue) {
  // Initialize from localStorage
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  // Memoized setter that also updates localStorage
  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue];
}

// Example usage
function UserPreferences() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [sidebarOpen, setSidebarOpen] = useLocalStorage('sidebarOpen', true);

  return (
    <div>
      <select value={theme} onChange={(e) => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>

      <label>
        <input
          type="checkbox"
          checked={sidebarOpen}
          onChange={(e) => setSidebarOpen(e.target.checked)}
        />
        Sidebar Open
      </label>
    </div>
  );
}

export default UserPreferences;
```

## 6. Context Setup

```javascript
import { createContext, useContext, useState, useMemo } from 'react';

/**
 * Theme context for app-wide theme management
 * 
 * Key concepts:
 * - createContext for global state
 * - Custom hook for accessing context
 * - useMemo to prevent unnecessary re-renders
 * - Separation of concerns
 */

// Create context
const ThemeContext = createContext();

// Provider component
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => (prev === 'light' ? 'dark' : 'light'));
  };

  // Memoize value to prevent unnecessary re-renders
  const value = useMemo(
    () => ({ theme, toggleTheme }),
    [theme]
  );

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook for using theme
export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used inside ThemeProvider');
  }
  return context;
}

// Usage in App
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}

// Usage in component
function Header() {
  const { theme, toggleTheme } = useTheme();

  return (
    <header className={theme}>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}

export default App;
```

## 7. Async Data Fetching

```javascript
import { useState, useEffect } from 'react';

/**
 * Async data fetching with proper cleanup
 * 
 * Key concepts:
 * - AbortController for canceling requests
 * - Cleanup on unmount
 * - Loading, success, and error states
 */
function SearchUsers({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Don't fetch if query is empty
    if (!query) {
      setResults([]);
      return;
    }

    // Create AbortController to cancel requests
    const controller = new AbortController();

    setLoading(true);
    setError(null);

    // Fetch data
    fetch(`https://api.example.com/users?search=${query}`, {
      signal: controller.signal,
    })
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then(data => {
        setResults(data);
        setLoading(false);
      })
      .catch(err => {
        // AbortError is expected on cleanup
        if (err.name !== 'AbortError') {
          setError(err.message);
          setLoading(false);
        }
      });

    // Cleanup: abort the request if component unmounts
    return () => controller.abort();
  }, [query]); // Re-fetch when query changes

  if (loading) return <div>Searching...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {results.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

export default SearchUsers;
```

## 8. Compound Component Pattern

```javascript
import { useState, createContext, useContext } from 'react';

/**
 * Compound component pattern for Tabs
 * 
 * Key concepts:
 * - Related components work together
 * - Internal state management via context
 * - Clean, intuitive API
 */

// Context for Tab communication
const TabsContext = createContext();

function Tabs({ children }) {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }) {
  return <div className="tab-list">{children}</div>;
}

function TabButton({ index, children }) {
  const { activeTab, setActiveTab } = useContext(TabsContext);

  return (
    <button
      className={`tab-button ${activeTab === index ? 'active' : ''}`}
      onClick={() => setActiveTab(index)}
    >
      {children}
    </button>
  );
}

function TabContent({ index, children }) {
  const { activeTab } = useContext(TabsContext);

  if (activeTab !== index) return null;

  return <div className="tab-content">{children}</div>;
}

// Export compound component
Tabs.List = TabList;
Tabs.Button = TabButton;
Tabs.Content = TabContent;

// Usage
function TabsExample() {
  return (
    <Tabs>
      <Tabs.List>
        <Tabs.Button index={0}>Tab 1</Tabs.Button>
        <Tabs.Button index={1}>Tab 2</Tabs.Button>
        <Tabs.Button index={2}>Tab 3</Tabs.Button>
      </Tabs.List>

      <Tabs.Content index={0}>
        <p>Content for tab 1</p>
      </Tabs.Content>
      <Tabs.Content index={1}>
        <p>Content for tab 2</p>
      </Tabs.Content>
      <Tabs.Content index={2}>
        <p>Content for tab 3</p>
      </Tabs.Content>
    </Tabs>
  );
}

export default TabsExample;
```

---

## Tips for Using These Templates

1. **Read the comments** - Each template includes comments explaining the key concepts
2. **Adapt to your needs** - These are starting points, modify them for your use case
3. **Follow the patterns** - Use these as examples for consistent code style
4. **Test thoroughly** - Add tests using the React Testable Code skill
5. **Performance** - Profile before optimizing; these templates use sensible defaults

---

## See Also

- [SKILL.md](./SKILL.md) - Detailed explanations of all concepts
- [EXAMPLES.md](./EXAMPLES.md) - More advanced examples
- [ANTI-PATTERNS.md](./ANTI-PATTERNS.md) - What NOT to do
