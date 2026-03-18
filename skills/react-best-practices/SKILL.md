# React Best Practices Skill

A comprehensive guide to React development patterns, design decisions, and proven techniques for building maintainable, performant React applications.

## Table of Contents

1. [Thinking in React](#thinking-in-react)
2. [Component Fundamentals](#component-fundamentals)
3. [State & Lifecycle](#state--lifecycle)
4. [Side Effects with useEffect](#side-effects-with-useeffect)
5. [Performance Optimization](#performance-optimization)
6. [Context API](#context-api)
7. [Custom Hooks](#custom-hooks)
8. [Component Composition Patterns](#component-composition-patterns)
9. [Forms & User Input](#forms--user-input)
10. [Lists & Keys](#lists--keys)
11. [Error Handling](#error-handling)
12. [Data Fetching Patterns](#data-fetching-patterns)
13. [Common Pitfalls](#common-pitfalls)
14. [Best Practices Summary](#best-practices-summary)

---

## Thinking in React

React's core philosophy is "think in React" - a structured approach to building UIs. This 5-step process from React.dev helps you build robust applications:

### Step 1: Break the UI Into a Component Hierarchy

Start by drawing boxes around every component in your mockup and naming them.

**Key principles:**
- Components should follow the **single responsibility principle** - do one thing well
- A component can represent a UI element or a logical piece of functionality
- Consider your data structure when breaking down components
- Name components based on what they display, not how they're used

**Example hierarchy:**
```
App
├── Header
│   ├── Logo
│   └── Navigation
├── MainContent
│   ├── SearchBar
│   ├── ProductList
│   │   └── ProductCard (repeated)
│   └── Sidebar
└── Footer
```

**Questions to ask:**
- Does this piece need its own state?
- Can it be reused elsewhere?
- Does it handle one responsibility?
- Is it too large to understand quickly?

### Step 2: Build a Static Version First

Create a version with hardcoded data and no interactivity.

**Why this matters:**
- Separates UI structure concerns from state/interactivity concerns
- Makes it easier to identify bugs
- Reduces complexity when adding state later
- Builds confidence in the component tree

**Static version characteristics:**
- Props flow down from parent to child
- No useState or useEffect hooks
- No event handlers (or handlers that do nothing)
- Only displaying data that's passed in

**Example approach:**
```javascript
// 1. Build components that accept data as props
function ProductCard({ product }) {
  return (
    <div className="product">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <p>{product.description}</p>
    </div>
  );
}

// 2. Build parent components that pass data down
function ProductList({ products }) {
  return (
    <div className="products">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// 3. Build the root component with hardcoded data
function App() {
  const mockProducts = [
    { id: 1, name: "Widget", price: 19.99, description: "A useful widget" },
    { id: 2, name: "Gadget", price: 29.99, description: "A cool gadget" },
  ];
  
  return <ProductList products={mockProducts} />;
}
```

### Step 3: Identify the Minimal State

Determine which data should be state (changes over time) vs props (passed from parent).

**State requirements:**
- The data changes over time
- It cannot be computed from other variables
- It's not passed in from a parent

**Questions for each piece of data:**
1. Does it change? If no, it's probably not state.
2. Can you calculate it from other variables? If yes, it's probably not state.
3. Is it passed from a parent via props? If yes, it's not state.

**Example:**
```javascript
// What should be state in a TodoApp?

// ❌ NOT state - can be computed
const completedCount = todos.filter(t => t.completed).length;

// ❌ NOT state - passed from parent
const userId = props.userId;

// ✅ IS state - user types into input
const [inputValue, setInputValue] = useState("");

// ✅ IS state - list changes when user adds/removes
const [todos, setTodos] = useState([]);

// ✅ IS state - user selects filter
const [filter, setFilter] = useState("all");
```

**Minimal state is crucial because:**
- Less state = less to keep in sync = fewer bugs
- Makes the data flow easier to understand
- Improves performance (less re-renders)
- Makes testing easier

### Step 4: Identify Where State Should Live

Determine the appropriate component to own each piece of state.

**Rules:**
1. State should live in the **lowest common ancestor** of all components that use it
2. If no component owns it yet, create a new component above the others to hold it
3. State used by multiple siblings belongs in their parent
4. State used by only one component belongs in that component

**Decision process:**

```
For each piece of state, ask:
1. Does Component A use it? ──→ Consider A as owner
2. Does Component B also use it? ──→ Move to parent of A and B
3. Do siblings need it? ──→ Move to parent
4. Is only one component using it? ──→ Keep it there
```

**Common patterns:**

```javascript
// Pattern 1: State belongs in the component that uses it
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// Pattern 2: State should be lifted to parent
function FilteredTodoList() {
  const [filter, setFilter] = useState("all");
  const [todos, setTodos] = useState([]);
  
  return (
    <>
      <FilterButtons filter={filter} onFilterChange={setFilter} />
      <TodoList todos={todos} filter={filter} />
    </>
  );
}

// Pattern 3: State in wrong place - avoid
function BadExample() {
  // ❌ State needed by both children but in wrong place
  return (
    <>
      <ChildA hasOwnState={true} />
      <ChildB needsStateFromA={false} />
    </>
  );
}

// Pattern 3 corrected:
function GoodExample() {
  // ✅ State in parent, passed to both children
  const [sharedState, setSharedState] = useState(false);
  return (
    <>
      <ChildA state={sharedState} onChange={setSharedState} />
      <ChildB state={sharedState} onChange={setSharedState} />
    </>
  );
}
```

### Step 5: Add Inverse Data Flow

Enable child components to update parent state through callbacks.

**The pattern:**
1. Parent holds state and a handler function
2. Parent passes the handler to children as a callback prop
3. Children call the callback when user interacts
4. Parent updates state based on callback

**Data flow visualization:**

```
Parent (state holder)
  ├─ forwards state down as props ──→ Child
  └─ receives updates up via callbacks ←── Child
```

**Example:**

```javascript
function Parent() {
  const [name, setName] = useState("");
  
  // Step 1: Parent has the state and handler
  const handleNameChange = (newName) => {
    setName(newName);
  };
  
  return (
    // Step 2: Pass handler to child
    <Input value={name} onChange={handleNameChange} />
  );
}

function Input({ value, onChange }) {
  return (
    <input
      value={value}
      // Step 3: Child calls callback on change
      onChange={(e) => onChange(e.target.value)}
    />
  );
}
```

**Key principles:**
- Data flows down, events flow up
- Child components don't modify parent state directly
- Keep handlers as simple functions when possible
- Consider using useCallback to avoid recreating handlers

---

## Component Fundamentals

### Functional Components

React uses **functional components** as the standard approach. These are JavaScript functions that return JSX.

**Why functional components?**
- Simpler mental model
- Hooks only work with functional components
- Easier to test and reason about
- Smaller bundle size

**Basic structure:**

```javascript
function MyComponent(props) {
  // Component logic here
  return <div>JSX here</div>;
}

// or with arrow function
const MyComponent = (props) => {
  return <div>JSX here</div>;
};
```

### Props: Receiving Data from Parents

Props are the way to pass data and behavior from parents to children.

**Props characteristics:**
- Read-only - components should never modify props
- Can be any JavaScript value (strings, numbers, objects, functions, components)
- Passed from parent to child
- Can be destructured for cleaner code

**Prop patterns:**

```javascript
// Pattern 1: Props object
function Greeting(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// Pattern 2: Destructuring (preferred)
function Greeting({ name, greeting = "Hello" }) {
  return <h1>{greeting}, {name}!</h1>;
}

// Pattern 3: Rest operator for flexibility
function Button({ type = "button", className, ...rest }) {
  return <button type={type} className={className} {...rest} />;
}

// Usage:
<Greeting name="Alice" greeting="Hi" />
<Button onClick={handleClick} disabled>Click me</Button>
```

**Prop naming conventions:**
- Use descriptive names (e.g., `onSubmit` not `onClick` for form submission)
- Use `on` prefix for callbacks (e.g., `onItemSelect`)
- Use `is` or `has` prefix for booleans (e.g., `isLoading`, `hasError`)
- Use clear domain language (e.g., `email` not `e`)

### Children and Composition

The `children` prop is React's way to allow components to contain other components.

**Children patterns:**

```javascript
// Pattern 1: Simple children
function Card({ children }) {
  return <div className="card">{children}</div>;
}

// Usage:
<Card>
  <h2>Title</h2>
  <p>Content</p>
</Card>

// Pattern 2: Multiple slots
function Layout({ header, sidebar, footer, children }) {
  return (
    <>
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
      <footer>{footer}</footer>
    </>
  );
}

// Usage:
<Layout
  header={<Header />}
  sidebar={<Sidebar />}
  footer={<Footer />}
>
  <MainContent />
</Layout>

// Pattern 3: Children as function (render props)
function DataProvider({ children }) {
  const data = useFetch("/api/data");
  return children(data);
}

// Usage:
<DataProvider>
  {(data) => <Display data={data} />}
</DataProvider>
```

**When to use children:**
- Content wrapping (Card, Container, Modal)
- Layout composition
- Plugin systems
- Keep parent components generic and reusable

### Default Props and Prop Types

Provide sensible defaults and document prop expectations.

**Using default parameters:**

```javascript
function Button({ label = "Click me", disabled = false, onClick }) {
  return <button disabled={disabled} onClick={onClick}>{label}</button>;
}
```

**Using PropTypes (optional but helpful):**

```javascript
import PropTypes from 'prop-types';

function ProductCard({ title, price, image }) {
  return (
    <div>
      <img src={image} alt={title} />
      <h3>{title}</h3>
      <p>${price}</p>
    </div>
  );
}

ProductCard.propTypes = {
  title: PropTypes.string.isRequired,
  price: PropTypes.number.isRequired,
  image: PropTypes.string.isRequired,
};
```

**Using TypeScript (recommended for larger projects):**

```typescript
interface ProductCardProps {
  title: string;
  price: number;
  image: string;
  onBuy?: (productId: string) => void;
}

function ProductCard({ title, price, image, onBuy }: ProductCardProps) {
  return (
    <div>
      <img src={image} alt={title} />
      <h3>{title}</h3>
      <p>${price}</p>
    </div>
  );
}
```

### Prop Drilling and When It's Okay

Prop drilling is passing props through multiple levels of components. It's okay when:
- The chain is 2-3 levels deep
- The data is closely related to the component hierarchy
- Using context would add unnecessary complexity

**When to avoid deep prop drilling:**
- More than 4-5 levels of drilling
- Data is used in many places but not related to hierarchy
- Consider using Context API or state management library

---

## State & Lifecycle

### useState Hook Basics

useState is the fundamental hook for managing component state.

**Basic usage:**

```javascript
import { useState } from 'react';

function Counter() {
  // Declare state variable
  const [count, setCount] = useState(0);
  
  return (
    <>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </>
  );
}
```

**Multiple state variables:**

```javascript
function Form() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [submitted, setSubmitted] = useState(false);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    setSubmitted(true);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**State initialization with function:**

```javascript
// Use function initializer for expensive computations
function LocalStorageExample() {
  const [items, setItems] = useState(() => {
    // This function runs only once on mount
    const saved = localStorage.getItem("items");
    return saved ? JSON.parse(saved) : [];
  });
  
  return <ItemList items={items} />;
}
```

### State Updates and Batching

React batches state updates for performance.

**State update patterns:**

```javascript
// Pattern 1: Simple value update
setCount(count + 1);

// Pattern 2: Update function (recommended for dependent updates)
setCount(prevCount => prevCount + 1);

// Pattern 3: Object state with spread
const [form, setForm] = useState({ name: "", email: "" });
setForm({ ...form, name: "Alice" });

// Pattern 4: Array state with spread
const [items, setItems] = useState([]);
setItems([...items, newItem]);
```

**Batching behavior:**

```javascript
function Example() {
  const [x, setX] = useState(0);
  const [y, setY] = useState(0);
  
  const handleClick = () => {
    // Both updates are batched - one re-render
    setX(x + 1);
    setY(y + 1);
  };
  
  const handleAsync = async () => {
    // Without batching (React 17 and earlier)
    // Each would trigger separate re-render
    await sleep(100);
    setX(x + 1);
    setY(y + 1);
  };
  
  return <button onClick={handleClick}>Click</button>;
}
```

### When to Use State vs Props

**Use state when:**
- Data changes over time
- Component needs to manage its own data
- Data is specific to this component instance
- User interaction changes the value

**Use props when:**
- Data comes from the parent
- Data is read-only for this component
- Multiple components need the same data
- Data is configuration/options

**Example decision:**

```javascript
// Form component - use state for user input
function SignupForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  // State - data changes as user types
  
  return (
    <form>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
    </form>
  );
}

// Display component - use props for received data
function UserProfile({ userId, userName, userEmail }) {
  // Props - read-only data from parent
  return (
    <div>
      <h1>{userName}</h1>
      <p>{userEmail}</p>
    </div>
  );
}
```

### Managing Complex State

For complex state, organize it well:

```javascript
// Pattern 1: Multiple simple states (okay for 2-3 variables)
const [email, setEmail] = useState("");
const [password, setPassword] = useState("");
const [isLoading, setIsLoading] = useState(false);

// Pattern 2: Single object state (better for related data)
const [form, setForm] = useState({
  email: "",
  password: "",
  isLoading: false,
});

// Pattern 3: useReducer for complex logic (best for complex scenarios)
const [state, dispatch] = useReducer(reducer, initialState);
```

### useReducer for Complex State

useReducer is better when:
- Multiple sub-values related to same concept
- Complex state transitions
- Easier to test logic separately from component

**Basic pattern:**

```javascript
const initialState = { count: 0, step: 1 };

function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + state.step };
    case "DECREMENT":
      return { ...state, count: state.count - state.step };
    case "SET_STEP":
      return { ...state, step: action.payload };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <>
      <p>Count: {state.count}, Step: {state.step}</p>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>
        Increment
      </button>
      <input
        value={state.step}
        onChange={(e) => dispatch({
          type: "SET_STEP",
          payload: parseInt(e.target.value)
        })}
      />
    </>
  );
}
```

---

## Side Effects with useEffect

### useEffect Fundamentals

useEffect lets you perform side effects in functional components.

**Basic syntax:**

```javascript
useEffect(() => {
  // Effect code runs after render
  console.log("Component mounted or updated");
  
  // Optional: return cleanup function
  return () => {
    console.log("Component unmounting or before effect re-runs");
  };
}, [dependencies]); // Dependency array
```

**Effect timing:**

```javascript
function Example() {
  // Runs after EVERY render
  useEffect(() => {
    console.log("After every render");
  });
  
  // Runs only on mount (empty dependency array)
  useEffect(() => {
    console.log("On mount only");
  }, []);
  
  // Runs when dependencies change
  useEffect(() => {
    console.log("When these dependencies change");
  }, [dependency1, dependency2]);
}
```

### Dependency Arrays (Why They Matter)

The dependency array tells React when to re-run the effect.

**Correct dependency array:**

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // This effect depends on userId
    fetchUser(userId).then(setUser);
  }, [userId]); // Include userId in dependencies
  
  return <div>{user?.name}</div>;
}
```

**Common mistakes:**

```javascript
// ❌ Missing dependency - stale userId
function BadFetch({ userId }) {
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, []); // Missing userId!
}

// ❌ Unnecessary dependencies - re-runs too often
function BadRendering({ userId, userName }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    // This only needs userId
    fetchData(userId).then(setData);
  }, [userId, userName]); // userName is unnecessary
}

// ✅ Correct dependencies
function GoodFetch({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]); // Only what matters
}
```

**Objects and dependency arrays:**

```javascript
// ❌ Creates new object every render - effect runs every time
const options = { sort: "name" };
useEffect(() => {
  fetchUsers(options);
}, [options]); // Object reference is new each time

// ✅ Move outside or useMemo
const options = useMemo(() => ({ sort: "name" }), []);
useEffect(() => {
  fetchUsers(options);
}, [options]);

// ✅ Or pass the specific value
useEffect(() => {
  fetchUsers({ sort: "name" });
}, ["name"]); // Depend on the specific value
```

### Cleanup Functions

Effects can return cleanup functions to prevent memory leaks.

**Common cleanup scenarios:**

```javascript
// Timer cleanup
useEffect(() => {
  const timer = setInterval(() => {
    console.log("Tick");
  }, 1000);
  
  return () => clearInterval(timer); // Cleanup
}, []);

// Event listener cleanup
useEffect(() => {
  const handleResize = () => {
    console.log("Window resized");
  };
  
  window.addEventListener("resize", handleResize);
  
  return () => {
    window.removeEventListener("resize", handleResize); // Cleanup
  };
}, []);

// Subscription cleanup
useEffect(() => {
  const subscription = dataSource.subscribe();
  
  return () => {
    subscription.unsubscribe(); // Cleanup
  };
}, []);

// Abort fetch cleanup
useEffect(() => {
  const controller = new AbortController();
  
  fetch("/api/data", { signal: controller.signal })
    .then(res => res.json())
    .then(data => setData(data));
  
  return () => {
    controller.abort(); // Cleanup
  };
}, []);
```

### Common useEffect Patterns

**Pattern 1: Fetch on mount**

```javascript
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    setLoading(true);
    fetch("/api/users")
      .then(res => res.json())
      .then(data => {
        setUsers(data);
        setLoading(false);
      });
  }, []); // Only run once
  
  if (loading) return <div>Loading...</div>;
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

**Pattern 2: Sync with prop change**

```javascript
function UserDetail({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, [userId]); // Re-fetch when userId changes
  
  return user ? <div>{user.name}</div> : <div>Loading...</div>;
}
```

**Pattern 3: Form field sync**

```javascript
function EditForm({ initialData }) {
  const [form, setForm] = useState(initialData);
  
  useEffect(() => {
    // Reset form when initialData changes
    setForm(initialData);
  }, [initialData]);
  
  return (
    <input
      value={form.name}
      onChange={(e) => setForm({ ...form, name: e.target.value })}
    />
  );
}
```

### useLayoutEffect vs useEffect

**useEffect:**
- Runs after paint
- Most effects should use this
- Non-blocking

**useLayoutEffect:**
- Runs before paint (synchronously)
- Use for DOM measurements/mutations
- Can block painting

```javascript
// Good use case for useLayoutEffect
function MeasuredComponent() {
  const [height, setHeight] = useState(0);
  const ref = useRef(null);
  
  useLayoutEffect(() => {
    // Measure DOM immediately before paint
    setHeight(ref.current.offsetHeight);
  }, []);
  
  return <div ref={ref}>Content</div>;
}

// Most effects should use useEffect
function DataComponent() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    // Fetch data after render
    fetch("/api/data").then(res => res.json()).then(setData);
  }, []);
  
  return <div>{data}</div>;
}
```

---

## Performance Optimization

### useMemo for Expensive Computations

useMemo memoizes expensive computations.

**When to use:**
- Computing is genuinely expensive
- Component re-renders frequently
- Value is passed to children (causes their re-renders)

**Correct usage:**

```javascript
// Good: expensive computation
function SortedList({ items, sortBy }) {
  const sortedItems = useMemo(() => {
    console.log("Sorting...");
    return [...items].sort((a, b) => {
      if (sortBy === "name") return a.name.localeCompare(b.name);
      if (sortBy === "price") return a.price - b.price;
      return 0;
    });
  }, [items, sortBy]);
  
  return sortedItems.map(item => <Item key={item.id} item={item} />);
}

// Overly cautious: simple object
function BadUseMemo() {
  const obj = useMemo(() => ({ count: 1 }), []);
  // Probably unnecessary - object is simple
  return <span>{obj.count}</span>;
}
```

### useCallback for Stable Function References

useCallback prevents unnecessary re-renders of children that receive the function as a prop.

**When to use:**
- Function is passed to memoized child
- Function is in a dependency array
- Function is expensive to recreate

**Correct usage:**

```javascript
function Parent() {
  const [items, setItems] = useState([]);
  
  // Without useCallback: new function every render
  // Child would re-render unnecessarily
  const handleAdd = useCallback((item) => {
    setItems(prev => [...prev, item]);
  }, []);
  
  return <MemoizedChild onAdd={handleAdd} />;
}

const MemoizedChild = React.memo(({ onAdd }) => {
  console.log("Child rendered");
  return <button onClick={() => onAdd("new")}>Add</button>;
});
```

**Common mistake:**

```javascript
// ❌ useCallback with unnecessary dependencies
function BadCallback() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log(count);
  }, [count]); // This defeats the purpose!
  
  return <button onClick={handleClick}>{count}</button>;
}

// ✅ Only depend on what's needed
function GoodCallback() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    // Don't access count if you don't need it
    console.log("Clicked");
  }, []);
  
  return <button onClick={handleClick}>{count}</button>;
}
```

### React.memo for Component Memoization

React.memo prevents re-rendering if props haven't changed.

**When to use:**
- Component is expensive to render
- It re-renders with same props frequently
- Props don't change every render

**Basic usage:**

```javascript
// Memoize expensive component
const ExpensiveList = React.memo(({ items }) => {
  console.log("Rendering list");
  return items.map(item => <Item key={item.id} item={item} />);
});

// Component that uses it
function Container() {
  const [filter, setFilter] = useState("");
  const items = useMemo(() => [...], [filter]);
  
  return (
    <>
      <Input value={filter} onChange={setFilter} />
      <ExpensiveList items={items} />
    </>
  );
}
```

**With custom comparison:**

```javascript
const UserCard = React.memo(
  ({ user, onUpdate }) => {
    return (
      <div>
        <h3>{user.name}</h3>
        <button onClick={onUpdate}>Update</button>
      </div>
    );
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    return prevProps.user.id === nextProps.user.id;
  }
);
```

### When Optimization Is Premature

**Don't optimize if:**
- Component renders quickly
- Renders infrequently
- You haven't profiled and found a problem
- It makes code harder to understand

**Profile first:**

```javascript
// Use Profiler API to measure
import { Profiler } from "react";

function MyApp() {
  const onRenderCallback = (id, phase, actualDuration) => {
    console.log(`${id} (${phase}) took ${actualDuration}ms`);
  };
  
  return (
    <Profiler id="MyApp" onRender={onRenderCallback}>
      <App />
    </Profiler>
  );
}
```

---

## Context API

### Creating and Using Context

Context provides a way to pass data without prop drilling.

**Basic pattern:**

```javascript
import { createContext, useState, useContext } from "react";

// Step 1: Create context
const ThemeContext = createContext();

// Step 2: Create provider component
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");
  
  const toggleTheme = () => {
    setTheme(t => t === "light" ? "dark" : "light");
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Step 3: Create custom hook for using context
export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error("useTheme must be used inside ThemeProvider");
  }
  return context;
}

// Step 4: Use in component
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}

function Header() {
  const { theme, toggleTheme } = useTheme();
  return (
    <header className={theme}>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </header>
  );
}
```

### Context vs Props vs State Management

**Use props when:**
- Data is only needed by 1-2 children
- Data comes from parent naturally
- Prop drilling is minimal (1-2 levels)

**Use context when:**
- Data needed by deeply nested components
- Data is "global" to a feature (theme, auth, language)
- Data is accessed by many unrelated components

**Use external state management when:**
- Complex state transitions
- Multiple independent state sources
- Need for devtools/debugging
- Large team working on app

```javascript
// ❌ Prop drilling too deep
function App() {
  const user = { name: "Alice" };
  return <Level1 user={user} />;
}

function Level1({ user }) {
  return <Level2 user={user} />;
}

function Level2({ user }) {
  return <Level3 user={user} />;
}

function Level3({ user }) {
  return <div>{user.name}</div>;
}

// ✅ Use context instead
const UserContext = createContext();

function App() {
  const user = { name: "Alice" };
  return (
    <UserContext.Provider value={user}>
      <Level1 />
    </UserContext.Provider>
  );
}

function Level3() {
  const user = useContext(UserContext);
  return <div>{user.name}</div>;
}
```

### Context Splitting for Performance

Large context values cause all consumers to re-render. Split context into separate providers.

```javascript
// ❌ Too much in one context
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState("light");
  const [notifications, setNotifications] = useState([]);
  
  return (
    <AppContext.Provider value={{ user, theme, notifications, setUser, setTheme }}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ Split into focused contexts
const UserContext = createContext();
const ThemeContext = createContext();
const NotificationContext = createContext();

export function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState("light");
  const [notifications, setNotifications] = useState([]);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <NotificationContext.Provider value={{ notifications, setNotifications }}>
          {children}
        </NotificationContext.Provider>
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// Further optimization: separate state and dispatch
const UserStateContext = createContext();
const UserDispatchContext = createContext();

function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  
  return (
    <UserStateContext.Provider value={user}>
      <UserDispatchContext.Provider value={setUser}>
        {children}
      </UserDispatchContext.Provider>
    </UserStateContext.Provider>
  );
}
```

### When NOT to Use Context

**Avoid context for:**
- Simple prop passing (just use props)
- Frequently changing values (causes many re-renders)
- Server state (use React Query, SWR)
- Form fields that change on every keystroke
- Complex state (use state management library)

---

## Custom Hooks

### Creating Reusable Hook Logic

Custom hooks let you extract logic into reusable functions.

**Basic pattern:**

```javascript
// Hook: useLocalStorage
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  const setStorageValue = useCallback((newValue) => {
    try {
      const valueToStore = newValue instanceof Function ? newValue(value) : newValue;
      setValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (e) {
      console.error(e);
    }
  }, [key, value]);
  
  return [value, setStorageValue];
}

// Usage:
function MyComponent() {
  const [name, setName] = useLocalStorage("name", "");
  
  return (
    <>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <p>Name saved to localStorage</p>
    </>
  );
}
```

### Hook Rules

React hooks have specific rules to work correctly:

**Rule 1: Only call hooks at the top level**
```javascript
// ❌ Inside condition - wrong
function BadHook() {
  if (condition) {
    const [state, setState] = useState(0); // WRONG!
  }
}

// ✅ Top level - correct
function GoodHook() {
  const [state, setState] = useState(0); // Correct
  
  if (condition) {
    // Use state here is fine
  }
}
```

**Rule 2: Only call hooks from React functions**
```javascript
// ❌ In regular function - wrong
function regularFunction() {
  useState(0); // WRONG!
}

// ✅ In component or custom hook - correct
function Component() {
  useState(0); // Correct
}

function useCustomHook() {
  useState(0); // Correct
}
```

### useEffect-based Hook Patterns

Common patterns for building hooks with useEffect:

```javascript
// Pattern 1: Fetch hook
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    if (!url) return;
    
    const fetchData = async () => {
      try {
        const res = await fetch(url);
        if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
        const json = await res.json();
        setData(json);
      } catch (e) {
        setError(e);
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
  }, [url]);
  
  return { data, loading, error };
}

// Pattern 2: Timer hook
function useTimer(duration) {
  const [timeLeft, setTimeLeft] = useState(duration);
  
  useEffect(() => {
    if (timeLeft === 0) return;
    
    const timer = setInterval(() => {
      setTimeLeft(t => t - 1);
    }, 1000);
    
    return () => clearInterval(timer);
  }, [timeLeft]);
  
  return timeLeft;
}

// Pattern 3: Async operation hook
function useAsync(asyncFunction, immediate = true) {
  const [status, setStatus] = useState("idle");
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  
  const execute = useCallback(async () => {
    setStatus("pending");
    try {
      const response = await asyncFunction();
      setData(response);
      setStatus("success");
    } catch (e) {
      setError(e);
      setStatus("error");
    }
  }, [asyncFunction]);
  
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);
  
  return { status, data, error, execute };
}
```

---

## Component Composition Patterns

### Composition Over Configuration

Build flexible components through composition rather than configuration props.

```javascript
// ❌ Configuration overload
function Card({ title, subtitle, image, imagePosition, body, footer, onFooterClick, footerVariant, ... }) {
  // Too many props, hard to use, inflexible
}

// ✅ Composition approach
function Card({ children }) {
  return <div className="card">{children}</div>;
}

function CardHeader({ children }) {
  return <div className="card-header">{children}</div>;
}

function CardBody({ children }) {
  return <div className="card-body">{children}</div>;
}

function CardFooter({ children }) {
  return <div className="card-footer">{children}</div>;
}

// Usage: flexible and clean
<Card>
  <CardHeader>
    <h2>Title</h2>
  </CardHeader>
  <CardBody>
    <p>Content here</p>
  </CardBody>
  <CardFooter>
    <button>Action</button>
  </CardFooter>
</Card>
```

### Compound Components

Compound components are a group of components designed to work together.

```javascript
// Compound component pattern
function Select({ children, value, onChange }) {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div className="select">
      <SelectButton value={value} onClick={() => setIsOpen(!isOpen)}>
        {value || "Select..."}
      </SelectButton>
      {isOpen && (
        <SelectOptions>
          {children}
        </SelectOptions>
      )}
    </div>
  );
}

function SelectOption({ value, children }) {
  return <div className="option">{children}</div>;
}

// Usage:
<Select value={selected} onChange={setSelected}>
  <SelectOption value="option1">Option 1</SelectOption>
  <SelectOption value="option2">Option 2</SelectOption>
</Select>
```

### Render Props Pattern

Pass a function as a prop to control rendering.

```javascript
function DataProvider({ render }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch("/api/data")
      .then(res => res.json())
      .then(setData);
  }, []);
  
  return render(data);
}

// Usage:
<DataProvider render={(data) => (
  data ? <Display data={data} /> : <Loading />
)} />

// Or with children as function:
function DataProvider({ children }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch("/api/data")
      .then(res => res.json())
      .then(setData);
  }, []);
  
  return children(data);
}

<DataProvider>
  {(data) => data ? <Display data={data} /> : <Loading />}
</DataProvider>
```

### Higher-Order Components (When to Use)

HOCs wrap a component and enhance it. Use sparingly.

```javascript
// Theme HOC
function withTheme(Component) {
  return function ThemedComponent(props) {
    const { theme, toggleTheme } = useTheme();
    
    return <Component {...props} theme={theme} toggleTheme={toggleTheme} />;
  };
}

// Usage:
function MyComponent({ theme, toggleTheme }) {
  return <div className={theme}>Content</div>;
}

export default withTheme(MyComponent);
```

**When to use HOCs:**
- Cross-cutting concerns (auth, logging)
- When you need to manipulate props
- When you need to manipulate render output

**Prefer hooks instead** for most cases.

### Passing Components as Props

Components can accept other components as props for maximum flexibility.

```javascript
function Layout({ Header, Sidebar, Footer, children }) {
  return (
    <>
      <Header />
      <aside><Sidebar /></aside>
      <main>{children}</main>
      <Footer />
    </>
  );
}

// Usage:
<Layout
  Header={MyHeader}
  Sidebar={MySidebar}
  Footer={MyFooter}
>
  <MainContent />
</Layout>
```

---

## Forms & User Input

### Controlled vs Uncontrolled Components

**Controlled components**: React manages the value

```javascript
function ControlledInput() {
  const [value, setValue] = useState("");
  
  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

**Uncontrolled components**: DOM manages the value

```javascript
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

**Prefer controlled components** for most cases because:
- React controls the state
- Validation easier
- Dynamic changes easier
- Testing easier

### Form Handling Patterns

```javascript
function ContactForm() {
  const [form, setForm] = useState({
    name: "",
    email: "",
    message: "",
  });
  
  const [submitted, setSubmitted] = useState(false);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm(prev => ({
      ...prev,
      [name]: value,
    }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Validate
    if (!form.name || !form.email || !form.message) {
      alert("All fields required");
      return;
    }
    
    // Submit
    console.log("Submitting:", form);
    setSubmitted(true);
    
    // Reset
    setTimeout(() => {
      setForm({ name: "", email: "", message: "" });
      setSubmitted(false);
    }, 2000);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={form.name}
        onChange={handleChange}
        placeholder="Name"
      />
      <input
        name="email"
        type="email"
        value={form.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <textarea
        name="message"
        value={form.message}
        onChange={handleChange}
        placeholder="Message"
      />
      <button type="submit" disabled={submitted}>
        {submitted ? "Submitting..." : "Submit"}
      </button>
    </form>
  );
}
```

### Input Validation

```javascript
function ValidatedForm() {
  const [form, setForm] = useState({ email: "", password: "" });
  const [errors, setErrors] = useState({});
  
  const validate = () => {
    const newErrors = {};
    
    if (!form.email) {
      newErrors.email = "Email required";
    } else if (!form.email.includes("@")) {
      newErrors.email = "Invalid email";
    }
    
    if (!form.password) {
      newErrors.password = "Password required";
    } else if (form.password.length < 8) {
      newErrors.password = "Password must be 8+ characters";
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (validate()) {
      console.log("Form valid, submitting");
    }
  };
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
    // Clear error for this field when user types
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: "" }));
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="email"
          value={form.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## Lists & Keys

### Why Keys Matter

Keys help React identify which items have changed, improving performance and preventing state issues.

```javascript
// ❌ No keys - can cause issues
function ItemList({ items }) {
  return (
    <ul>
      {items.map(item => <li>{item.name}</li>)}
    </ul>
  );
}

// ✅ With unique keys
function ItemList({ items }) {
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}
```

### Choosing Good Keys

Keys should be:
- Unique among siblings
- Stable across re-renders
- Not based on array index (which can change)

```javascript
// ❌ Index as key - bad for reorderable lists
{items.map((item, index) => <Item key={index} item={item} />)}

// ✅ Unique ID - best
{items.map(item => <Item key={item.id} item={item} />)}

// ✅ Content-based ID if no explicit ID
{items.map(item => <Item key={`${item.category}-${item.name}`} item={item} />)}
```

### Rendering Lists Efficiently

```javascript
function OptimizedList({ items, filter }) {
  // Filter outside render
  const filtered = useMemo(() => {
    return items.filter(item => item.category === filter);
  }, [items, filter]);
  
  return (
    <ul>
      {filtered.map(item => (
        <ListItem key={item.id} item={item} />
      ))}
    </ul>
  );
}

// Memoize list item to prevent re-renders
const ListItem = React.memo(({ item }) => {
  return <li>{item.name}</li>;
});
```

---

## Error Handling

### Error Boundaries

Error boundaries catch errors in their child components.

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
    console.log("Error caught:", error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong: {this.state.error.message}</h1>;
    }
    
    return this.props.children;
  }
}

// Usage:
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

**Limitations:**
- Only catches errors in render, not event handlers
- Doesn't catch async errors
- Doesn't catch server-side errors
- Need class components (no hook version yet)

### Error Handling in Event Handlers

```javascript
function SafeButton() {
  const handleClick = async () => {
    try {
      await riskyOperation();
    } catch (error) {
      console.error("Operation failed:", error);
      // Show error to user
    }
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

### Graceful Error Recovery

```javascript
function ResilientComponent() {
  const [error, setError] = useState(null);
  const [data, setData] = useState(null);
  
  const handleRetry = async () => {
    try {
      setError(null);
      const response = await fetch("/api/data");
      const json = await response.json();
      setData(json);
    } catch (e) {
      setError(e.message);
    }
  };
  
  useEffect(() => {
    handleRetry();
  }, []);
  
  if (error) {
    return (
      <div>
        <p>Error: {error}</p>
        <button onClick={handleRetry}>Try again</button>
      </div>
    );
  }
  
  return <div>{data && <Display data={data} />}</div>;
}
```

---

## Data Fetching Patterns

### Fetching in useEffect

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let isMounted = true;
    
    const fetchUser = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error("Failed to fetch");
        const data = await response.json();
        
        if (isMounted) {
          setUser(data);
        }
      } catch (e) {
        if (isMounted) {
          setError(e.message);
        }
      } finally {
        if (isMounted) {
          setLoading(false);
        }
      }
    };
    
    fetchUser();
    
    return () => {
      isMounted = false; // Cleanup: mark component as unmounted
    };
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>{user.name}</div>;
}
```

### Race Conditions and Cleanup

```javascript
// Better: use AbortController
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    if (!query) {
      setResults([]);
      return;
    }
    
    const controller = new AbortController();
    
    setLoading(true);
    fetch(`/api/search?q=${query}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => setResults(data))
      .catch(e => {
        if (e.name !== "AbortError") {
          console.error(e);
        }
      })
      .finally(() => setLoading(false));
    
    return () => controller.abort(); // Cancel fetch on cleanup
  }, [query]);
  
  return (
    <div>
      {loading && <div>Loading...</div>}
      {results.map(result => <Result key={result.id} result={result} />)}
    </div>
  );
}
```

### Loading and Error States

```javascript
function DataDisplay() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  useEffect(() => {
    dispatch({ type: "FETCH_START" });
    
    fetch("/api/data")
      .then(res => res.json())
      .then(data => dispatch({ type: "FETCH_SUCCESS", payload: data }))
      .catch(error => dispatch({ type: "FETCH_ERROR", payload: error }));
  }, []);
  
  const { status, data, error } = state;
  
  switch (status) {
    case "idle":
      return <div>Ready</div>;
    case "pending":
      return <div>Loading...</div>;
    case "success":
      return <Display data={data} />;
    case "error":
      return <div>Error: {error.message}</div>;
    default:
      return null;
  }
}

function reducer(state, action) {
  switch (action.type) {
    case "FETCH_START":
      return { ...state, status: "pending" };
    case "FETCH_SUCCESS":
      return { ...state, status: "success", data: action.payload };
    case "FETCH_ERROR":
      return { ...state, status: "error", error: action.payload };
    default:
      return state;
  }
}

const initialState = { status: "idle", data: null, error: null };
```

---

## Common Pitfalls

### Mutating State Directly

```javascript
// ❌ Direct mutation - state won't update
const [user, setUser] = useState({ name: "Alice", age: 30 });

const handleBirthday = () => {
  user.age = user.age + 1; // Wrong! Mutating state directly
  setUser(user); // React won't detect change
};

// ✅ Create new object with spread
const handleBirthday = () => {
  setUser({ ...user, age: user.age + 1 });
};

// Arrays too
const [items, setItems] = useState(["a", "b"]);

// ❌ Wrong - mutating array
items.push("c");
setItems(items);

// ✅ Correct - new array
setItems([...items, "c"]);

// ✅ Also correct - filter returns new array
setItems(items.filter(item => item !== "a"));
```

### Stale Closures

```javascript
// ❌ Stale closure - uses old count value
function Counter() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    setTimeout(() => {
      console.log(`Count is ${count}`); // Always logs 0
      setCount(count + 1); // Always sets to 1
    }, 1000);
  };
  
  return <button onClick={handleClick}>{count}</button>;
}

// ✅ Use function form of setState
function Counter() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    setTimeout(() => {
      setCount(prevCount => prevCount + 1); // Uses current value
    }, 1000);
  };
  
  return <button onClick={handleClick}>{count}</button>;
}
```

### Missing useEffect Dependencies

```javascript
// ❌ Missing dependency - uses stale userId
function UserData({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`).then(res => res.json()).then(setUser);
  }, []); // Missing userId!
}

// ✅ Include dependency
function UserData({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`).then(res => res.json()).then(setUser);
  }, [userId]); // Correct
}
```

### Creating Functions in Render

```javascript
// ❌ Creates new function every render
function MyComponent() {
  const [items, setItems] = useState([]);
  
  return (
    <List
      items={items}
      onAdd={() => setItems([...items, Math.random()])} // New function each time!
    />
  );
}

// ✅ Use useCallback or define outside
function MyComponent() {
  const [items, setItems] = useState([]);
  
  const handleAdd = useCallback(() => {
    setItems(prev => [...prev, Math.random()]);
  }, []);
  
  return <List items={items} onAdd={handleAdd} />;
}
```

### Prop Drilling Too Deep

See "Prop Drilling and When It's Okay" above.

### Context Overuse

```javascript
// ❌ Putting everything in context
const AppContext = createContext();
// Many values, frequent changes, causes excessive re-renders

// ✅ Use context only for data that:
// - Changes infrequently
// - Is used by many components
// - Is conceptually global
```

---

## Best Practices Summary

### Code Organization

**File structure:**
```
src/
├── components/
│   ├── Button/
│   │   ├── Button.jsx
│   │   ├── Button.css
│   │   └── Button.test.jsx
│   ├── Card/
│   │   ├── Card.jsx
│   │   └── Card.css
│   └── ...
├── hooks/
│   ├── useLocalStorage.js
│   ├── useFetch.js
│   └── ...
├── context/
│   ├── ThemeContext.js
│   └── AuthContext.js
├── utils/
│   ├── api.js
│   ├── helpers.js
│   └── ...
├── App.jsx
└── index.jsx
```

### Naming Conventions

- **Components**: PascalCase (UserProfile, Header)
- **Hooks**: camelCase starting with "use" (useLocalStorage, useFetch)
- **Functions/constants**: camelCase (handleClick, MAX_ITEMS)
- **Private/internal**: prefixed with underscore (_internalFunction)

### Import/Export Patterns

```javascript
// ✅ Named exports for multiple exports
export function Button() { ... }
export function Link() { ... }

// ✅ Default export for main component
export default function Card() { ... }

// ✅ Logical imports
import { Button, Link } from './components';
import Card from './components/Card';

// ✅ Avoid
export default { Button, Link }; // Makes named imports awkward
```

---

## See Also

- [EXAMPLES.md](./EXAMPLES.md) - Production-ready code examples
- [ANTI-PATTERNS.md](./ANTI-PATTERNS.md) - Common mistakes and how to fix them
- [COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md) - Deep dive into composition strategies
- [STATE-MANAGEMENT-GUIDE.md](./STATE-MANAGEMENT-GUIDE.md) - Deciding where state should live
- [HOOKS-REFERENCE.md](./HOOKS-REFERENCE.md) - Quick reference for all hooks
- [QUICKSTART.md](./QUICKSTART.md) - Copy-paste templates for common patterns
- [React Testable Code Skill](../react-testable-code/) - Writing React for testing
