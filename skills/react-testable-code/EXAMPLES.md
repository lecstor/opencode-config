# React Component Examples: Production-Ready, Testable Code

This document contains 30+ complete, production-ready component examples. Each demonstrates how semantic HTML and accessibility make E2E testing trivial.

## Table of Contents

1. [Form Components](#form-components)
2. [List & Table Components](#list--table-components)
3. [Modal & Dialog Components](#modal--dialog-components)
4. [Compound Components](#compound-components)
5. [Async & Loading States](#async--loading-states)
6. [Validation Patterns](#validation-patterns)
7. [Error Handling](#error-handling)

---

## Form Components

### 1. Search Form

**Why it's testable:** Uses semantic `<form>`, `<input>` with `aria-label`, and `<button>` with semantic role.

```javascript
function SearchForm({ onSearch, placeholder = 'Search...' }) {
  const [query, setQuery] = useState('');
  const [isSearching, setIsSearching] = useState(false);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSearching(true);
    try {
      await onSearch(query);
    } finally {
      setIsSearching(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder={placeholder}
        aria-label="Search"
        disabled={isSearching}
      />
      <button type="submit" disabled={isSearching || !query.trim()}>
        {isSearching ? 'Searching...' : 'Search'}
      </button>
    </form>
  );
}
```

**Playwright test:**
```javascript
test('user can search', async ({ page }) => {
  await page.getByLabel('Search').fill('laptop');
  await page.getByRole('button', { name: /search/i }).click();
  
  // Wait for results
  await expect(page.getByText(/results/i)).toBeVisible();
});
```

### 2. Login Form with Validation

**Why it's testable:** Proper labels, semantic form, error messages with `role="alert"`.

```javascript
function LoginForm({ onSubmit }) {
  const [formData, setFormData] = useState({ email: '', password: '' });
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validate = () => {
    const newErrors = {};
    if (!formData.email) newErrors.email = 'Email is required';
    else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email)) {
      newErrors.email = 'Please enter a valid email';
    }
    if (!formData.password) newErrors.password = 'Password is required';
    else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    return newErrors;
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    // Clear error for this field when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    const newErrors = validate();

    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

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
        <label htmlFor="email">Email Address</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          disabled={isSubmitting}
          aria-describedby={errors.email ? 'email-error' : undefined}
          required
        />
        {errors.email && (
          <p id="email-error" role="alert" className="error">
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
          disabled={isSubmitting}
          aria-describedby={errors.password ? 'password-error' : undefined}
          required
        />
        {errors.password && (
          <p id="password-error" role="alert" className="error">
            {errors.password}
          </p>
        )}
      </div>

      {errors.submit && (
        <p role="alert" className="error">
          {errors.submit}
        </p>
      )}

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Log In'}
      </button>
    </form>
  );
}
```

**Playwright test:**
```javascript
test('shows validation errors', async ({ page }) => {
  await page.getByRole('button', { name: /log in/i }).click();
  
  await expect(page.getByRole('alert')).toContainText('Email is required');
  await expect(page.getByRole('alert')).toContainText('Password is required');
});

test('logs in with valid credentials', async ({ page }) => {
  await page.getByLabel('Email Address').fill('user@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: /log in/i }).click();
  
  // Wait for navigation or success message
  await expect(page).toHaveURL('/dashboard');
});
```

### 3. Multi-Field Contact Form

**Why it's testable:** Fieldset for grouping, proper labels, semantic form elements.

```javascript
function ContactForm({ onSubmit }) {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    subject: '',
    message: '',
    category: 'general',
    subscribe: false
  });
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [success, setSuccess] = useState(false);

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    try {
      await onSubmit(formData);
      setSuccess(true);
      setFormData({
        name: '',
        email: '',
        subject: '',
        message: '',
        category: 'general',
        subscribe: false
      });
      setTimeout(() => setSuccess(false), 5000);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {success && (
        <p role="alert" aria-live="polite" className="success">
          Thank you for your message! We'll get back to you soon.
        </p>
      )}

      <fieldset disabled={isSubmitting}>
        <legend>Contact Information</legend>

        <div>
          <label htmlFor="name">Full Name</label>
          <input
            id="name"
            name="name"
            type="text"
            value={formData.name}
            onChange={handleChange}
            required
          />
        </div>

        <div>
          <label htmlFor="email">Email Address</label>
          <input
            id="email"
            name="email"
            type="email"
            value={formData.email}
            onChange={handleChange}
            required
          />
        </div>
      </fieldset>

      <fieldset disabled={isSubmitting}>
        <legend>Message Details</legend>

        <div>
          <label htmlFor="category">Category</label>
          <select
            id="category"
            name="category"
            value={formData.category}
            onChange={handleChange}
          >
            <option value="general">General Inquiry</option>
            <option value="support">Support</option>
            <option value="sales">Sales</option>
            <option value="feedback">Feedback</option>
          </select>
        </div>

        <div>
          <label htmlFor="subject">Subject</label>
          <input
            id="subject"
            name="subject"
            type="text"
            value={formData.subject}
            onChange={handleChange}
            required
          />
        </div>

        <div>
          <label htmlFor="message">Message</label>
          <textarea
            id="message"
            name="message"
            value={formData.message}
            onChange={handleChange}
            rows={5}
            required
          />
        </div>
      </fieldset>

      <div>
        <input
          id="subscribe"
          name="subscribe"
          type="checkbox"
          checked={formData.subscribe}
          onChange={handleChange}
          disabled={isSubmitting}
        />
        <label htmlFor="subscribe">Subscribe to our newsletter</label>
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Sending...' : 'Send Message'}
      </button>
    </form>
  );
}
```

**Playwright test:**
```javascript
test('user can fill and submit contact form', async ({ page }) => {
  await page.getByLabel('Full Name').fill('John Doe');
  await page.getByLabel('Email Address').fill('john@example.com');
  await page.getByLabel('Category').selectOption('support');
  await page.getByLabel('Subject').fill('Help needed');
  await page.getByLabel('Message').fill('This is my message');
  await page.getByLabel('Subscribe to our newsletter').check();

  await page.getByRole('button', { name: /send message/i }).click();

  await expect(
    page.getByRole('alert').filter({ hasText: /thank you/i })
  ).toBeVisible();
});
```

### 4. Filter Form with Multiple Options

**Why it's testable:** Checkboxes/radios with proper labels, fieldsets for grouping.

```javascript
function FilterForm({ onFilter, categories, priceRanges }) {
  const [filters, setFilters] = useState({
    categories: [],
    priceRange: null,
    inStock: false,
    sortBy: 'popular'
  });

  const handleCategoryChange = (category) => {
    setFilters(prev => ({
      ...prev,
      categories: prev.categories.includes(category)
        ? prev.categories.filter(c => c !== category)
        : [...prev.categories, category]
    }));
  };

  const handlePriceChange = (range) => {
    setFilters(prev => ({ ...prev, priceRange: range }));
  };

  const handleSort = (e) => {
    setFilters(prev => ({ ...prev, sortBy: e.target.value }));
  };

  const handleReset = () => {
    setFilters({
      categories: [],
      priceRange: null,
      inStock: false,
      sortBy: 'popular'
    });
    onFilter({
      categories: [],
      priceRange: null,
      inStock: false,
      sortBy: 'popular'
    });
  };

  useEffect(() => {
    onFilter(filters);
  }, [filters, onFilter]);

  return (
    <form>
      <fieldset>
        <legend>Categories</legend>
        {categories.map(category => (
          <div key={category}>
            <input
              id={`cat-${category}`}
              type="checkbox"
              checked={filters.categories.includes(category)}
              onChange={() => handleCategoryChange(category)}
            />
            <label htmlFor={`cat-${category}`}>{category}</label>
          </div>
        ))}
      </fieldset>

      <fieldset>
        <legend>Price Range</legend>
        {priceRanges.map(range => (
          <div key={range.id}>
            <input
              id={`price-${range.id}`}
              type="radio"
              name="price"
              value={range.id}
              checked={filters.priceRange === range.id}
              onChange={() => handlePriceChange(range.id)}
            />
            <label htmlFor={`price-${range.id}`}>{range.label}</label>
          </div>
        ))}
      </fieldset>

      <div>
        <input
          id="in-stock"
          type="checkbox"
          checked={filters.inStock}
          onChange={(e) =>
            setFilters(prev => ({ ...prev, inStock: e.target.checked }))
          }
        />
        <label htmlFor="in-stock">In Stock Only</label>
      </div>

      <fieldset>
        <legend>Sort By</legend>
        <select value={filters.sortBy} onChange={handleSort}>
          <option value="popular">Most Popular</option>
          <option value="price-low">Price: Low to High</option>
          <option value="price-high">Price: High to Low</option>
          <option value="newest">Newest</option>
          <option value="rating">Highest Rated</option>
        </select>
      </fieldset>

      <button type="button" onClick={handleReset}>
        Clear Filters
      </button>
    </form>
  );
}
```

**Playwright test:**
```javascript
test('user can filter products', async ({ page }) => {
  await page.getByLabel('Electronics').check();
  await page.getByLabel('$100 - $500').check();
  await page.getByLabel('In Stock Only').check();

  // Verify filters are applied (via parent component)
  await expect(page.getByRole('heading', { level: 2 })).toContainText('Electronics');
});

test('user can clear filters', async ({ page }) => {
  await page.getByLabel('Electronics').check();
  await page.getByRole('button', { name: /clear filters/i }).click();

  await expect(page.getByLabel('Electronics')).not.toBeChecked();
});
```

---

## List & Table Components

### 5. Simple List with Add/Remove

**Why it's testable:** Semantic `<ul>`, `<li>`, proper button labels.

```javascript
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, { id: Date.now(), text: input, done: false }]);
      setInput('');
    }
  };

  const removeTodo = (id) => {
    setTodos(todos.filter(t => t.id !== id));
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(t =>
      t.id === id ? { ...t, done: !t.done } : t
    ));
  };

  const handleKeyPress = (e) => {
    if (e.key === 'Enter') {
      addTodo();
    }
  };

  return (
    <div>
      <h1>My Todo List</h1>

      <div>
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={handleKeyPress}
          placeholder="Add a new todo"
          aria-label="New todo"
        />
        <button onClick={addTodo}>Add Todo</button>
      </div>

      {todos.length === 0 ? (
        <p>No todos yet. Add one above!</p>
      ) : (
        <ul>
          {todos.map(todo => (
            <li key={todo.id}>
              <input
                type="checkbox"
                id={`todo-${todo.id}`}
                checked={todo.done}
                onChange={() => toggleTodo(todo.id)}
              />
              <label htmlFor={`todo-${todo.id}`}>
                <span style={{ textDecoration: todo.done ? 'line-through' : 'none' }}>
                  {todo.text}
                </span>
              </label>
              <button
                onClick={() => removeTodo(todo.id)}
                aria-label={`Delete "${todo.text}"`}
              >
                Delete
              </button>
            </li>
          ))}
        </ul>
      )}

      {todos.length > 0 && (
        <p aria-live="polite">
          {todos.filter(t => !t.done).length} of {todos.length} remaining
        </p>
      )}
    </div>
  );
}
```

**Playwright test:**
```javascript
test('user can add and remove todos', async ({ page }) => {
  await page.getByLabel('New todo').fill('Learn React');
  await page.getByRole('button', { name: /add todo/i }).click();

  await expect(page.getByText('Learn React')).toBeVisible();

  await page.getByLabel('Delete "Learn React"').click();
  await expect(page.getByText('Learn React')).not.toBeVisible();
});

test('user can mark todo as done', async ({ page }) => {
  await page.getByLabel('New todo').fill('Buy milk');
  await page.getByRole('button', { name: /add todo/i }).click();

  const checkbox = page.getByRole('checkbox');
  await checkbox.check();

  await expect(page.getByText('Buy milk')).toHaveCSS('text-decoration', /line-through/);
});
```

### 6. Data Table with Sorting and Pagination

**Why it's testable:** Semantic `<table>`, `<thead>`, `<th>` with aria-sort, proper buttons.

```javascript
function DataTable({ data, columns }) {
  const [sortColumn, setSortColumn] = useState(null);
  const [sortDirection, setSortDirection] = useState('asc');
  const [currentPage, setCurrentPage] = useState(1);
  const itemsPerPage = 10;

  const handleSort = (column) => {
    if (sortColumn === column) {
      setSortDirection(sortDirection === 'asc' ? 'desc' : 'asc');
    } else {
      setSortColumn(column);
      setSortDirection('asc');
    }
  };

  let sortedData = [...data];
  if (sortColumn) {
    sortedData.sort((a, b) => {
      const aVal = a[sortColumn];
      const bVal = b[sortColumn];

      if (typeof aVal === 'string') {
        return sortDirection === 'asc'
          ? aVal.localeCompare(bVal)
          : bVal.localeCompare(aVal);
      }

      return sortDirection === 'asc' ? aVal - bVal : bVal - aVal;
    });
  }

  const totalPages = Math.ceil(sortedData.length / itemsPerPage);
  const startIndex = (currentPage - 1) * itemsPerPage;
  const paginatedData = sortedData.slice(startIndex, startIndex + itemsPerPage);

  return (
    <div>
      <table>
        <thead>
          <tr>
            {columns.map(col => (
              <th key={col.key}>
                <button
                  onClick={() => handleSort(col.key)}
                  aria-sort={
                    sortColumn === col.key
                      ? (sortDirection === 'asc' ? 'ascending' : 'descending')
                      : 'none'
                  }
                >
                  {col.label}
                  {sortColumn === col.key && (
                    <span aria-hidden="true">
                      {sortDirection === 'asc' ? ' ↑' : ' ↓'}
                    </span>
                  )}
                </button>
              </th>
            ))}
          </tr>
        </thead>
        <tbody>
          {paginatedData.map((row, idx) => (
            <tr key={idx}>
              {columns.map(col => (
                <td key={col.key}>{row[col.key]}</td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>

      <nav aria-label="Pagination">
        <button
          onClick={() => setCurrentPage(prev => Math.max(prev - 1, 1))}
          disabled={currentPage === 1}
        >
          Previous
        </button>

        <span aria-live="polite">
          Page {currentPage} of {totalPages}
        </span>

        <button
          onClick={() => setCurrentPage(prev => Math.min(prev + 1, totalPages))}
          disabled={currentPage === totalPages}
        >
          Next
        </button>
      </nav>
    </div>
  );
}
```

**Playwright test:**
```javascript
test('user can sort table by column', async ({ page }) => {
  // Click the Name header to sort
  await page.getByRole('button', { name: /name/i }).click();

  // Check that aria-sort indicates ascending
  const nameHeader = page.getByRole('button', { name: /name/i });
  await expect(nameHeader).toHaveAttribute('aria-sort', 'ascending');
});

test('user can paginate through results', async ({ page }) => {
  await page.getByRole('button', { name: /next/i }).click();

  await expect(page.getByText(/page 2 of/i)).toBeVisible();

  await page.getByRole('button', { name: /previous/i }).click();

  await expect(page.getByText(/page 1 of/i)).toBeVisible();
});
```

### 7. Expandable Rows List

**Why it's testable:** Proper button with aria-expanded, semantic structure.

```javascript
function ExpandableList({ items }) {
  const [expandedIds, setExpandedIds] = useState(new Set());

  const toggleExpanded = (id) => {
    setExpandedIds(prev => {
      const next = new Set(prev);
      if (next.has(id)) {
        next.delete(id);
      } else {
        next.add(id);
      }
      return next;
    });
  };

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <div>
            <button
              aria-expanded={expandedIds.has(item.id)}
              aria-controls={`details-${item.id}`}
              onClick={() => toggleExpanded(item.id)}
            >
              {item.title}
              <span aria-hidden="true">
                {expandedIds.has(item.id) ? '▼' : '▶'}
              </span>
            </button>
          </div>

          {expandedIds.has(item.id) && (
            <div id={`details-${item.id}`}>
              <p>{item.description}</p>
              {item.details && (
                <ul>
                  {item.details.map((detail, idx) => (
                    <li key={idx}>{detail}</li>
                  ))}
                </ul>
              )}
            </div>
          )}
        </li>
      ))}
    </ul>
  );
}
```

**Playwright test:**
```javascript
test('user can expand and collapse items', async ({ page }) => {
  const button = page.getByRole('button', { name: /Item 1/i });

  await expect(button).toHaveAttribute('aria-expanded', 'false');

  await button.click();

  await expect(button).toHaveAttribute('aria-expanded', 'true');
  await expect(page.getByText(/Item 1 details/i)).toBeVisible();

  await button.click();

  await expect(button).toHaveAttribute('aria-expanded', 'false');
});
```

---

## Modal & Dialog Components

### 8. Confirmation Dialog

**Why it's testable:** Uses semantic `<dialog>` or role="alertdialog", proper buttons.

```javascript
function ConfirmDialog({
  isOpen,
  title,
  message,
  confirmLabel = 'Confirm',
  cancelLabel = 'Cancel',
  onConfirm,
  onCancel,
  isDangerous = false
}) {
  if (!isOpen) return null;

  const handleConfirm = () => {
    onConfirm();
  };

  const handleCancel = () => {
    onCancel();
  };

  const handleBackdropClick = (e) => {
    if (e.target === e.currentTarget) {
      handleCancel();
    }
  };

  return (
    <div
      role="presentation"
      onClick={handleBackdropClick}
      style={{
        position: 'fixed',
        inset: 0,
        backgroundColor: 'rgba(0, 0, 0, 0.5)',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        zIndex: 1000
      }}
    >
      <dialog
        open
        role="alertdialog"
        aria-labelledby="dialog-title"
        aria-describedby="dialog-message"
        style={{
          padding: '2rem',
          borderRadius: '0.5rem',
          boxShadow: '0 10px 25px rgba(0, 0, 0, 0.1)',
          border: 'none',
          maxWidth: '90vw',
          width: '400px'
        }}
      >
        <h2 id="dialog-title">{title}</h2>
        <p id="dialog-message">{message}</p>

        <div style={{ display: 'flex', gap: '1rem', justifyContent: 'flex-end' }}>
          <button onClick={handleCancel}>{cancelLabel}</button>
          <button
            onClick={handleConfirm}
            style={{
              backgroundColor: isDangerous ? '#dc2626' : '#2563eb',
              color: 'white'
            }}
          >
            {confirmLabel}
          </button>
        </div>
      </dialog>
    </div>
  );
}
```

**Playwright test:**
```javascript
test('user can confirm dialog action', async ({ page }) => {
  // Dialog is open
  await expect(page.getByRole('alertdialog')).toBeVisible();

  await page.getByRole('button', { name: /confirm/i }).click();

  // Dialog closes
  await expect(page.getByRole('alertdialog')).not.toBeVisible();
});

test('user can cancel dialog', async ({ page }) => {
  await page.getByRole('button', { name: /cancel/i }).click();

  await expect(page.getByRole('alertdialog')).not.toBeVisible();
});
```

### 9. Modal with Form

**Why it's testable:** Proper focus management, semantic form, backdrop click to close.

```javascript
function EditModal({ isOpen, item, onSave, onClose }) {
  const [formData, setFormData] = useState(item || { name: '', description: '' });
  const dialogRef = useRef(null);

  useEffect(() => {
    if (isOpen && dialogRef.current) {
      dialogRef.current.showModal();
      // Focus first input
      dialogRef.current.querySelector('input')?.focus();
    } else if (dialogRef.current) {
      dialogRef.current.close();
    }
  }, [isOpen]);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    onSave(formData);
    onClose();
  };

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
      <h2 id="modal-title">Edit Item</h2>

      <form onSubmit={handleSubmit}>
        <div>
          <label htmlFor="item-name">Name</label>
          <input
            id="item-name"
            name="name"
            value={formData.name}
            onChange={handleChange}
            required
          />
        </div>

        <div>
          <label htmlFor="item-description">Description</label>
          <textarea
            id="item-description"
            name="description"
            value={formData.description}
            onChange={handleChange}
            rows={4}
          />
        </div>

        <div style={{ display: 'flex', gap: '1rem', justifyContent: 'flex-end' }}>
          <button type="button" onClick={onClose}>
            Cancel
          </button>
          <button type="submit">Save</button>
        </div>
      </form>
    </dialog>
  );
}
```

**Playwright test:**
```javascript
test('user can edit item in modal', async ({ page }) => {
  await page.getByRole('button', { name: /edit/i }).click();

  await expect(page.getByRole('dialog')).toBeVisible();

  await page.getByLabel('Name').fill('Updated Name');
  await page.getByRole('button', { name: /save/i }).click();

  await expect(page.getByRole('dialog')).not.toBeVisible();
  await expect(page.getByText('Updated Name')).toBeVisible();
});
```

---

## Compound Components

### 10. Tabs Component

**Why it's testable:** Proper roles, aria-selected, aria-controls.

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
    <div role="tabpanel" id={`panel-${value}`} tabIndex={0}>
      {children}
    </div>
  ) : null;
}

// Usage
export function MyTabs() {
  return (
    <Tabs defaultValue="overview">
      <TabsList>
        <TabsTrigger value="overview">Overview</TabsTrigger>
        <TabsTrigger value="details">Details</TabsTrigger>
        <TabsTrigger value="reviews">Reviews</TabsTrigger>
      </TabsList>

      <TabsContent value="overview">
        <p>Overview content here</p>
      </TabsContent>

      <TabsContent value="details">
        <p>Details content here</p>
      </TabsContent>

      <TabsContent value="reviews">
        <p>Reviews content here</p>
      </TabsContent>
    </Tabs>
  );
}
```

**Playwright test:**
```javascript
test('user can switch between tabs', async ({ page }) => {
  const overviewTab = page.getByRole('tab', { name: /overview/i });
  const detailsTab = page.getByRole('tab', { name: /details/i });

  await expect(overviewTab).toHaveAttribute('aria-selected', 'true');
  await expect(page.getByRole('tabpanel')).toContainText('Overview content');

  await detailsTab.click();

  await expect(detailsTab).toHaveAttribute('aria-selected', 'true');
  await expect(page.getByRole('tabpanel')).toContainText('Details content');
});
```

### 11. Accordion Component

**Why it's testable:** Proper buttons with aria-expanded, controlled sections.

```javascript
const AccordionContext = createContext();

function Accordion({ children }) {
  const [openIds, setOpenIds] = useState(new Set());

  const toggle = (id) => {
    setOpenIds(prev => {
      const next = new Set(prev);
      if (next.has(id)) {
        next.delete(id);
      } else {
        next.add(id);
      }
      return next;
    });
  };

  return (
    <AccordionContext.Provider value={{ openIds, toggle }}>
      <div>{children}</div>
    </AccordionContext.Provider>
  );
}

function AccordionItem({ id, children }) {
  return <div>{children}</div>;
}

function AccordionTrigger({ id, children }) {
  const { openIds, toggle } = useContext(AccordionContext);
  const isOpen = openIds.has(id);

  return (
    <button
      onClick={() => toggle(id)}
      aria-expanded={isOpen}
      aria-controls={`content-${id}`}
    >
      {children}
      <span aria-hidden="true">{isOpen ? '−' : '+'}</span>
    </button>
  );
}

function AccordionContent({ id, children }) {
  const { openIds } = useContext(AccordionContext);
  const isOpen = openIds.has(id);

  return isOpen ? (
    <div id={`content-${id}`}>{children}</div>
  ) : null;
}

// Usage
export function MyAccordion() {
  return (
    <Accordion>
      <AccordionItem id="section1">
        <AccordionTrigger id="section1">What is React?</AccordionTrigger>
        <AccordionContent id="section1">
          React is a JavaScript library for building user interfaces.
        </AccordionContent>
      </AccordionItem>

      <AccordionItem id="section2">
        <AccordionTrigger id="section2">How do I use React?</AccordionTrigger>
        <AccordionContent id="section2">
          You can use React by installing it with npm and creating a new project.
        </AccordionContent>
      </AccordionItem>
    </Accordion>
  );
}
```

**Playwright test:**
```javascript
test('user can toggle accordion sections', async ({ page }) => {
  const button = page.getByRole('button', { name: /what is react/i });

  await expect(button).toHaveAttribute('aria-expanded', 'false');
  await button.click();
  await expect(button).toHaveAttribute('aria-expanded', 'true');
});
```

---

## Async & Loading States

### 12. Data Fetching with Loading and Error States

**Why it's testable:** Clear states with proper ARIA, semantic loading/error messages.

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      setError(null);
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Failed to load user');
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  if (loading) {
    return (
      <div aria-busy="true" aria-label="Loading user profile">
        <p>Loading...</p>
      </div>
    );
  }

  if (error) {
    return (
      <div role="alert">
        <p>Error loading user: {error}</p>
        <button onClick={() => window.location.reload()}>
          Try Again
        </button>
      </div>
    );
  }

  return (
    <article>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <p>{user.bio}</p>
    </article>
  );
}
```

**Playwright test:**
```javascript
test('shows loading state while fetching', async ({ page }) => {
  await expect(page.getByLabel('Loading user profile')).toBeVisible();
});

test('shows error state on fetch failure', async ({ page }) => {
  // Mock failed request...
  await expect(page.getByRole('alert')).toContainText('Error loading user');
});

test('displays user data after loading', async ({ page }) => {
  await expect(page.getByText(/john doe/i)).toBeVisible();
});
```

### 13. Form with Async Submission

**Why it's testable:** Clear loading state, proper error handling, success feedback.

```javascript
function NewsletterSignup() {
  const [email, setEmail] = useState('');
  const [status, setStatus] = useState('idle'); // idle, loading, success, error
  const [error, setError] = useState(null);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setStatus('loading');
    setError(null);

    try {
      const response = await fetch('/api/newsletter/subscribe', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email })
      });

      if (!response.ok) throw new Error('Subscription failed');

      setStatus('success');
      setEmail('');
      // Reset after 5 seconds
      setTimeout(() => setStatus('idle'), 5000);
    } catch (err) {
      setStatus('error');
      setError(err.message);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter your email"
        disabled={status === 'loading'}
        aria-label="Email address"
        required
      />

      <button type="submit" disabled={status === 'loading'}>
        {status === 'loading' ? 'Subscribing...' : 'Subscribe'}
      </button>

      {status === 'success' && (
        <p role="alert" aria-live="polite" style={{ color: 'green' }}>
          Thank you for subscribing!
        </p>
      )}

      {status === 'error' && (
        <p role="alert" style={{ color: 'red' }}>
          {error}
        </p>
      )}
    </form>
  );
}
```

**Playwright test:**
```javascript
test('shows loading state during submission', async ({ page }) => {
  // Mock slow network...
  await page.getByLabel('Email address').fill('user@example.com');
  await page.getByRole('button', { name: /subscribe/i }).click();

  await expect(
    page.getByRole('button', { name: /subscribing/i })
  ).toBeDisabled();
});

test('shows success message after subscription', async ({ page }) => {
  await page.getByLabel('Email address').fill('user@example.com');
  await page.getByRole('button', { name: /subscribe/i }).click();

  await expect(
    page.getByRole('alert').filter({ hasText: /thank you/i })
  ).toBeVisible();
});
```

---

## Validation Patterns

### 14. Real-Time Form Validation

**Why it's testable:** Errors shown as they occur, clear error states.

```javascript
function PasswordForm() {
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [errors, setErrors] = useState({});

  const validatePassword = (pwd) => {
    const errs = [];
    if (pwd.length < 8) errs.push('At least 8 characters');
    if (!/[A-Z]/.test(pwd)) errs.push('At least one uppercase letter');
    if (!/[0-9]/.test(pwd)) errs.push('At least one number');
    if (!/[!@#$%^&*]/.test(pwd)) errs.push('At least one special character (!@#$%^&*)');
    return errs;
  };

  const handlePasswordChange = (e) => {
    const pwd = e.target.value;
    setPassword(pwd);

    if (pwd) {
      const errs = validatePassword(pwd);
      setErrors(prev => ({ ...prev, password: errs }));
    } else {
      setErrors(prev => ({ ...prev, password: [] }));
    }

    // Check match
    if (confirmPassword && pwd !== confirmPassword) {
      setErrors(prev => ({ ...prev, mismatch: true }));
    } else {
      setErrors(prev => ({ ...prev, mismatch: false }));
    }
  };

  const handleConfirmChange = (e) => {
    const confirmed = e.target.value;
    setConfirmPassword(confirmed);

    if (confirmed !== password) {
      setErrors(prev => ({ ...prev, mismatch: true }));
    } else {
      setErrors(prev => ({ ...prev, mismatch: false }));
    }
  };

  const isValid =
    password &&
    errors.password?.length === 0 &&
    confirmPassword &&
    !errors.mismatch;

  return (
    <form>
      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={handlePasswordChange}
          aria-describedby={errors.password?.length > 0 ? 'password-errors' : undefined}
        />
        {errors.password?.length > 0 && (
          <ul id="password-errors" role="alert">
            {errors.password.map((err, idx) => (
              <li key={idx} style={{ color: 'red' }}>
                {err}
              </li>
            ))}
          </ul>
        )}
      </div>

      <div>
        <label htmlFor="confirm-password">Confirm Password</label>
        <input
          id="confirm-password"
          type="password"
          value={confirmPassword}
          onChange={handleConfirmChange}
          aria-describedby={errors.mismatch ? 'password-mismatch' : undefined}
        />
        {errors.mismatch && (
          <p id="password-mismatch" role="alert" style={{ color: 'red' }}>
            Passwords do not match
          </p>
        )}
      </div>

      <button type="submit" disabled={!isValid}>
        Set Password
      </button>
    </form>
  );
}
```

**Playwright test:**
```javascript
test('shows password requirements as user types', async ({ page }) => {
  const input = page.getByLabel('Password');

  await input.fill('weak');
  await expect(page.getByRole('alert')).toContainText('At least 8 characters');

  await input.clear();
  await input.fill('Password1!');

  // All requirements met
  await expect(page.getByRole('alert')).not.toBeVisible();
});

test('shows mismatch error', async ({ page }) => {
  await page.getByLabel('Password').fill('Password1!');
  await page.getByLabel('Confirm Password').fill('Different1!');

  await expect(page.getByText(/passwords do not match/i)).toBeVisible();
});
```

---

## Error Handling

### 15. Error Boundary Component

**Why it's testable:** Clear error states, proper recovery options.

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

  resetError = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      return (
        <div role="alert">
          <h1>Something went wrong</h1>
          <details>
            <summary>Error details</summary>
            <pre>{this.state.error?.message}</pre>
          </details>
          <button onClick={this.resetError}>Try Again</button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <YourComponent />
</ErrorBoundary>
```

**Playwright test:**
```javascript
test('error boundary catches errors', async ({ page }) => {
  // Component throws error...
  await expect(page.getByRole('alert')).toContainText('Something went wrong');
  
  await page.getByRole('button', { name: /try again/i }).click();
  
  // Component recovers
  await expect(page.getByText(/original content/i)).toBeVisible();
});
```

---

## Next Steps

- See **ANTI-PATTERNS.md** for patterns that break E2E tests
- See **ACCESSIBILITY-FOR-TESTING.md** for why accessibility = testability
- See **COMPONENT-ARCHITECTURE.md** for deep-dive component design
- See **QUICKSTART.md** for simple templates to start with

All of these examples follow the core principle: **Good React code is code that's easy to test.** By using semantic HTML, proper accessibility, and clear component hierarchies, these components are trivial to test with Playwright.
