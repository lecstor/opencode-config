---
name: react-best-practices
description: Write React components following idiomatic patterns, hooks best practices, and performance optimizations
---

## Thinking in React (the 5-step process)

1. **Break UI into a component hierarchy** — single responsibility per component
2. **Build a static version first** — props only, no state, no handlers
3. **Identify minimal state** — only data that changes and can't be computed
4. **Identify where state lives** — lowest common ancestor of all consumers
5. **Add inverse data flow** — callbacks from parent, called by children

Data flows down via props. Events flow up via callbacks.

## Component rules

- Functional components only (hooks don't work in class components)
- One component per file, named to match filename
- Keep components small and focused — extract when a component does two things
- Derived values should be computed during render, not stored in state

```js
// ❌ Redundant state
const [todos, setTodos] = useState([])
const [count, setCount] = useState(0) // kept in sync manually

// ✅ Derived value
const [todos, setTodos] = useState([])
const count = todos.length // computed
```

## State rules

- Co-locate state with the component that owns it
- Lift state only when sibling components need to share it
- Initialise with the actual type you'll use ([], {}, '', 0, false)
- Never mutate state directly — always create new objects/arrays

## useEffect rules

- Only for synchronising with external systems (APIs, subscriptions, DOM APIs)
- Not for computing derived data — do that during render
- Always declare every reactive value in the dependency array
- Return a cleanup function when subscribing to anything
- Avoid effects that just set state based on other state — reorganise instead

## Performance (only when you have a measured problem)

- `useMemo` — memoize expensive computations
- `useCallback` — stable function references for child components that use `React.memo`
- `React.memo` — skip re-renders when props haven't changed
- Don't optimise prematurely — these add complexity

## Custom hooks

- Extract repeated stateful logic into `useXxx` functions
- A custom hook can call other hooks; a regular function cannot
- Return an object `{ value, setValue, reset }` for discoverability

## Supporting files

- **[QUICKSTART.md](QUICKSTART.md)** — common patterns ready to copy
- **[HOOKS-REFERENCE.md](HOOKS-REFERENCE.md)** — all hooks with usage examples
- **[COMPOSITION-PATTERNS.md](COMPOSITION-PATTERNS.md)** — render props, compound components, slots
- **[STATE-MANAGEMENT-GUIDE.md](STATE-MANAGEMENT-GUIDE.md)** — when to use local state vs Context vs Zustand
- **[ANTI-PATTERNS.md](ANTI-PATTERNS.md)** — common mistakes and how to fix them
- **[EXAMPLES.md](EXAMPLES.md)** — real-world component examples
