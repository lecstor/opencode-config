# React Best Practices - Production-Ready Examples

This document contains complete, runnable examples demonstrating React best practices.

## Component Hierarchy Design

### Example 1: Todo Application with Proper Component Hierarchy

```javascript
// Structure: App → TodoContainer → (TodoInput, TodoList → TodoItem)

import React, { useState } from 'react';

// Parent component that manages state
function TodoContainer() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState("");

  const addTodo = (text) => {
    if (text.trim()) {
      setTodos([...todos, {
        id: Date.now(),
        text,
        completed: false
      }]);
    }
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  return (
    <>
      <TodoInput onAdd={addTodo} />
      <TodoList 
        todos={todos}
        onToggle={toggleTodo}
        onDelete={deleteTodo}
      />
    </>
  );
}

// Input component - simple, controlled
function TodoInput({ onAdd }) {
  const [value, setValue] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault();
    onAdd(value);
    setValue("");
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Add a todo..."
      />
      <button type="submit">Add</button>
    </form>
  );
}

// List component - pure presentation
function TodoList({ todos, onToggle, onDelete }) {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={onToggle}
          onDelete={onDelete}
        />
      ))}
    </ul>
  );
}

// Item component - minimal, reusable
const TodoItem = React.memo(({ todo, onToggle, onDelete }) => (
  <li>
    <input
      type="checkbox"
      checked={todo.completed}
      onChange={() => onToggle(todo.id)}
    />
    <span style={{
      textDecoration: todo.completed ? 'line-through' : 'none'
    }}>
      {todo.text}
    </span>
    <button onClick={() => onDelete(todo.id)}>Delete</button>
  </li>
));

export default TodoContainer;
```

### Example 2: E-commerce Product View

```javascript
// Hierarchy: ProductPage → (ProductImage, ProductInfo → (Rating, Price, AddToCart))

function ProductPage({ productId }) {
  const [product, setProduct] = useState(null);
  const [quantity, setQuantity] = useState(1);

  useEffect(() => {
    fetch(`/api/products/${productId}`)
      .then(res => res.json())
      .then(setProduct);
  }, [productId]);

  if (!product) return <div>Loading...</div>;

  const handleAddToCart = () => {
    // Add to cart logic
    console.log(`Adding ${quantity} of product ${product.id}`);
  };

  return (
    <div className="product-page">
      <ProductImage image={product.image} alt={product.name} />
      <ProductInfo
        product={product}
        quantity={quantity}
        onQuantityChange={setQuantity}
        onAddToCart={handleAddToCart}
      />
    </div>
  );
}

function ProductImage({ image, alt }) {
  return <img src={image} alt={alt} className="product-image" />;
}

function ProductInfo({ product, quantity, onQuantityChange, onAddToCart }) {
  return (
    <div className="product-info">
      <h1>{product.name}</h1>
      <Rating rating={product.rating} reviews={product.reviewCount} />
      <Price
        original={product.originalPrice}
        current={product.price}
      />
      <QuantitySelector
        quantity={quantity}
        onChange={onQuantityChange}
        maxStock={product.stock}
      />
      <button onClick={onAddToCart} className="btn-primary">
        Add to Cart
      </button>
    </div>
  );
}

function Rating({ rating, reviews }) {
  return (
    <div className="rating">
      <span>⭐ {rating}/5</span>
      <span>({reviews} reviews)</span>
    </div>
  );
}

function Price({ original, current }) {
  const discount = Math.round(((original - current) / original) * 100);
  
  return (
    <div className="price">
      <span className="current">${current}</span>
      {original > current && (
        <>
          <span className="original">${original}</span>
          <span className="discount">-{discount}%</span>
        </>
      )}
    </div>
  );
}

function QuantitySelector({ quantity, onChange, maxStock }) {
  return (
    <div className="quantity">
      <button
        onClick={() => onChange(Math.max(1, quantity - 1))}
        disabled={quantity <= 1}
      >
        -
      </button>
      <input type="number" value={quantity} onChange={(e) => onChange(parseInt(e.target.value) || 1)} />
      <button
        onClick={() => onChange(Math.min(maxStock, quantity + 1))}
        disabled={quantity >= maxStock}
      >
        +
      </button>
    </div>
  );
}
```

## State Placement Decisions

### Example 1: State Belongs in Container Component

```javascript
// ✅ Correct: Timer state in one place
function StopwatchContainer() {
  const [time, setTime] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning) return;

    const interval = setInterval(() => {
      setTime(t => t + 1);
    }, 1000);

    return () => clearInterval(interval);
  }, [isRunning]);

  return (
    <>
      <TimeDisplay time={time} />
      <Controls
        isRunning={isRunning}
        onStart={() => setIsRunning(true)}
        onStop={() => setIsRunning(false)}
        onReset={() => { setTime(0); setIsRunning(false); }}
      />
    </>
  );
}

function TimeDisplay({ time }) {
  const minutes = Math.floor(time / 60);
  const seconds = time % 60;
  return <div>{minutes}:{seconds.toString().padStart(2, '0')}</div>;
}

function Controls({ isRunning, onStart, onStop, onReset }) {
  return (
    <>
      <button onClick={isRunning ? onStop : onStart}>
        {isRunning ? 'Stop' : 'Start'}
      </button>
      <button onClick={onReset}>Reset</button>
    </>
  );
}
```

### Example 2: Component-Specific State

```javascript
// ✅ Correct: Local state for UI only
const AccordionItem = React.memo(({ title, content }) => {
  const [isOpen, setIsOpen] = useState(false);
  // This state is local to this item, not needed elsewhere

  return (
    <div className="accordion-item">
      <button onClick={() => setIsOpen(!isOpen)}>
        {title}
      </button>
      {isOpen && <div className="content">{content}</div>}
    </div>
  );
});

// ✅ Correct: Modal state in its own component
function Modal({ isOpen, onClose, children }) {
  // Modal manages its own open/close state
  return isOpen ? (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  ) : null;
}
```

## Props Down, Callbacks Up Pattern

### Example 1: Multi-Level Communication

```javascript
// The pattern: Parent holds state, passes data down, receives updates up

function ParentWithList() {
  const [selectedId, setSelectedId] = useState(null);
  const [items, setItems] = useState([
    { id: 1, name: "Item 1" },
    { id: 2, name: "Item 2" },
  ]);

  const handleSelectItem = (id) => {
    setSelectedId(id);
  };

  const handleDeleteItem = (id) => {
    setItems(items.filter(item => item.id !== id));
    if (selectedId === id) setSelectedId(null);
  };

  const selectedItem = items.find(item => item.id === selectedId);

  return (
    <div className="container">
      <ItemList
        items={items}
        selectedId={selectedId}
        onSelectItem={handleSelectItem}
        onDeleteItem={handleDeleteItem}
      />
      {selectedItem && <ItemDetail item={selectedItem} />}
    </div>
  );
}

function ItemList({ items, selectedId, onSelectItem, onDeleteItem }) {
  return (
    <div className="list">
      {items.map(item => (
        <ItemRow
          key={item.id}
          item={item}
          isSelected={item.id === selectedId}
          onSelect={() => onSelectItem(item.id)}
          onDelete={() => onDeleteItem(item.id)}
        />
      ))}
    </div>
  );
}

function ItemRow({ item, isSelected, onSelect, onDelete }) {
  return (
    <div
      className={`item-row ${isSelected ? 'selected' : ''}`}
      onClick={onSelect}
    >
      <span>{item.name}</span>
      <button onClick={(e) => {
        e.stopPropagation();
        onDelete();
      }}>
        Delete
      </button>
    </div>
  );
}

function ItemDetail({ item }) {
  return (
    <div className="detail">
      <h2>{item.name}</h2>
      <p>Details for {item.name}</p>
    </div>
  );
}
```

## useEffect Patterns

### Example 1: Fetching Data on Mount

```javascript
function UsersList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        const response = await fetch('/api/users');
        if (!response.ok) throw new Error('Failed to fetch');
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []); // Empty dependency - runs once on mount

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
```

### Example 2: Reacting to Prop Changes

```javascript
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (!query) {
      setResults([]);
      return;
    }

    setLoading(true);
    const controller = new AbortController();

    fetch(`/api/search?q=${query}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => {
        setResults(data);
        setLoading(false);
      })
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error(err);
        }
      });

    return () => controller.abort(); // Cancel request on unmount or query change
  }, [query]); // Re-run when query changes

  return (
    <div>
      {loading && <div>Searching...</div>}
      {results.map(result => (
        <div key={result.id}>{result.title}</div>
      ))}
    </div>
  );
}
```

### Example 3: Cleanup - Timer

```javascript
function Timer({ duration }) {
  const [timeLeft, setTimeLeft] = useState(duration);

  useEffect(() => {
    if (timeLeft <= 0) return;

    const timer = setInterval(() => {
      setTimeLeft(prev => prev - 1);
    }, 1000);

    return () => clearInterval(timer); // Cleanup: clear timer
  }, [timeLeft]);

  return <div>Time left: {timeLeft}s</div>;
}
```

### Example 4: Event Listener with Cleanup

```javascript
function ResponsiveGrid() {
  const [columns, setColumns] = useState(1);

  useEffect(() => {
    const handleResize = () => {
      const width = window.innerWidth;
      setColumns(width > 1200 ? 4 : width > 768 ? 2 : 1);
    };

    handleResize(); // Call once on mount
    window.addEventListener('resize', handleResize);

    return () => window.removeEventListener('resize', handleResize); // Cleanup
  }, []);

  return (
    <div style={{ display: 'grid', gridTemplateColumns: `repeat(${columns}, 1fr)` }}>
      {/* Grid content */}
    </div>
  );
}
```

## useReducer for Complex State

### Example 1: Form State with Validation

```javascript
const formInitialState = {
  fields: {
    email: '',
    password: '',
    confirmPassword: '',
  },
  errors: {},
  isSubmitting: false,
  isSubmitted: false,
};

function formReducer(state, action) {
  switch (action.type) {
    case 'FIELD_CHANGE': {
      const { name, value } = action.payload;
      return {
        ...state,
        fields: { ...state.fields, [name]: value },
        errors: { ...state.errors, [name]: '' }, // Clear error for this field
      };
    }

    case 'FIELD_BLUR': {
      const { name } = action.payload;
      const field = state.fields[name];
      const error = validateField(name, field);
      return {
        ...state,
        errors: { ...state.errors, [name]: error },
      };
    }

    case 'VALIDATE': {
      const newErrors = {};
      Object.keys(state.fields).forEach(field => {
        const error = validateField(field, state.fields[field]);
        if (error) newErrors[field] = error;
      });
      return { ...state, errors: newErrors };
    }

    case 'SUBMIT_START':
      return { ...state, isSubmitting: true };

    case 'SUBMIT_SUCCESS':
      return { ...state, isSubmitting: false, isSubmitted: true };

    case 'SUBMIT_ERROR': {
      const { errors } = action.payload;
      return { ...state, isSubmitting: false, errors };
    }

    case 'RESET':
      return formInitialState;

    default:
      return state;
  }
}

function validateField(name, value) {
  if (!value) return `${name} is required`;
  if (name === 'email' && !value.includes('@')) return 'Invalid email';
  if (name === 'password' && value.length < 8) return 'Password must be at least 8 characters';
  if (name === 'confirmPassword' && value !== formState.fields.password) return 'Passwords do not match';
  return '';
}

function AdvancedForm() {
  const [formState, dispatch] = useReducer(formReducer, formInitialState);

  const handleChange = (e) => {
    const { name, value } = e.target;
    dispatch({ type: 'FIELD_CHANGE', payload: { name, value } });
  };

  const handleBlur = (e) => {
    const { name } = e.target;
    dispatch({ type: 'FIELD_BLUR', payload: { name } });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    dispatch({ type: 'VALIDATE' });

    const hasErrors = Object.values(formState.errors).some(err => err);
    if (hasErrors) return;

    dispatch({ type: 'SUBMIT_START' });

    try {
      await fetch('/api/register', {
        method: 'POST',
        body: JSON.stringify(formState.fields),
      });
      dispatch({ type: 'SUBMIT_SUCCESS' });
    } catch (error) {
      dispatch({ type: 'SUBMIT_ERROR', payload: { errors: { form: 'Submission failed' } } });
    }
  };

  const { fields, errors, isSubmitting, isSubmitted } = formState;

  if (isSubmitted) return <div>✓ Form submitted successfully!</div>;

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="email"
          value={fields.email}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>

      <div>
        <input
          name="password"
          type="password"
          value={fields.password}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Password"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Sign Up'}
      </button>
    </form>
  );
}
```

## Custom Hooks

### Example 1: useLocalStorage

```javascript
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

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

// Usage
function UserPreferences() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [sidebarOpen, setSidebarOpen] = useLocalStorage('sidebarOpen', true);

  return (
    <>
      <select value={theme} onChange={(e) => setTheme(e.target.value)}>
        <option>light</option>
        <option>dark</option>
      </select>
      <label>
        <input
          type="checkbox"
          checked={sidebarOpen}
          onChange={(e) => setSidebarOpen(e.target.checked)}
        />
        Sidebar Open
      </label>
    </>
  );
}
```

### Example 2: useFetch

```javascript
function useFetch(url, options = {}) {
  const [state, dispatch] = useReducer(
    (state, action) => {
      switch (action.type) {
        case 'LOADING':
          return { ...state, status: 'loading' };
        case 'SUCCESS':
          return { status: 'success', data: action.payload, error: null };
        case 'ERROR':
          return { status: 'error', data: null, error: action.payload };
        default:
          return state;
      }
    },
    { status: 'idle', data: null, error: null }
  );

  const [refetch, setRefetch] = useState(0);

  useEffect(() => {
    if (!url) return;

    const controller = new AbortController();
    dispatch({ type: 'LOADING' });

    fetch(url, { ...options, signal: controller.signal })
      .then(res => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then(data => dispatch({ type: 'SUCCESS', payload: data }))
      .catch(error => {
        if (error.name !== 'AbortError') {
          dispatch({ type: 'ERROR', payload: error });
        }
      });

    return () => controller.abort();
  }, [url, options, refetch]);

  return { ...state, refetch: () => setRefetch(r => r + 1) };
}

// Usage
function DataComponent() {
  const { status, data, error, refetch } = useFetch('/api/data');

  if (status === 'loading') return <div>Loading...</div>;
  if (status === 'error') return <div>Error: {error.message}</div>;

  return (
    <>
      <button onClick={refetch}>Refresh</button>
      <div>{data}</div>
    </>
  );
}
```

## Context API Usage

### Example 1: Theme Context

```javascript
const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState(() => {
    const saved = localStorage.getItem('theme');
    return saved || 'light';
  });

  const toggleTheme = useCallback(() => {
    setTheme(prev => {
      const newTheme = prev === 'light' ? 'dark' : 'light';
      localStorage.setItem('theme', newTheme);
      return newTheme;
    });
  }, []);

  const value = useMemo(() => ({ theme, toggleTheme }), [theme, toggleTheme]);

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used inside ThemeProvider');
  }
  return context;
}

// Usage
function App() {
  return (
    <ThemeProvider>
      <MainApp />
    </ThemeProvider>
  );
}

function MainApp() {
  const { theme, toggleTheme } = useTheme();
  return (
    <div className={theme}>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </div>
  );
}
```

## Forms

### Example 1: Controlled Form with Validation

```javascript
function ContactForm() {
  const [form, setForm] = useState({
    name: '',
    email: '',
    message: '',
  });

  const [errors, setErrors] = useState({});
  const [submitted, setSubmitted] = useState(false);

  const validateForm = () => {
    const newErrors = {};

    if (!form.name.trim()) newErrors.name = 'Name is required';
    if (!form.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email)) {
      newErrors.email = 'Invalid email format';
    }
    if (!form.message.trim()) newErrors.message = 'Message is required';

    return newErrors;
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    const newErrors = validateForm();

    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

    console.log('Submitting:', form);
    setSubmitted(true);
    setForm({ name: '', email: '', message: '' });

    setTimeout(() => setSubmitted(false), 3000);
  };

  if (submitted) {
    return <div className="success">✓ Thank you! Your message has been sent.</div>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Name</label>
        <input
          name="name"
          value={form.name}
          onChange={handleChange}
          className={errors.name ? 'error' : ''}
        />
        {errors.name && <span className="error-text">{errors.name}</span>}
      </div>

      <div>
        <label>Email</label>
        <input
          name="email"
          type="email"
          value={form.email}
          onChange={handleChange}
          className={errors.email ? 'error' : ''}
        />
        {errors.email && <span className="error-text">{errors.email}</span>}
      </div>

      <div>
        <label>Message</label>
        <textarea
          name="message"
          value={form.message}
          onChange={handleChange}
          className={errors.message ? 'error' : ''}
        />
        {errors.message && <span className="error-text">{errors.message}</span>}
      </div>

      <button type="submit">Send</button>
    </form>
  );
}
```

## Lists with Keys and Filtering

### Example 1: Proper List Rendering with Filtering

```javascript
function FilterableProductList({ products }) {
  const [filter, setFilter] = useState('');
  const [sortBy, setSortBy] = useState('name');

  const filtered = useMemo(() => {
    return products
      .filter(p => p.name.toLowerCase().includes(filter.toLowerCase()))
      .sort((a, b) => {
        if (sortBy === 'name') return a.name.localeCompare(b.name);
        if (sortBy === 'price') return a.price - b.price;
        return 0;
      });
  }, [products, filter, sortBy]);

  return (
    <>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Search products..."
      />

      <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
        <option value="name">Name</option>
        <option value="price">Price</option>
      </select>

      <ul>
        {filtered.map(product => (
          <ProductListItem key={product.id} product={product} />
        ))}
      </ul>
    </>
  );
}

const ProductListItem = React.memo(({ product }) => {
  return (
    <li>
      <span>{product.name}</span>
      <span>${product.price}</span>
    </li>
  );
});
```

## Error Boundaries

### Example 1: Simple Error Boundary

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // Log to error reporting service
    console.error('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-container">
          <h1>Something went wrong</h1>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

## Data Fetching

### Example 1: Complex Data Fetching with Cache

```javascript
function useDataFetching(url, options = {}) {
  const cacheRef = useRef(new Map());
  const [state, dispatch] = useReducer(
    (state, action) => {
      switch (action.type) {
        case 'FETCH_START':
          return { ...state, loading: true, error: null };
        case 'FETCH_SUCCESS':
          return { loading: false, data: action.payload, error: null };
        case 'FETCH_ERROR':
          return { loading: false, data: null, error: action.payload };
        default:
          return state;
      }
    },
    { loading: false, data: null, error: null }
  );

  const fetch = useCallback(async (fetchUrl = url) => {
    if (cacheRef.current.has(fetchUrl)) {
      dispatch({ type: 'FETCH_SUCCESS', payload: cacheRef.current.get(fetchUrl) });
      return;
    }

    dispatch({ type: 'FETCH_START' });

    try {
      const response = await window.fetch(fetchUrl, options);
      if (!response.ok) throw new Error('Failed to fetch');
      const data = await response.json();
      cacheRef.current.set(fetchUrl, data);
      dispatch({ type: 'FETCH_SUCCESS', payload: data });
    } catch (error) {
      dispatch({ type: 'FETCH_ERROR', payload: error });
    }
  }, [url, options]);

  useEffect(() => {
    fetch();
  }, [fetch]);

  return { ...state, refetch: fetch };
}

// Usage
function UserList({ userId }) {
  const { loading, data, error, refetch } = useDataFetching(`/api/users/${userId}`);

  return (
    <>
      {loading && <div>Loading...</div>}
      {error && (
        <div>
          Error: {error.message}
          <button onClick={refetch}>Retry</button>
        </div>
      )}
      {data && <UserInfo user={data} />}
    </>
  );
}
```

---

## See Also

- [SKILL.md](./SKILL.md) - Core React best practices reference
- [ANTI-PATTERNS.md](./ANTI-PATTERNS.md) - Common mistakes and solutions
- [COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md) - Advanced composition strategies
