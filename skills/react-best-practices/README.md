# React Best Practices Skill

A comprehensive guide to React development patterns, design decisions, and proven techniques for building maintainable, performant React applications.

## What This Skill Covers

This skill teaches React development best practices focused on **how to build robust React applications**, not how to test them. It covers:

- **Fundamental Patterns**: Component composition, data flow, state management
- **React Hooks**: All built-in hooks with clear examples and best practices
- **Common Patterns**: Forms, lists, data fetching, error handling
- **Performance**: When and how to optimize without premature optimization
- **Anti-Patterns**: Common mistakes and how to avoid them
- **Architecture**: Organizing code for maintainability and scalability

## Core Philosophy

React is fundamentally about **thinking in React** - a specific mental model for building UIs:

1. **Break down into components** - Identify component boundaries
2. **Build static first** - Structure UI before adding interactivity
3. **Identify state** - Determine what changes over time
4. **Place state correctly** - Put state where it's needed
5. **Add data flow** - Connect components with props and callbacks

This skill reinforces this mental model throughout, not as abstract theory, but with practical examples and decision-making frameworks.

## Learning Paths

### Fast Track (1-2 hours)
If you just need the essentials:

1. Read: [Thinking in React](./SKILL.md#thinking-in-react)
2. Read: [Component Fundamentals](./SKILL.md#component-fundamentals)
3. Read: [State & Lifecycle](./SKILL.md#state--lifecycle)
4. Browse: [QUICKSTART.md](./QUICKSTART.md) - Copy-paste templates
5. Reference: [ANTI-PATTERNS.md](./ANTI-PATTERNS.md) when confused

### Thorough Learning (4-6 hours)
For a solid understanding:

1. Read: [SKILL.md](./SKILL.md) - Core reference (skip details if needed)
2. Work through: [EXAMPLES.md](./EXAMPLES.md) - Run and modify examples
3. Study: [COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md)
4. Study: [STATE-MANAGEMENT-GUIDE.md](./STATE-MANAGEMENT-GUIDE.md)
5. Reference: [HOOKS-REFERENCE.md](./HOOKS-REFERENCE.md) as needed
6. Review: [ANTI-PATTERNS.md](./ANTI-PATTERNS.md) - Learn from mistakes

### Deep Dive (ongoing)
For ongoing reference and advanced patterns:

1. Keep [SKILL.md](./SKILL.md) as reference material
2. Use [HOOKS-REFERENCE.md](./HOOKS-REFERENCE.md) as cheat sheet
3. Consult [STATE-MANAGEMENT-GUIDE.md](./STATE-MANAGEMENT-GUIDE.md) for architecture decisions
4. Review [COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md) when building new components
5. Check [ANTI-PATTERNS.md](./ANTI-PATTERNS.md) when debugging issues

## Quick Reference

### Most Important Concepts

**Props Down, Callbacks Up**
```javascript
// Parent
<Child data={value} onChange={setValue} />

// Child
<input value={props.data} onChange={e => props.onChange(e.target.value)} />
```

**Lifting State**
```javascript
// State lives in the parent that needs it
// Shared by multiple children? Move to parent
// Deeply nested? Consider Context API
```

**useEffect Dependencies**
```javascript
// Include everything from outer scope that the effect uses
useEffect(() => {
  fetch(`/api/users/${userId}`);
}, [userId]); // Don't forget!
```

**Component Composition**
```javascript
// Flexible: <Card><Body>Content</Body></Card>
// Rigid: <Card title="Title" body="Content" />
```

## Document Overview

| Document | Purpose | Length | Best For |
|----------|---------|--------|----------|
| [SKILL.md](./SKILL.md) | Core reference covering all topics | 1,400 lines | Learning, reference |
| [EXAMPLES.md](./EXAMPLES.md) | Working, copy-paste examples | 1,100 lines | Learning by doing |
| [ANTI-PATTERNS.md](./ANTI-PATTERNS.md) | Common mistakes with solutions | 700 lines | Debugging, learning from mistakes |
| [COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md) | Advanced composition strategies | 600 lines | Advanced patterns, architecture |
| [STATE-MANAGEMENT-GUIDE.md](./STATE-MANAGEMENT-GUIDE.md) | Deciding where state lives | 600 lines | Architecture decisions |
| [HOOKS-REFERENCE.md](./HOOKS-REFERENCE.md) | Quick hook reference | 500 lines | Daily reference |
| [QUICKSTART.md](./QUICKSTART.md) | Copy-paste templates | 400 lines | Getting started fast |
| [INDEX.md](./INDEX.md) | Learning roadmap and navigation | 300 lines | Finding specific topics |
| [README.md](./README.md) | This file | - | Overview |

## What This Skill Does NOT Cover

This skill focuses on React development patterns and best practices. Other skills cover:

- **Testing React code** - See the React Testable Code skill
- **E2E testing** - See the Playwright skill
- **External state management** - Focus is on built-in React patterns
- **Next.js, React Native, etc.** - Framework-specific patterns
- **Specific UI libraries** - Component libraries and frameworks

## Getting Started Now

1. **First time with React?** Start with [QUICKSTART.md](./QUICKSTART.md)
2. **Want to improve?** Read [SKILL.md](./SKILL.md) then review [ANTI-PATTERNS.md](./ANTI-PATTERNS.md)
3. **Need specific pattern?** Check [INDEX.md](./INDEX.md) for topic index
4. **Debugging an issue?** Look up the pattern in [ANTI-PATTERNS.md](./ANTI-PATTERNS.md)
5. **Architectural decision?** Consult [STATE-MANAGEMENT-GUIDE.md](./STATE-MANAGEMENT-GUIDE.md)

## Key Files by Use Case

**I'm learning React:**
1. [SKILL.md](./SKILL.md) - Learn concepts
2. [QUICKSTART.md](./QUICKSTART.md) - Run examples
3. [EXAMPLES.md](./EXAMPLES.md) - See real patterns

**I'm building a component:**
1. [QUICKSTART.md](./QUICKSTART.md) - Find template
2. [COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md) - Choose pattern
3. [STATE-MANAGEMENT-GUIDE.md](./STATE-MANAGEMENT-GUIDE.md) - Decide state placement

**I'm debugging an issue:**
1. [ANTI-PATTERNS.md](./ANTI-PATTERNS.md) - Find the mistake
2. [SKILL.md](./SKILL.md) - Understand the concept
3. [EXAMPLES.md](./EXAMPLES.md) - See the correct pattern

**I'm stuck on hooks:**
1. [HOOKS-REFERENCE.md](./HOOKS-REFERENCE.md) - Quick reference
2. [SKILL.md](./SKILL.md) - Detailed explanation
3. [ANTI-PATTERNS.md](./ANTI-PATTERNS.md) - Common mistakes

## Cross-References with Other Skills

### React Testable Code Skill

This skill complements the React Testable Code skill:
- This skill teaches **how to write React correctly**
- React Testable Code skill teaches **how to write React for testing**
- Together they create maintainable, tested applications

Key differences:
- This skill focuses on component design and data flow
- React Testable Code focuses on testability patterns
- Both use similar foundational patterns (composition, custom hooks)

### Playwright Skill

This skill creates components that can be tested with Playwright:
- Proper component structure makes E2E testing easier
- Clean data flow makes test scenarios clear
- Good accessibility practices help with selectors

## Best Practices Philosophy

This skill is based on several core principles:

1. **React's Way First** - Use React's built-in patterns before external libraries
2. **Composition Over Configuration** - Flexible component composition beats many props
3. **Explicit Over Implicit** - Clear data flow beats magic
4. **Local Over Global** - Keep state as close as possible to where it's used
5. **Simple Before Complex** - useState before useReducer, props before context
6. **Measure Before Optimizing** - Profile before premature optimization

## Modern React Focus

This skill covers modern React with:
- **Functional components** (not class components)
- **React Hooks** (the recommended approach)
- **React 18+** (Suspense, automatic batching, etc.)
- **TypeScript** (with examples, though not required)

## Maintenance and Updates

This skill is kept current with:
- Latest React best practices
- Modern hook patterns (React 18+)
- Community-validated techniques
- Real-world application experience

---

## Quick Links

- **[SKILL.md](./SKILL.md)** - Complete reference guide
- **[QUICKSTART.md](./QUICKSTART.md)** - Templates to get started fast
- **[EXAMPLES.md](./EXAMPLES.md)** - Real working examples
- **[ANTI-PATTERNS.md](./ANTI-PATTERNS.md)** - What NOT to do
- **[HOOKS-REFERENCE.md](./HOOKS-REFERENCE.md)** - Hook cheat sheet
- **[COMPOSITION-PATTERNS.md](./COMPOSITION-PATTERNS.md)** - Advanced patterns
- **[STATE-MANAGEMENT-GUIDE.md](./STATE-MANAGEMENT-GUIDE.md)** - State placement decisions
- **[INDEX.md](./INDEX.md)** - Full topic index

---

**Ready to learn?** Start with [SKILL.md](./SKILL.md) or jump to [QUICKSTART.md](./QUICKSTART.md) for templates!
