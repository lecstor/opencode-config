# Playwright E2E Testing Skill

A comprehensive guide to writing robust, maintainable end-to-end tests with Playwright, following React Testing Library best practices.

## 📚 Contents

This skill includes three comprehensive guides:

### 1. **SKILL.md** - The Complete Philosophy & Guide (780 lines)
The foundational document covering:
- **Core Philosophy**: Why testing user behavior beats testing implementation
- **Locator Strategy**: Complete priority hierarchy from `getByRole` to CSS (never)
- **Web-First Assertions**: How Playwright's auto-waiting works and why manual waits are harmful
- **Auto-Waiting & Actionability**: What Playwright checks before actions
- **Test Isolation**: Designing independent tests that don't depend on each other
- **Common Pitfalls**: What to avoid and why
- **Accessibility as a Side Benefit**: How semantic locators guarantee accessibility
- **Complete Test Structure Examples**: Production-ready test patterns

### 2. **EXAMPLES.md** - Practical Code Examples (954 lines)
Real-world, copy-paste-ready examples for:
- **Form Interactions**: Simple forms, validation, checkboxes, selects, password toggles
- **List & Table Operations**: Filtering, sorting, pagination, inline editing
- **Modal/Dialog Interactions**: Confirmations, form modals, tabbed modals
- **Error Handling**: API failures, timeouts, validation errors
- **Success Messages**: Toast notifications, success pages, inline messages
- **Accessible Form Validation**: ARIA live regions, field associations
- **Keyboard Navigation**: Tab order, Escape key handling
- **Complex Interactions**: Drag & drop, file uploads, multi-select
- **DO's & DON'Ts**: Side-by-side comparisons of right vs wrong patterns
- **Quick Reference**: One-page cheat sheet of common locators and assertions

### 3. **ANTI-PATTERNS.md** - What to Avoid (738 lines)
Detailed analysis of the most common mistakes:
1. Manual waits (setTimeout) - Why they're slow and unreliable
2. Explicit waits (waitForTimeout, waitForSelector) - The problems they cause
3. Test IDs everywhere - When to avoid and why
4. CSS/XPath chains - The brittleness problem
5. Improper assertions - Race conditions explained
6. Testing implementation details - Why it fails during refactoring
7. Flaky selectors - Making tests resilient
8. Test interdependence - Isolation principles
9. Ignoring errors - False positive tests
10. Visibility checking - What Playwright handles automatically

Each anti-pattern includes:
- The problem (with code)
- Why it's harmful (detailed explanation)
- The right way (with code)
- Why it works (the benefits)

## 🎯 Key Principles

### 1. Locator Priority (Strict Hierarchy)
```
1. getByRole (most preferred)
2. getByLabel
3. getByPlaceholder
4. getByAltText
5. getByTitle
6. getByTestId (use sparingly)
7. getByText (as fallback)
8. CSS/XPath (NEVER)
```

### 2. Web-First Testing
- Always use `await expect()` not `expect()`
- Playwright auto-retries for up to 5 seconds
- No manual waits needed
- Clear error messages when things fail

### 3. User-Centric Thinking
- Test what users see and do
- Not internal state or props
- Verify behavior, not implementation
- Tests serve as user stories

### 4. Test Isolation
- Each test is completely independent
- Use `beforeEach()` for common setup
- Tests can run in any order
- Tests can run in parallel

## 💡 Common Scenarios

Here are the key examples you'll find in EXAMPLES.md:

### Forms
```javascript
await page.getByLabel('Email').fill('user@example.com');
await page.getByRole('button', { name: 'Submit' }).click();
await expect(page.getByText('Success!')).toBeVisible();
```

### Lists with Filtering
```javascript
await page.getByRole('button', { name: 'Filters' }).click();
await page.getByLabel('Category').selectOption('Electronics');
await page.getByRole('button', { name: 'Apply' }).click();
await expect(page.getByRole('listitem')).toHaveCount(5);
```

### Modals
```javascript
await page.getByRole('button', { name: 'Delete' }).click();
const dialog = page.getByRole('alertdialog');
await dialog.getByRole('button', { name: 'Confirm' }).click();
await expect(dialog).not.toBeVisible();
```

### Error Handling
```javascript
await page.route('**/api/**', route => route.abort());
await page.getByRole('button', { name: 'Save' }).click();
await expect(page.getByRole('alert')).toContainText('Failed');
```

## 🚀 Getting Started

1. **Read SKILL.md first** - Understand the philosophy and principles
2. **Check EXAMPLES.md** - Find examples matching your use case
3. **Review ANTI-PATTERNS.md** - Learn what to avoid
4. **Apply the 3 rules**:
   - Use semantic locators (getByRole, getByLabel)
   - Use web-first assertions (await expect)
   - Test user behavior, not implementation

## ❌ Red Flags (Refactor Immediately)

```javascript
// 🚩 Manual waits
await page.waitForTimeout(1000);

// 🚩 CSS chains
page.locator('div.modal > form > input:nth-child(2)')

// 🚩 Immediate assertions
expect(await page.innerText()).toContain('...');

// 🚩 Testing state
window.__state.value

// 🚩 Test dependencies
// Test assumes previous test ran first
```

**Replace with:**
- `await expect()` (web-first)
- `getByRole()` / `getByLabel()`
- Independent test setup
- User behavior verification

## 📊 By The Numbers

- **2,472 total lines** of comprehensive documentation
- **50+ complete code examples** ready to use
- **10 anti-patterns** detailed with solutions
- **Accessibility testing** as a built-in side effect
- **0 manual waits** needed (Playwright handles it)

## 🎓 What You'll Learn

### Core Concepts
- ✅ Why `getByRole` is the strongest locator
- ✅ How web-first assertions eliminate flakiness
- ✅ Why manual waits are the enemy of reliability
- ✅ How test isolation prevents cascading failures
- ✅ Why accessibility testing is free with semantic locators

### Practical Skills
- ✅ Writing tests that read like user stories
- ✅ Handling async operations properly
- ✅ Testing forms with validation
- ✅ Testing modals, lists, and complex interactions
- ✅ Error handling and edge cases
- ✅ Keyboard navigation testing
- ✅ Accessibility-first testing

### What NOT To Do
- ✅ Avoid manual waits
- ✅ Avoid CSS/XPath selectors
- ✅ Avoid testing implementation details
- ✅ Avoid test interdependence
- ✅ Avoid over-using test IDs

## 🔗 Related Concepts

This skill builds on:
- **Playwright Documentation**: Official API and best practices
- **React Testing Library Philosophy**: Testing user behavior, not implementation
- **WCAG/Accessibility Guidelines**: Semantic HTML and ARIA
- **Testing JavaScript by Kent C. Dodds**: User-centric testing principles

## 💬 Key Quotes

> "Test behavior, not implementation" - React Testing Library
>
> "If it's not visible to a screen reader, it's not visible to your test" - This Skill
>
> "Manual waits are a code smell that indicates a real synchronization problem" - Playwright Docs
>
> "Write tests that give you confidence, not coverage" - Testing Best Practices

## 📝 File Structure

```
/Users/jason/.config/opencode/skills/e2e-testing/
├── SKILL.md           (24 KB) - Core philosophy & complete guide
├── EXAMPLES.md        (26 KB) - 50+ practical code examples
├── ANTI-PATTERNS.md   (21 KB) - 10 common mistakes & solutions
└── README.md          (this file) - Overview & quick start
```

## 🎯 Usage

Use this skill when:
- **Starting a new Playwright test suite**
- **Migrating from other testing tools**
- **Your tests are flaky or slow**
- **You need to test forms, lists, and modals**
- **You want accessibility testing built-in**
- **You're teaching E2E testing best practices**

## 🤝 Best Practices Summary

1. **Always await assertions**: `await expect(...)`
2. **Use semantic locators**: `getByRole` > `getByLabel` > others
3. **Never use CSS/XPath**: They're brittle and don't verify accessibility
4. **Isolate tests**: No test should depend on another
5. **Test user behavior**: Not implementation details
6. **Embrace auto-waiting**: Don't fight it with manual waits
7. **Accessibility is free**: Semantic locators verify it automatically

---

**Version**: 1.0  
**Last Updated**: March 2026  
**Status**: Production-Ready  
**Coverage**: Comprehensive guide to Playwright E2E testing with RTL best practices
