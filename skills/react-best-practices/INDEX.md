# React Best Practices - Topic Index & Learning Roadmap

Complete navigation guide and topic index for the React Best Practices skill.

## Learning Roadmaps

### Beginner Path (New to React)

Week 1 - Fundamentals:
1. **[SKILL.md - Thinking in React](./SKILL.md#thinking-in-react)**
   - 5-step process for building UIs
   - Breaking down into components
   - Identifying state

2. **[SKILL.md - Component Fundamentals](./SKILL.md#component-fundamentals)**
   - Functional components
   - Props
   - Children and composition

3. **[QUICKSTART.md - Template 1](./QUICKSTART.md#1-simple-component-with-state)**
   - Build first component

Week 2 - State & Effects:
1. **[SKILL.md - State & Lifecycle](./SKILL.md#state--lifecycle)**
   - useState basics
   - Multiple states
   - When to use state vs props

2. **[SKILL.md - Side Effects](./SKILL.md#side-effects-with-useeffect)**
   - useEffect fundamentals
   - Dependency arrays (most important!)
   - Cleanup functions

3. **[QUICKSTART.md - Templates 2-3](./QUICKSTART.md#2-component-with-side-effects)**
   - Build components with effects
   - Build form component

Week 3 - Lists & Forms:
1. **[SKILL.md - Forms](./SKILL.md#forms--user-input)**
   - Controlled components
   - Form handling
   - Validation

2. **[SKILL.md - Lists & Keys](./SKILL.md#lists--keys)**
   - Why keys matter
   - Proper key selection
   - List rendering

3. **[QUICKSTART.md - Template 4](./QUICKSTART.md#4-list-with-filtering-and-sorting)**
   - Build filtered list

### Intermediate Path (Comfortable with Basics)

Month 1 - Advanced Patterns:
1. **[COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md)**
   - Composition over configuration
   - Compound components
   - Custom hooks

2. **[STATE-MANAGEMENT-GUIDE.md](./STATE-MANAGEMENT-GUIDE.md)**
   - State placement decisions
   - Context API
   - Deep component trees

3. **[SKILL.md - Custom Hooks](./SKILL.md#custom-hooks)**
   - Creating reusable hooks
   - Hook rules
   - Custom hook patterns

Month 2 - Performance & Optimization:
1. **[SKILL.md - Performance Optimization](./SKILL.md#performance-optimization)**
   - useMemo
   - useCallback
   - React.memo
   - When optimization is premature

2. **[SKILL.md - Context API](./SKILL.md#context-api)**
   - Context splitting
   - Performance implications
   - When NOT to use context

3. **[ANTI-PATTERNS.md](./ANTI-PATTERNS.md)**
   - Common mistakes
   - Why they're wrong
   - Correct solutions

### Advanced Path (Looking to Master)

1. **[SKILL.md - Complete Read](./SKILL.md)**
   - All sections thoroughly
   - Understand the "why"

2. **[EXAMPLES.md - All Examples](./EXAMPLES.md)**
   - Run each example
   - Modify and experiment
   - Understand tradeoffs

3. **[COMPOSITION-PATTERNS.md - Deep Dive](./COMPOSITION-PATTERNS.md)**
   - All patterns
   - Pattern selection
   - Tradeoffs between approaches

4. **[STATE-MANAGEMENT-GUIDE.md - Architecture](./STATE-MANAGEMENT-GUIDE.md)**
   - Complete decision framework
   - Complex state scenarios
   - Multi-feature apps

---

## Topic Index by Subject

### Understanding React's Mental Model

- **[Thinking in React](./SKILL.md#thinking-in-react)** - The 5-step process
  - [Breaking UI into hierarchy](./SKILL.md#step-1-break-the-ui-into-a-component-hierarchy)
  - [Building static version](./SKILL.md#step-2-build-a-static-version-first)
  - [Identifying state](./SKILL.md#step-3-identify-the-minimal-state)
  - [Placing state](./SKILL.md#step-4-identify-where-state-should-live)
  - [Adding data flow](./SKILL.md#step-5-add-inverse-data-flow)
- **[Props Down, Callbacks Up](./COMPOSITION-PATTERNS.md#props-down-callbacks-up)** - The fundamental pattern

### Components

**Fundamentals:**
- **[Functional Components](./SKILL.md#functional-components)** - Modern React components
- **[Props](./SKILL.md#props-receiving-data-from-parents)** - Passing data
- **[Children](./SKILL.md#children-and-composition)** - Composition with children
- **[Prop Defaults](./SKILL.md#default-props-and-prop-types)** - Sensible defaults

**Patterns:**
- **[Component Composition](./COMPOSITION-PATTERNS.md#component-composition-over-configuration)** - Flexible components
- **[Compound Components](./COMPOSITION-PATTERNS.md#compound-components)** - Component families
- **[Render Props](./COMPOSITION-PATTERNS.md#render-props-pattern)** - Function as children
- **[HOCs](./COMPOSITION-PATTERNS.md#higher-order-components-when-to-use)** - Component wrapping
- **[Component Props](./COMPOSITION-PATTERNS.md#passing-components-as-props)** - Maximum flexibility

**Examples:**
- **[Component Hierarchy Example](./EXAMPLES.md#example-1-todo-application-with-proper-component-hierarchy)** - Real structure
- **[E-commerce Example](./EXAMPLES.md#example-2-e-commerce-product-view)** - Complex hierarchy

### State Management

**Fundamentals:**
- **[useState](./SKILL.md#usestate-hook-basics)** - Basic state
- **[State vs Props](./SKILL.md#when-to-use-state-vs-props)** - Decision making
- **[State Placement](./SKILL.md#step-4-identify-where-state-should-live)** - Correct location
- **[Lifting State](./STATE-MANAGEMENT-GUIDE.md#shared-state-between-siblings)** - Moving state up

**Complex State:**
- **[useReducer](./SKILL.md#usereducer-for-complex-state)** - Multiple related values
- **[State Bloat](./ANTI-PATTERNS.md#6-state-bloat)** - Anti-pattern to avoid
- **[Derived State](./STATE-MANAGEMENT-GUIDE.md#state-organization-by-type)** - Computing vs storing

**Decision Making:**
- **[State Management Guide](./STATE-MANAGEMENT-GUIDE.md)** - Complete decision framework
- **[Local Component State](./STATE-MANAGEMENT-GUIDE.md#local-component-state)** - Simple cases
- **[Deep Trees](./STATE-MANAGEMENT-GUIDE.md#deep-component-tree-state-3-levels)** - Props vs context
- **[Form State](./STATE-MANAGEMENT-GUIDE.md#form-state-management)** - Form patterns
- **[Server State](./STATE-MANAGEMENT-GUIDE.md#server-state-management)** - Data fetching

### Hooks

**Core Hooks:**
- **[useState](./HOOKS-REFERENCE.md#usestate)** - State basics
- **[useEffect](./HOOKS-REFERENCE.md#useeffect)** - Side effects
- **[useContext](./HOOKS-REFERENCE.md#usecontext)** - Context values
- **[useReducer](./HOOKS-REFERENCE.md#usereducer)** - Complex state

**Performance Hooks:**
- **[useCallback](./HOOKS-REFERENCE.md#usecallback)** - Function memoization
- **[useMemo](./HOOKS-REFERENCE.md#usememo)** - Value memoization
- **[React.memo](./SKILL.md#reactmemo-for-component-memoization)** - Component memoization

**Other Hooks:**
- **[useRef](./HOOKS-REFERENCE.md#useref)** - DOM access, persistent values
- **[useLayoutEffect](./HOOKS-REFERENCE.md#uselayouteffect)** - Before paint timing

**Custom Hooks:**
- **[Creating Custom Hooks](./SKILL.md#custom-hooks)** - Reusable logic
- **[Hook Rules](./HOOKS-REFERENCE.md#hook-rules)** - Must follow these!
- **[useLocalStorage Example](./EXAMPLES.md#example-1-uselocalstorage)** - Common pattern
- **[useFetch Example](./EXAMPLES.md#example-2-usefetch)** - Data fetching

### Side Effects & Data Fetching

**useEffect Patterns:**
- **[useEffect Fundamentals](./SKILL.md#useeffect-fundamentals)** - Basics
- **[Dependency Arrays](./SKILL.md#dependency-arrays-why-they-matter)** - CRUCIAL!
- **[Cleanup Functions](./SKILL.md#cleanup-functions)** - Preventing leaks
- **[useEffect Patterns](./SKILL.md#common-useeffect-patterns)** - 3 key patterns
- **[useLayoutEffect](./SKILL.md#uselayouteffect-vs-useeffect)** - Timing matters

**Data Fetching:**
- **[Fetching in useEffect](./SKILL.md#fetching-in-useeffect)** - How to do it
- **[Race Conditions](./SKILL.md#race-conditions-and-cleanup)** - AbortController
- **[Loading & Error States](./SKILL.md#loading-and-error-states)** - User feedback
- **[Async Example](./QUICKSTART.md#7-async-data-fetching)** - Working template
- **[Fetching Examples](./EXAMPLES.md#data-fetching)** - Real patterns

**Common Mistakes:**
- **[Missing Dependencies](./ANTI-PATTERNS.md#2-missing-useeffect-dependencies)** - Stale closures
- **[Infinite Loops](./ANTI-PATTERNS.md#8-infinite-loops)** - Wrong dependencies
- **[useEffect as Constructor](./ANTI-PATTERNS.md#7-useeffect-as-constructor)** - Wrong mental model

### Forms

**Fundamentals:**
- **[Controlled vs Uncontrolled](./SKILL.md#controlled-vs-uncontrolled-components)** - Which to use
- **[Form Handling](./SKILL.md#form-handling-patterns)** - Patterns and examples
- **[Input Validation](./SKILL.md#input-validation)** - Validation strategies
- **[Form State](./STATE-MANAGEMENT-GUIDE.md#form-state-management)** - State placement

**Examples:**
- **[Form Template](./QUICKSTART.md#3-form-component-controlled)** - Copy-paste
- **[Contact Form](./EXAMPLES.md#example-1-controlled-form-with-validation)** - Real implementation
- **[Advanced Form](./EXAMPLES.md#form-state-with-validation)** - useReducer approach

**Anti-Patterns:**
- **[Uncontrolled/Controlled](./ANTI-PATTERNS.md#9-switching-between-controlled-and-uncontrolled)** - Never switch!

### Lists

**Fundamentals:**
- **[Lists & Keys](./SKILL.md#lists--keys)** - Why keys matter
- **[Why Keys Matter](./SKILL.md#why-keys-matter)** - React reconciliation
- **[Choosing Keys](./SKILL.md#choosing-good-keys)** - Good vs bad keys
- **[Efficient Lists](./SKILL.md#rendering-lists-efficiently)** - Optimization

**Examples:**
- **[List Template](./QUICKSTART.md#4-list-with-filtering-and-sorting)** - Working example
- **[Filtered List](./EXAMPLES.md#example-1-proper-list-rendering-with-filtering)** - Real pattern

**Anti-Patterns:**
- **[Key Chaos](./ANTI-PATTERNS.md#10-key-chaos)** - Index as key, missing keys

### Context API

**Using Context:**
- **[Creating Context](./SKILL.md#creating-and-using-context)** - Provider pattern
- **[Context vs Props](./SKILL.md#context-vs-props-vs-state-management)** - When to use
- **[Context Splitting](./SKILL.md#context-splitting-for-performance)** - Optimization
- **[When NOT to use](./SKILL.md#when-not-to-use-context)** - Avoid overuse

**Examples:**
- **[Theme Context](./EXAMPLES.md#example-1-theme-context)** - Real implementation
- **[Context Template](./QUICKSTART.md#6-context-setup)** - Copy-paste

**Anti-Patterns:**
- **[Context Overuse](./ANTI-PATTERNS.md#5-context-overuse)** - Too much data

### Performance

**Optimization Techniques:**
- **[useMemo](./SKILL.md#usememo-for-expensive-computations)** - Memoize values
- **[useCallback](./SKILL.md#usecallback-for-stable-function-references)** - Memoize functions
- **[React.memo](./SKILL.md#reactmemo-for-component-memoization)** - Memoize components
- **[When to Optimize](./SKILL.md#when-optimization-is-premature)** - Measure first!

**Anti-Patterns:**
- **[Functions in Render](./ANTI-PATTERNS.md#3-creating-functions-in-render)** - Performance killer
- **[Over-optimization](./ANTI-PATTERNS.md#when-optimization-is-premature)** - Not always needed

### Error Handling

**Fundamentals:**
- **[Error Boundaries](./SKILL.md#error-boundaries)** - What they do
- **[Event Handlers](./SKILL.md#error-handling-in-event-handlers)** - try/catch
- **[Graceful Recovery](./SKILL.md#graceful-error-recovery)** - User feedback
- **[Error Messages](./SKILL.md#user-friendly-error-messages)** - Good UX

**Examples:**
- **[Error Boundary](./EXAMPLES.md#example-1-simple-error-boundary)** - Implementation

### Common Pitfalls

**Essential Reading:**
- **[Anti-Patterns Overview](./ANTI-PATTERNS.md)** - All 10 patterns

**Specific Issues:**
1. **[Mutating State](./ANTI-PATTERNS.md#1-mutating-state-directly)** - Reference equality
2. **[Stale Closures](./ANTI-PATTERNS.md#stale-closures)** - Old values
3. **[Missing Dependencies](./ANTI-PATTERNS.md#2-missing-useeffect-dependencies)** - useEffect issues
4. **[Functions in Render](./ANTI-PATTERNS.md#3-creating-functions-in-render)** - Performance
5. **[Prop Drilling](./ANTI-PATTERNS.md#4-prop-drilling-hell)** - Too many levels
6. **[Context Overuse](./ANTI-PATTERNS.md#5-context-overuse)** - Global state
7. **[State Bloat](./ANTI-PATTERNS.md#6-state-bloat)** - Too much state
8. **[useEffect as Constructor](./ANTI-PATTERNS.md#7-useeffect-as-constructor)** - Wrong model
9. **[Infinite Loops](./ANTI-PATTERNS.md#8-infinite-loops)** - Browser freeze
10. **[Uncontrolled/Controlled](./ANTI-PATTERNS.md#9-switching-between-controlled-and-uncontrolled)** - Form issues

### Code Organization

**Best Practices:**
- **[Code Organization](./SKILL.md#code-organization)** - File structure
- **[Naming Conventions](./SKILL.md#naming-conventions)** - Consistent names
- **[Import/Export](./SKILL.md#importexport-patterns)** - Clean imports

---

## Quick Question Guide

**Q: Where should this state live?**
A: See [STATE-MANAGEMENT-GUIDE.md](./STATE-MANAGEMENT-GUIDE.md) - complete decision framework

**Q: How do I fetch data?**
A: See [SKILL.md - Data Fetching](./SKILL.md#data-fetching-patterns) or [QUICKSTART.md - Template 7](./QUICKSTART.md#7-async-data-fetching)

**Q: How do I structure my components?**
A: See [SKILL.md - Thinking in React](./SKILL.md#thinking-in-react) then [COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md)

**Q: My component re-renders too much**
A: See [ANTI-PATTERNS.md - Performance Issues](./ANTI-PATTERNS.md#3-creating-functions-in-render) or [SKILL.md - Performance](./SKILL.md#performance-optimization)

**Q: useEffect is confusing**
A: See [SKILL.md - useEffect](./SKILL.md#side-effects-with-useeffect) - especially dependency arrays!

**Q: I have too many props**
A: See [ANTI-PATTERNS.md - Prop Drilling](./ANTI-PATTERNS.md#4-prop-drilling-hell) or [COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md)

**Q: My state is getting complicated**
A: See [ANTI-PATTERNS.md - State Bloat](./ANTI-PATTERNS.md#6-state-bloat) or try [SKILL.md - useReducer](./SKILL.md#usereducer-for-complex-state)

**Q: How do I share state between components?**
A: See [STATE-MANAGEMENT-GUIDE.md - Shared State](./STATE-MANAGEMENT-GUIDE.md#shared-state-between-siblings)

**Q: Context is causing too many re-renders**
A: See [ANTI-PATTERNS.md - Context Overuse](./ANTI-PATTERNS.md#5-context-overuse) and [SKILL.md - Context Splitting](./SKILL.md#context-splitting-for-performance)

**Q: How do I write forms?**
A: See [QUICKSTART.md - Template 3](./QUICKSTART.md#3-form-component-controlled) or [EXAMPLES.md - Form Examples](./EXAMPLES.md#forms)

---

## Cross-References

### With React Testable Code Skill

These skills work together:
- This skill teaches **component design**
- React Testable Code teaches **testing those components**

Key overlaps:
- **Custom hooks** - Both teach patterns for reusable logic
- **Component composition** - Affects testability
- **Props vs state** - Impacts test strategy

When you finish a topic here, see React Testable Code skill for how to test it.

### With Playwright Skill

Clean React components are easier to test with E2E tests:
- **Proper component structure** → Clear test boundaries
- **Accessible components** → Better selectors
- **Clean data flow** → Predictable behavior

---

## Exercises & Practice

### Beginner Exercises

1. **Break Down a UI** - Take a mockup, draw component boundaries
2. **Identify State** - What changes in the app? What's configuration?
3. **Place State** - Where should each piece of state live?
4. **Build Todo App** - Combine all concepts together

### Intermediate Exercises

1. **Implement a Custom Hook** - Extract logic from a component
2. **Refactor with Context** - Replace prop drilling with context
3. **Optimize a Component** - Identify and fix performance issues
4. **Build a Form** - Multi-step form with validation

### Advanced Exercises

1. **Create a Compound Component** - Build a reusable component family
2. **Complex State Machine** - useReducer with many states
3. **Architecture Decision** - Design state management for multi-feature app
4. **Performance Audit** - Profile and optimize a complex component

---

## See Also

- **[README.md](./README.md)** - Overview and paths
- **[SKILL.md](./SKILL.md)** - Complete reference
- **[QUICKSTART.md](./QUICKSTART.md)** - Templates
- **[EXAMPLES.md](./EXAMPLES.md)** - Real examples
- **[ANTI-PATTERNS.md](./ANTI-PATTERNS.md)** - What NOT to do
