# React State Management Guide

Deciding where state should live is one of the most important decisions in React development.

## State Management Decision Framework

Use this flowchart to determine where each piece of state should live:

```
Does this data ever change?
│
├─ NO → It's configuration/constant
│       └─ Store it outside React (constants, config file)
│
└─ YES → It changes during user interaction
    │
    Does only ONE component use this data?
    │
    ├─ YES → Store state in that component (local state)
    │        Use: useState, useReducer
    │
    └─ NO → Multiple components need this data
        │
        Are the components SIBLINGS or PARENT/CHILD?
        │
        ├─ SIBLINGS or DEEPLY NESTED
        │  │
        │  How deep is the tree? (how many levels)?
        │  │
        │  ├─ 1-2 levels → Use props (prop drilling is fine)
        │  │              Parent holds state, passes to children via props
        │  │              Children update via callbacks
        │  │
        │  └─ 3+ levels → Use Context API or state management
        │                 Extract state to parent, use context
        │
        └─ Is this the ROOT level? (entire app needs it)
            │
            └─ YES → Use Context API or state management library
                     Data: auth, theme, language, notifications
                     Tools: Context, Redux, Zustand, Jotai
```

## Local Component State

**When to use:**
- Data used by only one component
- UI state (open/closed, expanded/collapsed, focus)
- Form input as user is typing
- Temporary data

**Examples:**

```javascript
// UI state - stays in component
function Accordion({ title, content }) {
  const [isOpen, setIsOpen] = useState(false); // ✅ Local
  
  return (
    <>
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? '▼' : '▶'} {title}
      </button>
      {isOpen && <div>{content}</div>}
    </>
  );
}

// Form input - stays in component while typing
function SearchInput() {
  const [value, setValue] = useState(""); // ✅ Local
  const [results, setResults] = useState([]); // ✅ Local
  
  const handleChange = (e) => {
    setValue(e.target.value);
    // fetch and update results
  };
  
  return (
    <>
      <input value={value} onChange={handleChange} />
      {results.map(r => <Result key={r.id} result={r} />)}
    </>
  );
}

// Temporary data
function ImageUpload() {
  const [preview, setPreview] = useState(null); // ✅ Local
  const [uploading, setUploading] = useState(false); // ✅ Local
  
  return (
    <>
      {preview && <img src={preview} alt="preview" />}
      <button disabled={uploading}>
        {uploading ? "Uploading..." : "Upload"}
      </button>
    </>
  );
}
```

**When NOT to use:**
- Data needed by multiple components
- Data that should persist across navigation
- Global settings or configuration
- Data that affects UI globally

## Shared State Between Siblings

**Scenario:** Two siblings need access to same state

**Solution:** Lift state to parent

```javascript
// ❌ WRONG: State in separate components
function BadSiblings() {
  return (
    <>
      <UserProfile />
      <UserPosts />
    </>
  );
}

function UserProfile() {
  const [user, setUser] = useState(null); // Separate state
  // ...
}

function UserPosts() {
  const [user, setUser] = useState(null); // Another copy - gets out of sync
  // ...
}

// ✅ CORRECT: Lift state to parent
function GoodSiblings({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data.user);
        setPosts(data.posts);
      });
  }, [userId]);
  
  return (
    <>
      <UserProfile user={user} onUpdateUser={setUser} />
      <UserPosts posts={posts} user={user} />
    </>
  );
}

function UserProfile({ user, onUpdateUser }) {
  // Use passed props and callback
  return <div>{user?.name}</div>;
}

function UserPosts({ posts, user }) {
  // Use passed props
  return <ul>{posts.map(p => <li key={p.id}>{p.title}</li>)}</ul>;
}
```

## Deep Component Tree State (3+ Levels)

**Scenario:** State needed deep in component tree, not just immediate children

**Solutions:**

### Option 1: Short Prop Chain (1-2 levels)

```javascript
// ✅ Fine for shallow trees
function Parent() {
  const [filter, setFilter] = useState("all");
  return <Child filter={filter} onChange={setFilter} />;
}

function Child({ filter, onChange }) {
  return <GrandChild filter={filter} onChange={onChange} />;
}

function GrandChild({ filter, onChange }) {
  // Uses the data
  return <select value={filter} onChange={(e) => onChange(e.target.value)} />;
}
```

### Option 2: Context API (3+ levels, moderate frequency)

```javascript
const FilterContext = createContext();

export function FilterProvider({ children }) {
  const [filter, setFilter] = useState("all");
  const value = useMemo(() => ({ filter, setFilter }), [filter]);
  
  return (
    <FilterContext.Provider value={value}>
      {children}
    </FilterContext.Provider>
  );
}

export function useFilter() {
  return useContext(FilterContext);
}

// Usage
function App() {
  return (
    <FilterProvider>
      <Level1 />
    </FilterProvider>
  );
}

function Level5() {
  const { filter, setFilter } = useFilter();
  // Use data directly without prop drilling
  return <select value={filter} onChange={(e) => setFilter(e.target.value)} />;
}
```

### Option 3: State Management Library (Complex, frequent changes)

```javascript
// Using Zustand or Redux
import { useStore } from './store';

function Level5() {
  const filter = useStore(state => state.filter);
  const setFilter = useStore(state => state.setFilter);
  
  return <select value={filter} onChange={(e) => setFilter(e.target.value)} />;
}
```

**Decision:**

- 1-2 levels deep → Use props
- 3-5 components → Context API
- Complex state transitions → State management library
- Entire app needs it → Context API + possibly library

## Form State Management

**For simple forms:**

```javascript
function SimpleForm() {
  const [form, setForm] = useState({
    name: "",
    email: "",
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm(prev => ({ ...prev, [name]: value }));
  };
  
  return (
    <>
      <input name="name" value={form.name} onChange={handleChange} />
      <input name="email" value={form.email} onChange={handleChange} />
    </>
  );
}
```

**For complex forms with validation:**

```javascript
function ComplexForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);
  
  const handleChange = (e) => {
    dispatch({ type: "FIELD_CHANGE", payload: e.target });
  };
  
  const handleBlur = (e) => {
    dispatch({ type: "FIELD_BLUR", payload: e.target.name });
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    dispatch({ type: "SUBMIT_START" });
    try {
      await submitForm(state.fields);
      dispatch({ type: "SUBMIT_SUCCESS" });
    } catch (error) {
      dispatch({ type: "SUBMIT_ERROR", payload: error });
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
    </form>
  );
}
```

**For multi-page forms:**

Use Context API to maintain state across pages:

```javascript
const FormContext = createContext();

export function FormProvider({ children }) {
  const [page, setPage] = useState(1);
  const [form, setForm] = useState({
    page1: {},
    page2: {},
  });
  
  return (
    <FormContext.Provider value={{ page, setPage, form, setForm }}>
      {children}
    </FormContext.Provider>
  );
}

function WizardForm() {
  const { page, setPage, form, setForm } = useContext(FormContext);
  
  return (
    <>
      {page === 1 && <Page1 />}
      {page === 2 && <Page2 />}
      <button onClick={() => setPage(page - 1)} disabled={page === 1}>Back</button>
      <button onClick={() => setPage(page + 1)} disabled={page === 2}>Next</button>
    </>
  );
}
```

## Server State Management

**Important:** Server state is different from client state.

**Server state characteristics:**
- Fetched from server
- Not owned by client
- Can become stale
- Needs cache invalidation
- Can be updated by other users

**For simple data fetching:**

```javascript
function UserProfile({ userId }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(error => {
        setError(error);
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{data.name}</div>;
}
```

**For complex server state:**

Use a data-fetching library (React Query, SWR):

```javascript
// Using React Query (better than useState for server state)
import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }) {
  const { data, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(res => res.json()),
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{data.name}</div>;
}
```

**Why use a library for server state:**
- Automatic caching
- Stale-while-revalidate strategy
- Automatic refetching
- Background updates
- Offline support
- Better devtools

## State Organization by Type

```
┌─ CLIENT STATE
│  ├─ UI State (local to component)
│  │  └─ isMenuOpen, isLoading, focusedField
│  │
│  ├─ Form State (component or context)
│  │  └─ formData, errors, touched fields
│  │
│  └─ Global UI State (context)
│     └─ theme, language, notifications
│
├─ SERVER STATE (use data-fetching library)
│  ├─ Cache from queries
│  ├─ Mutations
│  └─ Optimistic updates
│
└─ PERSISTENT STATE
   ├─ User preferences (localStorage)
   ├─ Authentication tokens (localStorage/sessionStorage)
   └─ Session data (URL, cookies)
```

## Complete Decision Matrix

```
┌─────────────────┬──────────────────┬──────────────────┬──────────────────┐
│ Data Type       │ Used By          │ Change Frequency │ Best Location    │
├─────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ UI state        │ One component    │ On user action   │ useState in comp  │
│ Form input      │ One component    │ Every keystroke  │ useState in comp  │
│ Form data       │ Related comps    │ On submission    │ Props + callback │
│ List filter     │ Container + list │ Occasional       │ Props or context │
│ App theme       │ Entire app       │ Rare             │ Context API      │
│ Auth data       │ Most components  │ On login/logout  │ Context API      │
│ Notifications   │ Entire app       │ Frequent         │ Context API      │
│ Server data     │ Multiple comps   │ Fetch interval   │ React Query/SWR  │
│ Form validation │ One form         │ On change/blur   │ useReducer       │
│ Modal open      │ One modal        │ Toggle           │ useState in comp  │
└─────────────────┴──────────────────┴──────────────────┴──────────────────┘
```

## Anti-Patterns

**❌ Don't:**
- Keep all state in one context
- Store server data in context without invalidation strategy
- Pass all props through every component
- Use state for computed values
- Lift state higher than needed
- Create new objects in dependency arrays

**✅ Do:**
- Split contexts by concern
- Use data-fetching libraries for server state
- Use props for local data flow
- Derive computed values from source state
- Keep state as close as possible to where it's used
- Memoize objects in dependency arrays

---

## See Also

- [SKILL.md](./SKILL.md) - State and lifecycle fundamentals
- [EXAMPLES.md](./EXAMPLES.md) - Working state management examples
- [COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md) - Managing state in composed components
