# React Component Composition Patterns

A deep dive into strategies for building flexible, reusable components through composition.

## Props Down, Callbacks Up

The fundamental React data flow pattern.

**The Pattern:**
- Data flows from parent to child via props
- Events and state changes flow from child to parent via callbacks
- Child components don't modify parent state directly

**Example: Real-World Todo Item**

```javascript
// Parent manages the list and handles updates
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: "Learn React", completed: false },
    { id: 2, text: "Build app", completed: false },
  ]);

  const handleToggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  const handleDeleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  const handleAddTodo = (text) => {
    setTodos([...todos, {
      id: Math.max(...todos.map(t => t.id), 0) + 1,
      text,
      completed: false
    }]);
  };

  return (
    <>
      <TodoInput onAddTodo={handleAddTodo} />
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={handleToggleTodo}
          onDelete={handleDeleteTodo}
        />
      ))}
    </>
  );
}

// Input component - simple, controlled
function TodoInput({ onAddTodo }) {
  const [value, setValue] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault();
    if (value.trim()) {
      onAddTodo(value);
      setValue("");
    }
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

// Item component - receives everything as props
const TodoItem = React.memo(({ todo, onToggle, onDelete }) => {
  return (
    <div className="todo-item">
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </div>
  );
});
```

**Benefits:**
- Data flow is clear and predictable
- Components are reusable
- Testing is straightforward
- Debugging is easier

**When to use:**
- For most component hierarchies
- When components are closely related
- When the chain is 2-3 levels deep

## Component Composition Over Configuration

Build flexible components through composition instead of configuration props.

**Example: Card Component**

```javascript
// ❌ Over-configured - hard to use, inflexible
function BadCard({ 
  title, 
  subtitle, 
  image, 
  imagePosition, 
  body, 
  footer, 
  onFooterClick, 
  footerVariant,
  ... // 20+ props
}) {
  // Impossible to handle all combinations
}

// ✅ Composition approach - flexible and simple
function Card({ children }) {
  return <div className="card">{children}</div>;
}

function CardHeader({ children }) {
  return <div className="card-header">{children}</div>;
}

function CardBody({ children }) {
  return <div className="card-body">{children}</div>;
}

function CardImage({ src, alt }) {
  return <img className="card-image" src={src} alt={alt} />;
}

function CardFooter({ children }) {
  return <div className="card-footer">{children}</div>;
}

// Usage - much cleaner
<Card>
  <CardImage src="image.jpg" alt="Product" />
  <CardHeader>
    <h2>Product Name</h2>
    <p>Subtitle</p>
  </CardHeader>
  <CardBody>
    <p>Product description goes here</p>
  </CardBody>
  <CardFooter>
    <button>Add to Cart</button>
    <button>Save for Later</button>
  </CardFooter>
</Card>
```

**Benefits:**
- Unlimited flexibility
- Simple, clear component interface
- Easy to test
- Fewer props to manage

**When to use:**
- Building UI components
- When content varies significantly
- When you want maximum flexibility

## Compound Components

Components designed to work together as a family, managing internal state and communication.

**Example: Tabs Component**

```javascript
// The parent component manages state
function Tabs({ children }) {
  const [activeTab, setActiveTab] = useState(0);

  return (
    <div className="tabs">
      {React.Children.map(children, (child, index) => {
        if (child.type === TabButton) {
          return React.cloneElement(child, {
            isActive: index === activeTab,
            onClick: () => setActiveTab(index),
          });
        }
        return child;
      })}
      {React.Children.toArray(children).find((_, i) => i === activeTab)?.type === TabPanel
        ? React.Children.toArray(children)[activeTab]
        : null}
    </div>
  );
}

function TabButton({ children, isActive, onClick }) {
  return (
    <button
      className={`tab-button ${isActive ? 'active' : ''}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

function TabPanel({ children }) {
  return <div className="tab-panel">{children}</div>;
}

// Usage
<Tabs>
  <TabButton>Tab 1</TabButton>
  <TabButton>Tab 2</TabButton>
  <TabButton>Tab 3</TabButton>
  <TabPanel>Content for tab 1</TabPanel>
  <TabPanel>Content for tab 2</TabPanel>
  <TabPanel>Content for tab 3</TabPanel>
</Tabs>
```

**Example: Select Component**

```javascript
const SelectContext = createContext();

export function Select({ children, value, onChange }) {
  const [isOpen, setIsOpen] = useState(false);

  const selectContextValue = useMemo(() => ({
    isOpen,
    setIsOpen,
    value,
    onChange,
  }), [isOpen, value, onChange]);

  return (
    <SelectContext.Provider value={selectContextValue}>
      <div className="select">
        {children}
      </div>
    </SelectContext.Provider>
  );
}

export function SelectButton({ children }) {
  const { isOpen, setIsOpen, value } = useContext(SelectContext);

  return (
    <button
      className="select-button"
      onClick={() => setIsOpen(!isOpen)}
    >
      {children || value || "Select..."}
    </button>
  );
}

export function SelectOptions({ children }) {
  const { isOpen } = useContext(SelectContext);

  if (!isOpen) return null;

  return (
    <div className="select-options">
      {children}
    </div>
  );
}

export function SelectOption({ value, children }) {
  const { onChange, setIsOpen } = useContext(SelectContext);

  const handleClick = () => {
    onChange(value);
    setIsOpen(false);
  };

  return (
    <button className="select-option" onClick={handleClick}>
      {children}
    </button>
  );
}

// Usage
<Select value={selected} onChange={setSelected}>
  <SelectButton />
  <SelectOptions>
    <SelectOption value="apple">Apple</SelectOption>
    <SelectOption value="banana">Banana</SelectOption>
    <SelectOption value="orange">Orange</SelectOption>
  </SelectOptions>
</Select>
```

**Benefits:**
- Implicit communication between related components
- Clean, intuitive API
- Flexible internal state management
- Can be extended without changing parent

**When to use:**
- Multi-part components (Tabs, Select, Modal)
- Components with internal communication
- Component families

## Render Props Pattern

Pass a function as a prop to control what and how something renders.

**Example: Data Provider**

```javascript
function DataProvider({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(error => {
        setError(error);
        setLoading(false);
      });
  }, [url]);

  return render({ data, loading, error });
}

// Usage with render prop
<DataProvider
  url="/api/users"
  render={({ data, loading, error }) => {
    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error.message}</div>;
    return <UserList users={data} />;
  }}
/>

// Alternative: children as function (more common in modern React)
function DataProvider({ url, children }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(error => {
        setError(error);
        setLoading(false);
      });
  }, [url]);

  return children({ data, loading, error });
}

// Usage with children as function
<DataProvider url="/api/users">
  {({ data, loading, error }) => {
    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error.message}</div>;
    return <UserList users={data} />;
  }}
</DataProvider>
```

**Example: Mouse Tracker**

```javascript
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return render(position);
}

// Usage
<MouseTracker
  render={({ x, y }) => (
    <div style={{ position: 'absolute', left: x, top: y }}>
      Cursor is at ({x}, {y})
    </div>
  )}
/>
```

**Benefits:**
- Maximum flexibility in rendering
- Can share logic without inheritance
- Easy to test logic separately from rendering

**When to use:**
- When logic and rendering need to vary independently
- Sharing complex logic between components
- Advanced patterns requiring full control

**Note:** Hooks (custom hooks) have largely replaced render props, but both patterns are still useful in different scenarios.

## Higher-Order Components

A function that takes a component and returns a new component with added functionality.

**Example: With Theme**

```javascript
function withTheme(Component) {
  return function ThemedComponent(props) {
    const theme = useTheme();
    return <Component {...props} theme={theme} />;
  };
}

// Usage
function MyComponent({ theme }) {
  return <div className={theme}>Content</div>;
}

export default withTheme(MyComponent);
```

**Example: With Authentication**

```javascript
function withAuthRequired(Component) {
  return function ProtectedComponent(props) {
    const { user, loading } = useAuth();

    if (loading) return <div>Loading...</div>;
    if (!user) return <Navigate to="/login" />;

    return <Component {...props} user={user} />;
  };
}

// Usage
const ProtectedPage = withAuthRequired(Dashboard);
```

**When to use:**
- Cross-cutting concerns (auth, logging, theme)
- Need to manipulate props before passing to component
- Need to render something before the component

**Caveats:**
- Prefer custom hooks when possible
- Can make component tree harder to debug
- Can interfere with statics and refs

**Hooks are generally better**, but HOCs are still useful for certain patterns.

## Passing Components as Props

Components can receive other components as props for maximum flexibility.

**Example: Layout**

```javascript
function DashboardLayout({ 
  Header, 
  Sidebar, 
  MainContent, 
  Footer 
}) {
  return (
    <div className="dashboard">
      <Header />
      <div className="dashboard-body">
        <Sidebar />
        <main><MainContent /></main>
      </div>
      <Footer />
    </div>
  );
}

// Usage
<DashboardLayout
  Header={AppHeader}
  Sidebar={NavSidebar}
  MainContent={Dashboard}
  Footer={AppFooter}
/>

// Or with inline components
<DashboardLayout
  Header={() => <h1>Dashboard</h1>}
  Sidebar={Navigation}
  MainContent={MainDashboard}
  Footer={() => <footer>© 2024</footer>}
/>
```

**Example: List with Custom Item**

```javascript
function List({ items, ItemComponent, onItemClick }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick?.(item)}>
          <ItemComponent item={item} />
        </li>
      ))}
    </ul>
  );
}

// Usage with different item components
function UserItem({ item }) {
  return <div>{item.name} ({item.email})</div>;
}

function ProductItem({ item }) {
  return <div>{item.name} - ${item.price}</div>;
}

<List items={users} ItemComponent={UserItem} onItemClick={handleSelect} />
<List items={products} ItemComponent={ProductItem} onItemClick={handleSelect} />
```

**Benefits:**
- Maximum flexibility
- Component reusability
- Clean separation of concerns
- Easy to extend

**When to use:**
- Building layout systems
- Custom rendering in lists
- Plugin-like architectures

## Choosing Between Patterns

**Decision Tree:**

```
Does the component wrap other content?
├─ YES → Use composition (children)
│
Does the component family have internal communication?
├─ YES → Use compound components
│
Does the component provide logic that rendering depends on?
├─ YES → Use custom hooks or render props
│  ├─ Modern apps → Custom hooks (preferred)
│  └─ Shared library → Render props
│
Do you need to enhance/wrap a component?
├─ YES → Use HOC or custom hook
│  ├─ Need to manipulate props/rendering → HOC
│  └─ Extracting logic → Custom hook (preferred)
│
Do you need maximum flexibility?
└─ YES → Pass components as props
```

**Quick Comparison:**

| Pattern | Use Case | Flexibility | Complexity |
|---------|----------|------------|-----------|
| Composition | Content wrapping | Medium | Low |
| Compound Components | Related component families | High | Medium |
| Render Props | Logic sharing with rendering | Very High | Medium-High |
| Custom Hooks | Logic sharing | High | Low |
| HOC | Component enhancement | Medium | High |
| Component Props | Layout/structure | Very High | Low |

---

## See Also

- [SKILL.md](./SKILL.md) - Component fundamentals and best practices
- [EXAMPLES.md](./EXAMPLES.md) - Working examples of these patterns
- [STATE-MANAGEMENT-GUIDE.md](./STATE-MANAGEMENT-GUIDE.md) - Where to manage state in composed components
