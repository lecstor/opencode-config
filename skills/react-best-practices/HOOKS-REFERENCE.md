# React Hooks Reference

Quick reference for React's built-in hooks.

## useState

Adds state to functional components.

**What it does:**
- Manages a single piece of state
- Returns [currentValue, setterFunction]
- Setter can take a new value or update function

**When to use:**
- Managing local component state
- Form inputs
- UI toggles (modal open, menu expanded)
- Any value that changes over time

**Common mistakes:**
```javascript
// ❌ Don't mutate state directly
const [items, setItems] = useState([]);
items.push(newItem); // WRONG

// ✅ Create new array
setItems([...items, newItem]);

// ❌ Don't depend on state within same effect
const [count, setCount] = useState(0);
setCount(count + 1); // Stale closure

// ✅ Use function form
setCount(prevCount => prevCount + 1);
```

**Example:**
```javascript
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </>
  );
}
```

**Performance implications:**
- One useState call per re-render (cheap)
- Setting state causes re-render
- setState is batched (React 18+)

---

## useEffect

Performs side effects in functional components.

**What it does:**
- Runs code after component renders
- Returns cleanup function (optional)
- Dependency array controls when it runs

**When to use:**
- Fetching data
- Subscribing to events
- Timers and intervals
- DOM manipulation
- Analytics tracking

**Common mistakes:**
```javascript
// ❌ Missing dependencies - stale data
useEffect(() => {
  fetch(`/api/users/${userId}`).then(setUser);
}, []); // Missing userId

// ✅ Include all dependencies
useEffect(() => {
  fetch(`/api/users/${userId}`).then(setUser);
}, [userId]);

// ❌ Infinite loop - updating dependency
useEffect(() => {
  setCount(count + 1);
}, [count]); // Sets count, triggers effect again

// ✅ Use function form
useEffect(() => {
  setCount(prev => prev + 1);
}, []); // Safe, no dependencies needed
```

**Example:**
```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, [userId]); // Re-fetch when userId changes

  return <div>{user?.name}</div>;
}
```

**Cleanup:**
```javascript
useEffect(() => {
  const timer = setInterval(() => console.log('tick'), 1000);
  
  // Cleanup function - runs before effect re-runs or component unmounts
  return () => clearInterval(timer);
}, []);
```

**Performance implications:**
- Can impact performance if dependencies are wrong
- Cleanup prevents memory leaks
- Don't overuse - keep effects focused

---

## useContext

Accesses context values without drilling props.

**What it does:**
- Returns the context value
- Must be used inside a Provider
- Causes re-render when context value changes

**When to use:**
- Global configuration (theme, language)
- Authentication state
- Avoiding prop drilling
- Deeply nested components

**Common mistakes:**
```javascript
// ❌ Using outside provider
const value = useContext(MyContext); // Error if not in provider

// ✅ Always have provider
<MyContext.Provider value={...}>
  <Component />
</MyContext.Provider>

// ❌ All consumers re-render on any change
const BigContext = createContext();
const { thing1, thing2 } = useContext(BigContext);
// If thing1 OR thing2 changes, component re-renders

// ✅ Split contexts by concern
const Thing1Context = createContext();
const Thing2Context = createContext();
```

**Example:**
```javascript
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Header />
    </ThemeContext.Provider>
  );
}

function Header() {
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Toggle Theme
    </button>
  );
}
```

**Performance implications:**
- All consumers re-render when context changes
- Even if they only use part of it
- Memoize context value to prevent unnecessary renders

---

## useReducer

Manages complex state with multiple related values.

**What it does:**
- More powerful than useState for complex state
- Takes a reducer function and initial state
- Reducer defines how state changes
- Returns [state, dispatch]

**When to use:**
- Multiple related state values
- State transitions that depend on previous state
- Sharing state logic between components
- When setState logic is complex

**Common mistakes:**
```javascript
// ❌ Mutating state in reducer
function reducer(state, action) {
  state.count++; // WRONG - mutating
  return state;
}

// ✅ Return new state
function reducer(state, action) {
  return { ...state, count: state.count + 1 };
}

// ❌ Not including all needed dependencies
const increment = () => dispatch({ type: 'INCREMENT' });
useEffect(() => {
  const timer = setInterval(increment, 1000);
  return () => clearInterval(timer);
}, []); // increment changes every render
```

**Example:**
```javascript
const initialState = { count: 0, loading: false };

function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'LOAD_START':
      return { ...state, loading: true };
    case 'LOAD_SUCCESS':
      return { ...state, loading: false, count: action.payload };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>
        Increment
      </button>
    </>
  );
}
```

**Performance implications:**
- Slightly more overhead than useState
- Better for complex state organization
- Can optimize with memoization

---

## useCallback

Memoizes a function so it has a stable reference.

**What it does:**
- Returns the same function reference if dependencies haven't changed
- Prevents unnecessary re-renders of memoized children
- Prevents unnecessary re-runs of effects

**When to use:**
- Function is passed to memoized child component
- Function is in an effect dependency array
- Function should not be recreated on every render

**Common mistakes:**
```javascript
// ❌ Including unnecessary dependencies
const handleClick = useCallback(() => {
  console.log('clicked');
}, [somethingThatChangesOften]); // Defeats the purpose

// ✅ Only include what's needed
const handleClick = useCallback(() => {
  console.log('clicked');
}, []);

// ❌ Over-optimizing
const getInitials = useCallback((name) => {
  // Simple computation doesn't need memoization
  return name.split(' ').map(n => n[0]).join('');
}, []);

// ✅ Use only for expensive operations
const expensiveCalculation = useCallback(() => {
  return complexAlgorithm();
}, []);
```

**Example:**
```javascript
function Parent() {
  const [items, setItems] = useState([]);
  
  const handleAddItem = useCallback((item) => {
    setItems(prev => [...prev, item]);
  }, []); // Function reference is stable
  
  return <MemoizedChild onAdd={handleAddItem} />;
}

const MemoizedChild = React.memo(({ onAdd }) => {
  // Only re-renders if onAdd changes (it won't)
  return <button onClick={() => onAdd('new')}>Add</button>;
});
```

**Performance implications:**
- useCallback itself has overhead
- Only beneficial if preventing expensive re-renders
- Measure before optimizing

---

## useMemo

Memoizes an expensive value.

**What it does:**
- Caches computation result
- Re-computes only when dependencies change
- Returns the same reference if dependencies unchanged

**When to use:**
- Expensive computations
- Object/array passed as dependency to other hooks
- Object/array passed to memoized child components

**Common mistakes:**
```javascript
// ❌ Memoizing simple computation
const doubled = useMemo(() => count * 2, [count]); // Not needed

// ✅ Use for expensive operations
const filteredAndSorted = useMemo(() => {
  return largeArray
    .filter(item => item.matches(criteria))
    .sort((a, b) => a.name.localeCompare(b.name));
}, [largeArray, criteria]);

// ❌ Memoizing objects that don't need it
const person = useMemo(() => ({ name: 'Alice' }), []);

// ✅ Memoize if passed to memoized child or effects
const person = useMemo(() => ({ name: 'Alice' }), []);
return <MemoizedChild person={person} />;
```

**Example:**
```javascript
function FilteredList({ items, filter }) {
  // Without memoization, this computes on every render
  const filtered = useMemo(() => {
    console.log('Computing...');
    return items.filter(item => item.name.includes(filter));
  }, [items, filter]); // Re-compute only when items or filter change
  
  return (
    <ul>
      {filtered.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
}
```

**Performance implications:**
- useMemo itself has overhead
- Use for genuinely expensive operations
- Benchmark before assuming it helps

---

## useRef

Creates a persistent reference to a value.

**What it does:**
- Returns an object with `.current` property
- Doesn't cause re-render when changed
- Persists across renders
- Mutable (unlike state)

**When to use:**
- Accessing DOM elements directly
- Storing mutable value that doesn't trigger re-render
- Keeping track of previous value
- Managing focus, selection, or media playback

**Common mistakes:**
```javascript
// ❌ Using ref for state
const count = useRef(0);
count.current++; // Updates but doesn't re-render

// ✅ Use state for values that should cause re-render
const [count, setCount] = useState(0);

// ❌ Setting ref in render
function BadComponent() {
  const ref = useRef(null);
  ref.current = document.getElementById('input'); // DON'T DO THIS
}

// ✅ Use in effects
function GoodComponent() {
  const ref = useRef(null);
  useEffect(() => {
    // ref.current is set by JSX, safe to use here
  }, []);
  return <input ref={ref} />;
}
```

**Example:**
```javascript
function TextInput() {
  const inputRef = useRef(null);
  
  const handleFocus = () => {
    inputRef.current?.focus();
  };
  
  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleFocus}>Focus Input</button>
    </>
  );
}
```

**Performance implications:**
- No performance cost (doesn't cause re-renders)
- Use sparingly (prefer uncontrolled components)

---

## useLayoutEffect

Synchronous version of useEffect (runs before paint).

**What it does:**
- Runs after DOM mutations but before browser paint
- Synchronous (blocks rendering)
- Has same signature as useEffect

**When to use:**
- Measuring DOM elements
- Synchronous DOM mutations
- Preventing visual flicker

**Common mistakes:**
```javascript
// ❌ Using for data fetching
useLayoutEffect(() => {
  fetch('/api/data').then(setData); // Blocks rendering!
}, []);

// ✅ Use for DOM measurements
useLayoutEffect(() => {
  const height = ref.current.offsetHeight;
  setHeight(height);
}, []);
```

**Example:**
```javascript
function MeasuredComponent() {
  const [height, setHeight] = useState(0);
  const ref = useRef(null);
  
  useLayoutEffect(() => {
    // Measure immediately before paint to prevent flicker
    setHeight(ref.current.offsetHeight);
  }, []);
  
  return <div ref={ref}>Content</div>;
}
```

**Performance implications:**
- Can block rendering if expensive
- Use sparingly (useEffect is usually better)
- Most use cases should use useEffect

---

## Custom Hooks

Create reusable logic by extracting hooks.

**What they do:**
- JavaScript functions that call other hooks
- Must follow hook rules
- Reuse stateful logic between components

**When to use:**
- Logic repeated in multiple components
- Complex side effect logic
- Logic related to other hooks

**Example:**
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
  
  return (
    <select value={theme} onChange={(e) => setTheme(e.target.value)}>
      <option>light</option>
      <option>dark</option>
    </select>
  );
}
```

**Performance implications:**
- Custom hooks have same performance as built-in hooks
- Reuse logic without performance penalty
- Good for organization and testing

---

## Hook Rules

1. **Only call hooks at the top level**
   - Not inside loops, conditions, or nested functions
   - Must be unconditional

2. **Only call hooks from React components or custom hooks**
   - Not from regular JavaScript functions
   - Not from event handlers (use them, don't call them from handlers)

3. **Custom hook names must start with "use"**
   - Follows naming convention
   - Helps React recognize it's a hook

---

## See Also

- [SKILL.md](./SKILL.md) - Detailed explanations of hooks
- [EXAMPLES.md](./EXAMPLES.md) - Real-world hook usage
- [ANTI-PATTERNS.md](./ANTI-PATTERNS.md) - Common hook mistakes
