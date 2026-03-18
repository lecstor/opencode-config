# React Testable Code Skill: Navigation Guide

Welcome to the comprehensive guide for writing React code that's easy to test with Playwright. This skill is built around a simple principle: **good React code is code that's easy to test.**

---

## Quick Navigation

### 🚀 Getting Started
- **New to this skill?** Start with [QUICKSTART.md](QUICKSTART.md) - 5 copy-paste templates
- **Want the full story?** Read [SKILL.md](SKILL.md) - the comprehensive guide
- **Learning by example?** See [EXAMPLES.md](EXAMPLES.md) - 30+ production components

### 📚 Learning Paths

**Path 1: Fast-Track Learner**
1. [QUICKSTART.md](QUICKSTART.md) - Get started immediately (20 min)
2. [ANTI-PATTERNS.md](ANTI-PATTERNS.md) - Learn what NOT to do (15 min)
3. [EXAMPLES.md](EXAMPLES.md) - See real implementations (30 min)

**Path 2: Thorough Learner**
1. [SKILL.md](SKILL.md) - Understand the principles (60 min)
2. [ACCESSIBILITY-FOR-TESTING.md](ACCESSIBILITY-FOR-TESTING.md) - Why accessibility = testability (30 min)
3. [COMPONENT-ARCHITECTURE.md](COMPONENT-ARCHITECTURE.md) - How to structure components (40 min)
4. [EXAMPLES.md](EXAMPLES.md) - See everything in practice (45 min)
5. [ANTI-PATTERNS.md](ANTI-PATTERNS.md) - Learn from mistakes (20 min)

**Path 3: Reference Lookups**
- Specific component pattern? → [EXAMPLES.md](EXAMPLES.md)
- What breaks tests? → [ANTI-PATTERNS.md](ANTI-PATTERNS.md)
- Accessibility questions? → [ACCESSIBILITY-FOR-TESTING.md](ACCESSIBILITY-FOR-TESTING.md)
- Component design questions? → [COMPONENT-ARCHITECTURE.md](COMPONENT-ARCHITECTURE.md)

---

## Document Overview

### [SKILL.md](SKILL.md) - The Comprehensive Guide (~1100 lines)
**The full philosophy and implementation guide**

Contains:
- The Testing Philosophy: why testability matters
- Thinking in React (5-step process)
- Semantic HTML & Accessibility
- Component Architecture
- State Management
- Props vs State
- Data Flow Patterns (one-way, inverse)
- Event Handlers & Forms
- Composition Over Configuration
- Testing Patterns

**Read this when:** You want to understand the full picture and build a mental model of testable React.

**Key sections:**
- "The Philosophy" explains why good React = testable React
- "Thinking in React for Testability" walks through component design
- "Semantic HTML & Accessibility First" shows how HTML choices affect tests
- "Component Architecture" covers component boundaries and composition
- "Data Flow Patterns" explains props, state, and callbacks

---

### [EXAMPLES.md](EXAMPLES.md) - 30+ Production-Ready Components (~1200 lines)
**Copy-paste ready, fully testable component examples**

Contains:
1. **Form Components** (5 examples)
   - Search form
   - Login form with validation
   - Multi-field contact form
   - Filter form with multiple options
   - Real-time validation

2. **List & Table Components** (5 examples)
   - Todo list with add/remove
   - Data table with sorting and pagination
   - Expandable rows
   - Product list with filtering
   - Search results with highlighting

3. **Modal & Dialog Components** (3 examples)
   - Confirmation dialog
   - Modal with form
   - Alert dialog

4. **Compound Components** (4 examples)
   - Tabs
   - Accordion
   - Menu/dropdown
   - Breadcrumbs

5. **Async & Loading States** (3 examples)
   - Data fetching with loading/error
   - Form with async submission
   - Infinite scroll with loading states

6. **Validation Patterns** (2 examples)
   - Real-time form validation
   - Password strength indicator

7. **Error Handling** (1 example)
   - Error boundary component

Each example includes:
- Complete working code
- Why it's testable (semantic HTML, accessibility)
- Playwright test examples

**Read this when:** You need a specific pattern or want to see how real components are built.

---

### [ANTI-PATTERNS.md](ANTI-PATTERNS.md) - What Breaks E2E Tests (~600 lines)
**Learn what not to do and why**

Contains 8 anti-patterns:
1. Divs with onClick instead of buttons
2. Missing labels on form inputs
3. Using test IDs instead of semantic HTML
4. Missing accessible names on icon buttons
5. State in the wrong component
6. Props drilling too deep
7. Boolean prop proliferation
8. Missing error boundaries and error states

Each anti-pattern includes:
- ❌ What not to do (broken code)
- Why it breaks tests
- ✅ How to fix it (correct code)
- Test examples showing the difference

**Read this when:** You want to avoid common mistakes or understand why something didn't work.

---

### [ACCESSIBILITY-FOR-TESTING.md](ACCESSIBILITY-FOR-TESTING.md) - Why They're the Same (~500 lines)
**Understand the critical connection between accessibility and testability**

Contains:
- The Accessibility Tree: what Playwright actually sees
- Semantic HTML builds the right tree
- ARIA attributes enhance the tree (aria-label, aria-live, aria-expanded, etc.)
- Semantic HTML elements and their roles
- Building accessible (and testable) forms
- Keyboard navigation testing
- Image and alternative text
- Complete accessibility checklist

**Key insight:** Playwright queries the accessibility tree, not the DOM. When you build accessible code, E2E testing works automatically.

**Read this when:** You're struggling to find elements in tests, or you want to understand why semantic HTML matters.

---

### [COMPONENT-ARCHITECTURE.md](COMPONENT-ARCHITECTURE.md) - How to Structure Components (~500 lines)
**Design patterns for component boundaries and composition**

Contains:
- Component Boundaries: Single Responsibility Principle
- Composition Over Inheritance
- Thinking About State: Where should it live?
- Context vs Props: When to use each
- Custom Hooks: Extracting logic
- Compound Components: Flexible APIs
- Avoiding Prop Drilling
- Performance Optimization

**Read this when:** You're designing a new component or refactoring an existing one.

---

### [QUICKSTART.md](QUICKSTART.md) - 5 Copy-Paste Templates (~400 lines)
**Get started immediately with working templates**

Contains 5 templates:
1. Form Component
2. Data Fetching with Loading/Error
3. List with Add/Remove
4. Modal/Dialog
5. Tabs

Plus:
- Converting styled components to semantic HTML
- Playwright test structure
- Common Playwright queries
- Testable components checklist

**Read this when:** You need to build something right now.

---

## The Core Principle

Every pattern in this skill connects back to one core principle:

> **Good React code is code that's easy to test with E2E tests.**

When you write code that:
- Uses semantic HTML
- Follows accessibility standards
- Has clear component hierarchies
- Makes data flow explicit
- Avoids implementation details

...then Playwright testing becomes trivial. You don't need `data-testid` hacks or brittle selectors. You just ask "what would a user see or do?"

---

## Key Concepts Across Documents

### Semantic HTML
- [SKILL.md](SKILL.md#semantic-html--accessibility-first) - Why it matters
- [ACCESSIBILITY-FOR-TESTING.md](ACCESSIBILITY-FOR-TESTING.md#semantic-html-builds-the-right-tree) - How it works
- [EXAMPLES.md](EXAMPLES.md) - 30+ examples using semantic HTML
- [ANTI-PATTERNS.md](ANTI-PATTERNS.md#anti-pattern-1-divs-with-onclick-instead-of-buttons) - What happens without it

### Component Boundaries
- [SKILL.md](SKILL.md#thinking-in-react-for-testability) - 5-step process
- [COMPONENT-ARCHITECTURE.md](COMPONENT-ARCHITECTURE.md#component-boundaries-the-single-responsibility-principle) - Deep dive
- [EXAMPLES.md](EXAMPLES.md) - Every example demonstrates good boundaries
- [ANTI-PATTERNS.md](ANTI-PATTERNS.md#anti-pattern-6-props-drilling-too-deep) - What goes wrong

### State Management
- [SKILL.md](SKILL.md#state-management) - Patterns and best practices
- [COMPONENT-ARCHITECTURE.md](COMPONENT-ARCHITECTURE.md#thinking-about-state-where-should-it-live) - Decision tree
- [EXAMPLES.md](EXAMPLES.md) - Every example shows state in right place
- [ANTI-PATTERNS.md](ANTI-PATTERNS.md#anti-pattern-5-state-in-the-wrong-component) - Consequences of mistakes

### Accessibility
- [ACCESSIBILITY-FOR-TESTING.md](ACCESSIBILITY-FOR-TESTING.md) - Complete deep dive
- [SKILL.md](SKILL.md#semantic-html--accessibility-first) - Best practices section
- [EXAMPLES.md](EXAMPLES.md) - Every component is built accessibly
- [ANTI-PATTERNS.md](ANTI-PATTERNS.md#anti-pattern-4-missing-accessible-names-on-icon-buttons) - Common mistakes

---

## How to Use This Skill

### Scenario 1: Building a New Component
1. Start with [QUICKSTART.md](QUICKSTART.md) for the right template
2. Check [COMPONENT-ARCHITECTURE.md](COMPONENT-ARCHITECTURE.md) if you need design guidance
3. Refer to [EXAMPLES.md](EXAMPLES.md) for similar patterns
4. Check [ANTI-PATTERNS.md](ANTI-PATTERNS.md) before finalizing
5. Write tests using Playwright patterns from [SKILL.md](SKILL.md#testing-patterns)

### Scenario 2: Debugging a Flaky Test
1. Read [ANTI-PATTERNS.md](ANTI-PATTERNS.md) - your component likely uses one
2. Check [ACCESSIBILITY-FOR-TESTING.md](ACCESSIBILITY-FOR-TESTING.md) - you probably need semantic HTML
3. Verify [SKILL.md](SKILL.md#data-flow-patterns) - data flow might be unclear
4. Fix the component (not the test!)

### Scenario 3: Learning React Properly
1. Read [SKILL.md](SKILL.md) start to finish
2. Work through [COMPONENT-ARCHITECTURE.md](COMPONENT-ARCHITECTURE.md)
3. Study [EXAMPLES.md](EXAMPLES.md) and understand each pattern
4. Review [ANTI-PATTERNS.md](ANTI-PATTERNS.md) to see what you now understand
5. Write your own components, test with [QUICKSTART.md](QUICKSTART.md) patterns

### Scenario 4: Quick Reference
- Form component? → [EXAMPLES.md - Login Form](EXAMPLES.md#2-login-form-with-validation)
- State confusion? → [SKILL.md - Props vs State](SKILL.md#props-vs-state)
- Test won't find element? → [ACCESSIBILITY-FOR-TESTING.md](ACCESSIBILITY-FOR-TESTING.md)
- Component design question? → [COMPONENT-ARCHITECTURE.md](COMPONENT-ARCHITECTURE.md)
- Seeing a code smell? → [ANTI-PATTERNS.md](ANTI-PATTERNS.md)

---

## Beyond This Skill

Once you master these patterns, consider:

1. **Performance optimization** - See [COMPONENT-ARCHITECTURE.md](COMPONENT-ARCHITECTURE.md#performance-optimization-when-needed)
2. **State management libraries** - After mastering props/state/context
3. **Advanced Playwright** - After mastering basic queries from [QUICKSTART.md](QUICKSTART.md)
4. **Accessibility standards** - WCAG 2.1 AA level compliance
5. **Type safety** - TypeScript for better component APIs

---

## Summary

This skill teaches you that **testability is a design goal, not an afterthought**. When you design components with testing in mind from day one, you naturally:

- Use semantic HTML
- Write accessible code
- Create clear component boundaries
- Make data flow explicit
- Avoid implementation details
- Write code that's easy to maintain

And as a bonus, Playwright tests that verify user behavior work perfectly.

Start with [QUICKSTART.md](QUICKSTART.md) and build from there!
