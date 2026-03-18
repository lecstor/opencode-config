# React Anti-Patterns: What NOT to Do

This document highlights common mistakes and their solutions.

## 1. Mutating State Directly

### The Problem

React relies on reference equality to detect state changes. When you mutate state directly, the reference stays the same, so React doesn't detect the change.

```javascript
// ❌ WRONG: Direct mutation
function BadCounter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    count++; // ❌ Mutating primitive (doesn't work)
    setCount(count);
  };
  
  return <div>{count}</div>;
}

// ❌ WRONG: Mutating object
function BadUser() {
  const [user, setUser] = useState({ name: "Alice", age: 30 });
  
  const celebrateBirthday = () => {
    user.age++; // ❌ Mutating object property
    setUser(user); // React doesn't see the change!
  };
  
  return <div>{user.name} is {user.age}</div>;
}

// ❌ WRONG: Mutating array
function BadList() {
  const [items, setItems] = useState(["a", "b"]);
  
  const addItem = () => {
    items.push("c"); // ❌ Mutating array
    setItems(items); // React doesn't see the change!
  };
  
  return <ul>{items.map(i => <li key={i}>{i}</li>)}</ul>;
}
```

### Why It's Wrong

- React uses `Object.is()` to check if state changed
- When you mutate, the reference is identical, so React skips the update
- Your component won't re-render
- Leads to subtle, hard-to-debug bugs
- Makes it harder to track state changes

### The Solution

Always create new objects/arrays when updating state:

```javascript
// ✅ CORRECT: Using updater function
function GoodCounter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    setCount(prevCount => prevCount + 1); // Creates new value
  };
  
  return <div>{count}</div>;
}

// ✅ CORRECT: Spread operator for objects
function GoodUser() {
  const [user, setUser] = useState({ name: "Alice", age: 30 });
  
  const celebrateBirthday = () => {
    setUser({ ...user, age: user.age + 1 }); // New object
  };
  
  return <div>{user.name} is {user.age}</div>;
}

// ✅ CORRECT: Array methods that create new arrays
function GoodList() {
  const [items, setItems] = useState(["a", "b"]);
  
  const addItem = () => {
    setItems([...items, "c"]); // New array
  };
  
  const removeItem = (index) => {
    setItems(items.filter((_, i) => i !== index)); // New array
  };
  
  const updateItem = (index, value) => {
    setItems(items.map((item, i) => i === index ? value : item)); // New array
  };
  
  return <ul>{items.map((i, idx) => <li key={idx}>{i}</li>)}</ul>;
}

// ✅ CORRECT: Nested objects
function GoodForm() {
  const [form, setForm] = useState({
    personal: { name: "", email: "" },
    address: { street: "", city: "" }
  });
  
  const updateName = (name) => {
    setForm({
      ...form,
      personal: { ...form.personal, name }
    });
  };
  
  return <input value={form.personal.name} onChange={(e) => updateName(e.target.value)} />;
}
```

## 2. Missing useEffect Dependencies

### The Problem

Incomplete dependency arrays cause the effect to use stale values, leading to bugs that are hard to track.

```javascript
// ❌ WRONG: Missing dependency
function BadUserDetail({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, []); // ❌ Missing userId! Won't re-fetch when userId changes
  
  return <div>{user?.name}</div>;
}

// ❌ WRONG: Using stale value in effect
function BadSearchForm() {
  const [query, setQuery] = useState("");
  
  const handleSearch = () => {
    // query here is stale from when effect was defined
  };
  
  useEffect(() => {
    window.addEventListener("keydown", handleSearch);
    return () => window.removeEventListener("keydown", handleSearch);
  }, []); // ❌ handleSearch dependency missing, effect uses stale handleSearch
}
```

### Why It's Wrong

- Component has stale data from previous renders
- User updates prop, but effect doesn't run
- Causes race conditions and data inconsistency
- Hard to debug because there are no error messages
- User sees outdated or incorrect information

### The Solution

Include all values from outer scope that the effect uses:

```javascript
// ✅ CORRECT: Include userId dependency
function GoodUserDetail({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, [userId]); // ✅ Includes userId - re-fetches when it changes
  
  return <div>{user?.name}</div>;
}

// ✅ CORRECT: Use useCallback with dependencies
function GoodSearchForm() {
  const [query, setQuery] = useState("");
  
  const handleSearch = useCallback(() => {
    // This always has access to current query
    console.log("Searching for:", query);
  }, [query]); // ✅ Dependency included
  
  useEffect(() => {
    window.addEventListener("keydown", handleSearch);
    return () => window.removeEventListener("keydown", handleSearch);
  }, [handleSearch]); // ✅ Dependency included
}

// ✅ CORRECT: Use function form of setState to avoid dependencies
function GoodCounter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(prev => prev + 1); // No dependency on count needed!
    }, 1000);
    return () => clearInterval(timer);
  }, []); // Empty array is safe here
}

// ✅ CORRECT: Memoize complex dependencies
function GoodUserSearch({ userId, options }) {
  const [results, setResults] = useState([]);
  
  const memoizedOptions = useMemo(() => options, [options.sort, options.limit]); // Only recreate if these change
  
  useEffect(() => {
    fetchResults(userId, memoizedOptions).then(setResults);
  }, [userId, memoizedOptions]); // Dependencies are stable
}
```

## 3. Creating Functions in Render

### The Problem

Creating new function instances on every render causes unnecessary re-renders of memoized children and breaks callback dependencies.

```javascript
// ❌ WRONG: Function created every render
function BadChildProvider() {
  const [items, setItems] = useState([]);
  
  return (
    <MemoizedChild
      items={items}
      onAdd={() => setItems([...items, Math.random()])} // ❌ New function every render!
    />
  );
}

// ❌ WRONG: Handler in JSX
function BadForm() {
  const [email, setEmail] = useState("");
  
  return (
    <input
      value={email}
      onChange={(e) => {
        // ❌ This function is created on every render
        setEmail(e.target.value);
        logAnalytics("email_changed");
      }}
    />
  );
}

// ❌ WRONG: Effect dependency
function BadTimer() {
  const [time, setTime] = useState(0);
  
  const tick = () => setTime(t => t + 1); // ❌ New function every render
  
  useEffect(() => {
    const interval = setInterval(tick, 1000);
    return () => clearInterval(interval);
  }, [tick]); // ❌ tick changes every render, effect re-runs constantly!
}
```

### Why It's Wrong

- Every instance is a new object in memory
- `React.memo` can't optimize children because props.onAdd always changes
- If function is in dependency array, effect re-runs unnecessarily
- Performance degradation, especially in lists
- Confusing debugging experience

### The Solution

Use `useCallback` to memoize functions:

```javascript
// ✅ CORRECT: useCallback memoizes function
function GoodChildProvider() {
  const [items, setItems] = useState([]);
  
  const handleAdd = useCallback(() => {
    setItems(prev => [...prev, Math.random()]);
  }, []); // ✅ Function reference stays the same
  
  return (
    <MemoizedChild
      items={items}
      onAdd={handleAdd}
    />
  );
}

// ✅ CORRECT: Extract handler
function GoodForm() {
  const [email, setEmail] = useState("");
  
  const handleEmailChange = useCallback((e) => {
    setEmail(e.target.value);
    logAnalytics("email_changed");
  }, []);
  
  return <input value={email} onChange={handleEmailChange} />;
}

// ✅ CORRECT: useCallback in effect
function GoodTimer() {
  const [time, setTime] = useState(0);
  
  const tick = useCallback(() => setTime(t => t + 1), []);
  
  useEffect(() => {
    const interval = setInterval(tick, 1000);
    return () => clearInterval(interval);
  }, [tick]); // ✅ tick is stable, effect runs once
}

// ✅ CORRECT: When dependencies require the function
function GoodDataFetcher({ userId, onSuccess }) {
  const [data, setData] = useState(null);
  
  const handleFetchSuccess = useCallback((data) => {
    setData(data);
    onSuccess(data); // If onSuccess is from parent, it must be in dependencies
  }, [onSuccess]);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(handleFetchSuccess);
  }, [userId, handleFetchSuccess]); // ✅ Both included
}
```

## 4. Prop Drilling Hell

### The Problem

Passing props through many intermediate components makes the code hard to follow and refactor.

```javascript
// ❌ WRONG: Props drilled 5+ levels
function App() {
  const [user, setUser] = useState({ name: "Alice", theme: "dark" });
  return <Level1 user={user} theme={user.theme} />;
}

function Level1({ user, theme }) {
  return <Level2 user={user} theme={theme} />;
}

function Level2({ user, theme }) {
  return <Level3 user={user} theme={theme} />;
}

function Level3({ user, theme }) {
  return <Level4 user={user} theme={theme} />;
}

function Level4({ user, theme }) {
  return <Level5 user={user} theme={theme} />;
}

function Level5({ user, theme }) {
  // ❌ Finally uses the props, but had to pass through 4 components
  return <div className={theme}>{user.name}</div>;
}
```

### Why It's Wrong

- Intermediate components become tightly coupled to data flow
- Hard to refactor or move components
- Props leak implementation details to components that don't use them
- Makes it unclear which props matter to which components
- Difficult to test intermediate components in isolation

### The Solution

Use Context API for deeply nested data:

```javascript
// ✅ CORRECT: Use context for global-ish data
const UserContext = createContext();
const ThemeContext = createContext();

function App() {
  const [user, setUser] = useState({ name: "Alice", theme: "dark" });
  
  return (
    <UserContext.Provider value={user}>
      <ThemeContext.Provider value={user.theme}>
        <Level1 /> {/* No props needed! */}
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// Level2, 3, 4 don't need to know about props
function Level1() {
  return <Level2 />;
}

function Level2() {
  return <Level3 />;
}

function Level3() {
  return <Level4 />;
}

function Level4() {
  return <Level5 />;
}

function Level5() {
  const user = useContext(UserContext);
  const theme = useContext(ThemeContext);
  
  return <div className={theme}>{user.name}</div>;
}

// ✅ CORRECT: Keep short prop chains (1-2 levels) as props
// Props are fine when the chain is shallow
function Parent() {
  const data = useSomething();
  return <Child data={data} />;
}

function Child({ data }) {
  return <GrandChild data={data} />;
}

function GrandChild({ data }) {
  // ✅ Only 2 levels, this is acceptable
  return <div>{data.value}</div>;
}
```

## 5. Context Overuse

### The Problem

Putting too much data in context or data that changes frequently causes unnecessary re-renders of all consumers.

```javascript
// ❌ WRONG: Everything in one context
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState("light");
  const [notifications, setNotifications] = useState([]);
  const [loading, setLoading] = useState(false);
  const [formData, setFormData] = useState({});
  
  return (
    <AppContext.Provider value={{
      user, setUser,
      theme, setTheme,
      notifications, setNotifications,
      loading, setLoading,
      formData, setFormData
    }}>
      {children}
    </AppContext.Provider>
  );
}

// Every change to ANY value causes ALL consumers to re-render
function Consumer() {
  const context = useContext(AppContext);
  // Re-renders when user, theme, notifications, or anything else changes
  return <div>{context.user?.name}</div>;
}
```

### Why It's Wrong

- All consumers re-render when ANY value in context changes
- Even if component only uses one value, it re-renders for changes to others
- `React.memo` can't help because the value object changes
- Performance degrades significantly with large app state
- Can become a performance bottleneck

### The Solution

Split context by concern:

```javascript
// ✅ CORRECT: Separate contexts for separate concerns
const UserContext = createContext();
const ThemeContext = createContext();
const NotificationContext = createContext();

export function AppProvider({ children }) {
  return (
    <UserProvider>
      <ThemeProvider>
        <NotificationProvider>
          {children}
        </NotificationProvider>
      </ThemeProvider>
    </UserProvider>
  );
}

function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const value = useMemo(() => ({ user, setUser }), [user]);
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

// ✅ CORRECT: Split state and dispatch for even better optimization
const ThemeStateContext = createContext();
const ThemeDispatchContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");
  
  return (
    <ThemeStateContext.Provider value={theme}>
      <ThemeDispatchContext.Provider value={setTheme}>
        {children}
      </ThemeDispatchContext.Provider>
    </ThemeStateContext.Provider>
  );
}

function useTheme() {
  return useContext(ThemeStateContext);
}

function useSetTheme() {
  return useContext(ThemeDispatchContext);
}

// ✅ CORRECT: Only subscribe to what you need
function ThemeButton() {
  const setTheme = useSetTheme(); // Only re-renders if dispatch changes (never)
  return <button onClick={() => setTheme(prev => prev === "light" ? "dark" : "light")}>;
}

function ThemeDisplay() {
  const theme = useTheme(); // Only re-renders when theme changes
  return <div className={theme}>Content</div>;
}
```

## 6. State Bloat

### The Problem

Keeping too much data in component state makes it hard to manage and causes unnecessary re-renders.

```javascript
// ❌ WRONG: Too much state
function Dashboard() {
  const [users, setUsers] = useState([]);
  const [filteredUsers, setFilteredUsers] = useState([]);
  const [filter, setFilter] = useState("");
  const [sortBy, setSortBy] = useState("name");
  const [sortedAndFiltered, setSortedAndFiltered] = useState([]);
  const [selectedUser, setSelectedUser] = useState(null);
  const [selectedUserDetails, setSelectedUserDetails] = useState(null);
  const [isLoadingUsers, setIsLoadingUsers] = useState(false);
  const [isLoadingDetails, setIsLoadingDetails] = useState(false);
  const [usersError, setUsersError] = useState(null);
  const [detailsError, setDetailsError] = useState(null);
  
  // Too many state variables - hard to keep in sync
}
```

### Why It's Wrong

- Hard to keep derived state in sync with source state
- Easy to get into inconsistent state
- Too many re-renders as unrelated state changes
- Difficult to understand what state matters
- Testing is complex

### The Solution

Derive state instead of storing it:

```javascript
// ✅ CORRECT: Only store source data and UI state
function Dashboard() {
  // Source data
  const [users, setUsers] = useState([]);
  const [selectedUserId, setSelectedUserId] = useState(null);
  
  // Loading and error states
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  // UI filters
  const [filter, setFilter] = useState("");
  const [sortBy, setSortBy] = useState("name");
  
  // Derive filtered/sorted data instead of storing it
  const filteredAndSorted = useMemo(() => {
    return users
      .filter(user => user.name.toLowerCase().includes(filter.toLowerCase()))
      .sort((a, b) => {
        if (sortBy === "name") return a.name.localeCompare(b.name);
        if (sortBy === "email") return a.email.localeCompare(b.email);
        return 0;
      });
  }, [users, filter, sortBy]);
  
  // Derive selected user details instead of storing
  const selectedUser = useMemo(() => {
    return users.find(u => u.id === selectedUserId);
  }, [users, selectedUserId]);
  
  // Single loading state for the whole component
  // If you need separate loading states, use a reducer
  
  return (
    <>
      <Filters filter={filter} onFilterChange={setFilter} sortBy={sortBy} onSortChange={setSortBy} />
      <UserList users={filteredAndSorted} onSelectUser={setSelectedUserId} />
      {selectedUser && <UserDetail user={selectedUser} />}
    </>
  );
}

// ✅ CORRECT: Use reducer for complex state relationships
const dashboardReducer = (state, action) => {
  switch (action.type) {
    case "LOAD_START":
      return { ...state, isLoading: true, error: null };
    case "LOAD_SUCCESS":
      return { ...state, isLoading: false, users: action.payload };
    case "LOAD_ERROR":
      return { ...state, isLoading: false, error: action.payload };
    case "SET_FILTER":
      return { ...state, filter: action.payload };
    case "SET_SORT":
      return { ...state, sortBy: action.payload };
    case "SELECT_USER":
      return { ...state, selectedUserId: action.payload };
    default:
      return state;
  }
};

function GoodDashboard() {
  const [state, dispatch] = useReducer(dashboardReducer, {
    users: [],
    selectedUserId: null,
    isLoading: false,
    error: null,
    filter: "",
    sortBy: "name"
  });
  
  // Much clearer state management
}
```

## 7. useEffect as Constructor

### The Problem

Using useEffect like it's a class constructor lifecycle method, trying to replicate `componentDidMount` + `componentWillUnmount` patterns.

```javascript
// ❌ WRONG: useEffect used like constructor
function BadComponent({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  
  useEffect(() => {
    // This runs on EVERY render because no dependency array
    // Like having no constructor, just running code constantly
    setUser(null);
    setPosts([]);
    
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data.user);
        setPosts(data.posts);
      });
  }); // ❌ No dependency array - runs on every render!
}
```

### Why It's Wrong

- Effect runs on every single render
- Network requests are made constantly
- State is reset on every render
- Performance nightmare
- Misunderstands React's mental model

### The Solution

Use appropriate dependencies:

```javascript
// ✅ CORRECT: Only fetch when userId changes
function GoodComponent({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  
  useEffect(() => {
    // Only runs when userId changes
    const fetchData = async () => {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      setUser(data.user);
      setPosts(data.posts);
    };
    
    fetchData();
  }, [userId]); // ✅ Dependency array specifies when to re-run
  
  return <div>{user?.name}</div>;
}

// ✅ CORRECT: Run once on mount
function GoodMountComponent() {
  const [config, setConfig] = useState(null);
  
  useEffect(() => {
    // Runs once on mount
    fetch("/api/config")
      .then(res => res.json())
      .then(setConfig);
  }, []); // ✅ Empty array - runs once
  
  return <div>{config?.name}</div>;
}

// ✅ CORRECT: Think in terms of sync, not lifecycle
function GoodSyncComponent({ source, destination }) {
  useEffect(() => {
    // Sync destination whenever source changes
    destination.update(source);
  }, [source, destination]); // Depends on what to sync
}
```

## 8. Infinite Loops

### The Problem

Incorrect dependencies or missing cleanup create infinite loops.

```javascript
// ❌ WRONG: Object dependency without memoization
function BadInfiniteLoop() {
  const [data, setData] = useState(null);
  const options = { timeout: 5000 }; // ❌ New object every render
  
  useEffect(() => {
    fetchData(options).then(setData);
  }, [options]); // ❌ options changes every render - infinite loop!
}

// ❌ WRONG: Updating same dependency
function BadDependencyCycle() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setCount(count + 1); // ❌ Updates count
  }, [count]); // ❌ count change triggers effect again - infinite loop!
}

// ❌ WRONG: Missing cleanup
function BadEventListener() {
  const [events, setEvents] = useState([]);
  
  useEffect(() => {
    const handler = () => setEvents(prev => [...prev, 'event']);
    
    for (let i = 0; i < 100; i++) {
      document.addEventListener('click', handler); // ❌ Adds hundreds of listeners
    }
    // ❌ No cleanup - listeners accumulate
  }, []);
}
```

### Why It's Wrong

- Component is frozen, unresponsive
- CPU usage spikes to 100%
- Browser may crash or freeze
- Makes development impossible

### The Solution

Fix dependencies and cleanup:

```javascript
// ✅ CORRECT: Memoize object dependency
function GoodWithObjectDependency() {
  const [data, setData] = useState(null);
  const options = useMemo(() => ({ timeout: 5000 }), []); // Memoized - stable reference
  
  useEffect(() => {
    fetchData(options).then(setData);
  }, [options]); // Now options is stable
}

// ✅ CORRECT: Use function form of setState
function GoodWithoutDependency() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setCount(prevCount => prevCount + 1); // ✅ Doesn't depend on count
  }, []); // Empty dependency - runs once
}

// ✅ CORRECT: Proper cleanup
function GoodEventListener() {
  const [events, setEvents] = useState([]);
  
  useEffect(() => {
    const handler = () => setEvents(prev => [...prev, 'event']);
    
    document.addEventListener('click', handler);
    
    return () => {
      document.removeEventListener('click', handler); // ✅ Cleanup
    };
  }, []);
}

// ✅ CORRECT: Don't update dependency in the same effect
function GoodEffectWithCleanup() {
  const [time, setTime] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setTime(prev => prev + 1); // ✅ Function form
    }, 1000);
    
    return () => clearInterval(timer); // ✅ Cleanup
  }, []); // No dependency on time
}
```

## 9. Switching Between Controlled and Uncontrolled

### The Problem

Switching a component between controlled (value from prop) and uncontrolled (own state) causes errors.

```javascript
// ❌ WRONG: Sometimes controlled, sometimes uncontrolled
function BadInput({ value }) {
  const [internalValue, setInternalValue] = useState("");
  
  // If value is provided, use it (controlled)
  // If not, use internal state (uncontrolled)
  const isControlled = value !== undefined;
  const displayValue = isControlled ? value : internalValue;
  
  const handleChange = (e) => {
    // ❌ This doesn't work right - props change don't update internalValue
    setInternalValue(e.target.value);
  };
  
  return <input value={displayValue} onChange={handleChange} />;
}

// Usage that switches between controlled and uncontrolled
<BadInput /> {/* Uncontrolled */}
<BadInput value={someValue} /> {/* Controlled */}
```

### Why It's Wrong

- React warns about this pattern
- Switching causes the input value to get "stuck"
- User input might be ignored or lost
- Very confusing behavior
- Input state gets out of sync with React state

### The Solution

Choose one pattern and stick with it:

```javascript
// ✅ CORRECT: Always controlled
function GoodControlledInput({ value, onChange, defaultValue = "" }) {
  const handleChange = (e) => {
    onChange(e.target.value);
  };
  
  return <input value={value} onChange={handleChange} />;
}

// Always pass value and onChange
<GoodControlledInput value={myValue} onChange={setMyValue} />

// ✅ CORRECT: Always uncontrolled
function GoodUncontrolledInput({ defaultValue = "" }) {
  const inputRef = useRef(null);
  
  const getValue = () => inputRef.current?.value;
  
  return <input ref={inputRef} defaultValue={defaultValue} />;
}

// Use ref to get value when needed
const inputRef = useRef();
<GoodUncontrolledInput ref={inputRef} />
<button onClick={() => console.log(inputRef.current.value)}>Get Value</button>

// ✅ CORRECT: Optional controlled with defaultValue
function GoodFlexibleInput({ value, onChange, defaultValue = "" }) {
  const [internalValue, setInternalValue] = useState(defaultValue);
  const isControlled = value !== undefined;
  
  const displayValue = isControlled ? value : internalValue;
  const handleChange = (e) => {
    const newValue = e.target.value;
    if (!isControlled) {
      setInternalValue(newValue);
    }
    onChange?.(newValue);
  };
  
  useEffect(() => {
    if (!isControlled) {
      setInternalValue(defaultValue);
    }
  }, [defaultValue, isControlled]);
  
  return <input value={displayValue} onChange={handleChange} />;
}
```

## 10. Key Chaos

### The Problem

Using array indexes as keys, missing keys, or non-unique keys causes React to incorrectly map list items.

```javascript
// ❌ WRONG: Index as key (reorderable list)
function BadList({ items }) {
  return (
    <ul>
      {items.map((item, index) => (
        // ❌ When list reorders, key stays with index, not item
        <li key={index}>{item.name}</li>
      ))}
    </ul>
  );
}

// ❌ WRONG: Missing keys
function BadList2({ items }) {
  return (
    <ul>
      {items.map((item) => (
        // ❌ React uses default key (undefined) for all items
        <li>{item.name}</li>
      ))}
    </ul>
  );
}

// ❌ WRONG: Non-unique keys
function BadList3({ items }) {
  return (
    <ul>
      {items.map((item) => (
        // ❌ Multiple items with same key
        <li key={item.category}>{item.name}</li>
      ))}
    </ul>
  );
}
```

### Why It's Wrong

- React uses keys to map old DOM nodes to new ones
- Without correct keys, React re-mounts components
- Component state gets mixed up
- Input focus is lost
- Animation frames get skipped
- List with form inputs loses user input

### The Solution

Use stable, unique keys:

```javascript
// ✅ CORRECT: Unique ID as key
function GoodList({ items }) {
  return (
    <ul>
      {items.map((item) => (
        // ✅ Each item has unique, stable ID
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// ✅ CORRECT: Index OK for static lists
function StaticList({ items }) {
  // If items never reorder, delete, or insert, index is safe
  return (
    <ul>
      {items.map((item, index) => (
        <li key={`${item.id}-${index}`}>{item.name}</li>
      ))}
    </ul>
  );
}

// ✅ CORRECT: Content-based key if no ID
function KeyedByContent({ items }) {
  return (
    <ul>
      {items.map((item) => (
        // Only if name + category combination is always unique
        <li key={`${item.category}-${item.name}`}>{item.name}</li>
      ))}
    </ul>
  );
}

// ✅ CORRECT: When list item has state
function ItemWithState({ item }) {
  const [isExpanded, setIsExpanded] = useState(false);
  // This component's state won't get mixed up if key is stable
  
  return (
    <li>
      <button onClick={() => setIsExpanded(!isExpanded)}>
        {isExpanded ? "▼" : "▶"} {item.name}
      </button>
      {isExpanded && <Details item={item} />}
    </li>
  );
}

function GoodList({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <ItemWithState key={item.id} item={item} /> // ✅ Stable key
      ))}
    </ul>
  );
}
```

---

## Summary

| Anti-Pattern | Symptom | Solution |
|---|---|---|
| Mutating state | Component doesn't re-render | Use spread operator, new arrays |
| Missing dependencies | Stale values in effects | Include all dependencies |
| Functions in render | Unnecessary re-renders | Use `useCallback` |
| Prop drilling | Hard to refactor | Use Context API |
| Context overuse | Poor performance | Split contexts by concern |
| State bloat | Complex state management | Derive instead of storing |
| useEffect as constructor | Infinite loops | Understand dependency arrays |
| Infinite loops | Browser freezes | Fix dependencies and cleanup |
| Uncontrolled/controlled | Input value stuck | Pick one pattern |
| Bad keys | State gets mixed up | Use stable, unique IDs |

---

## See Also

- [SKILL.md](./SKILL.md) - Correct patterns explained
- [EXAMPLES.md](./EXAMPLES.md) - Production-ready examples
- [COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md) - Advanced patterns
