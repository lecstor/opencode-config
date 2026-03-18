# Playwright E2E Testing with React Testing Library Best Practices

## Overview

This skill teaches you how to write robust, maintainable, and reliable end-to-end tests using Playwright, following the philosophy and patterns of React Testing Library. The core principle is: **test user behavior, not implementation details**. It is also useful for writing Playright code for purposes other than testing.

## Core Philosophy

### Why This Matters

The best tests are those that mimic how real users interact with your application. Rather than testing internal state or implementation details, you should verify what users see and can do.

**Benefits:**
- Tests break only when behavior actually changes
- Refactoring code doesn't break tests
- Tests serve as documentation for actual user workflows
- Accessibility is guaranteed as a side effect
- Fewer false positives and flaky tests

### The User-Centric Approach

Imagine a user navigating your application:
1. They see elements (buttons, inputs, headings)
2. They click, type, and interact with them
3. They wait for responses and see results
4. They verify outcomes (new page, success message, updated data)

Your tests should reflect this exact flow, nothing more, nothing less.

---

## Locator Strategy: Priority Order

The hierarchy below reflects how you should prioritize your selectors. This order matters because it moves from **semantic, accessible selectors** to **brittle, implementation-dependent** ones.

### 1. **getByRole** (Most Preferred)

Query elements by their accessibility role—the strongest signal for user-facing interfaces.

```javascript
// Button by role
await page.getByRole('button', { name: 'Submit' }).click();

// Link by role
await page.getByRole('link', { name: 'Home' }).click();

// Heading by role
await expect(page.getByRole('heading', { level: 1, name: 'Welcome' })).toBeVisible();

// Checkbox by role
await page.getByRole('checkbox', { name: 'Accept terms' }).check();

// Tab by role
await page.getByRole('tab', { name: 'Settings' }).click();

// Dialog/modal
const dialog = page.getByRole('dialog');
await expect(dialog).toContainText('Confirm action');
```

**Why getByRole:**
- Aligns with screen readers
- Ensures semantic HTML structure
- Forces accessibility-first development
- Won't break if CSS classes change
- Resistant to styling changes

**Common Roles:**
- `button`, `link`, `heading`, `checkbox`, `radio`, `textbox`, `combobox`
- `tab`, `tablist`, `dialog`, `alertdialog`, `menuitem`
- `table`, `row`, `rowgroup`, `columnheader`, `rowheader`, `cell`

### 2. **getByLabel** (Second Best)

Query form elements by their associated label text. Perfect for inputs, checkboxes, and radio buttons.

```javascript
// Text input with label
await page.getByLabel('Email address').fill('user@example.com');

// Password input
await page.getByLabel('Password').fill('secretpassword');

// Checkbox with label
await page.getByLabel('Subscribe to newsletter').check();

// Radio button
await page.getByLabel('Agree to terms').check();
```

**Why getByLabel:**
- Labels are semantically correct HTML
- Creates accessible forms
- Changes to CSS don't affect tests
- Self-documenting (label text is the expectation)

### 3. **getByPlaceholder** (Third Priority)

Query input fields by their placeholder text.

```javascript
await page.getByPlaceholder('Enter your name').fill('John');

// When no label exists
await page.getByPlaceholder('john@example.com').fill('test@test.com');
```

**When to use:**
- Only when no label is present
- For search fields with placeholder guidance
- Temporary solution (should add proper labels)

### 4. **getByAltText** (Fourth Priority)

Query images by their alt text. Essential for images without visible labels.

```javascript
// Find image by alt text
const logo = page.getByAltText('Company logo');

// Verify image is visible
await expect(logo).toBeVisible();

// Interact with image (if it's a button)
await page.getByAltText('Delete item').click();
```

**When to use:**
- Every image should have meaningful alt text
- Tests verify alt text exists for accessibility
- Only for non-decorative images

### 5. **getByTitle** (Fifth Priority)

Query elements by their title attribute. Less common, used when other locators don't apply.

```javascript
// Tooltip or title attribute
await expect(page.getByTitle('This field is required')).toBeVisible();

// Icon button with title
await page.getByTitle('Close menu').click();
```

**When to use:**
- Fallback for icon buttons with no visible text
- Tooltip elements
- Rare, usually indicates missing accessibility

### 6. **getByTestId** (Sixth Priority - Use Sparingly)

Query elements by test ID attribute. Only use when all semantic queries fail.

```javascript
// Last resort locator
await page.locator('[data-testid="special-widget"]').click();

// Or using getByTestId helper
const widget = page.getByTestId('special-widget');
```

**When to use:**
- Complex components with no semantic role
- Third-party widgets without proper semantics
- Temporary workaround while waiting for proper HTML structure

**Why it's last:**
- Test IDs couple tests to implementation
- They don't verify accessibility
- Encourages poor HTML practices
- Tests don't validate what users see

### 7. **getByText** (Almost Never First)

Query by visible text content. Use as fallback only, after all semantic locators fail.

```javascript
// Last resort for clickable text
await page.getByText('Click here').click();

// Verify text appears
await expect(page.getByText('Success!')).toBeVisible();
```

**Avoid because:**
- Brittle—breaks if text changes
- Doesn't guarantee functionality
- Button with text should use `getByRole('button', { name: ... })`

### 8. **CSS/XPath Selectors (NEVER)**

**NEVER use CSS or XPath selectors as your primary locators.**

```javascript
// ❌ NEVER DO THIS
await page.locator('div.modal > button.close').click();
await page.locator('//div[@class="item"]').fill('text');

// ❌ Also avoid
await page.locator('.btn-primary:nth-child(2)').click();
```

**Why they're forbidden:**
- Brittle—break with CSS changes
- Don't verify accessibility
- Tightly coupled to DOM structure
- Maintenance nightmare
- Don't verify user-facing elements
- Invisible elements can still be selected
- No auto-waiting capability

---

## Web-First Assertions with Retry-Ability

### The Game-Changer: Auto-Waiting Assertions

Playwright's `await expect()` is fundamentally different from synchronous assertions. It automatically waits (with retry-ability built in) until the condition is met.

```javascript
// ✅ GOOD: Web-first assertion with automatic retries
await expect(page.getByText('Loading...')).toBeVisible();
// Waits up to 5 seconds for this text to appear

// ❌ BAD: Immediate assertion without retry
expect(await page.getByText('Loading...').isVisible()).toBe(true);
// May fail if page isn't loaded yet
```

### How Auto-Waiting Works

When you use `await expect()` with a Playwright locator, Playwright:

1. **Evaluates the condition immediately**
2. **If it fails, re-evaluates every 100ms**
3. **Continues until condition passes or timeout expires** (default 5000ms)
4. **Never needs manual waits**

```javascript
// ✅ Element will be found and clicked when ready
await page.getByRole('button', { name: 'Submit' }).click();
// Playwright auto-waits for element to be actionable

// ✅ Text will appear and assertion will pass
await expect(page.getByText('Success!')).toBeVisible();
// Waits until success message appears
```

### Key Web-First Assertions

```javascript
// Visibility assertions (wait for element to be visible)
await expect(page.getByRole('button', { name: 'Save' })).toBeVisible();
await expect(page.getByText('Error message')).toBeHidden();

// Content assertions (wait for text/value)
await expect(page.getByLabel('Email')).toHaveValue('user@example.com');
await expect(page.getByText('Total')).toContainText('$99.99');

// State assertions (wait for state change)
await expect(page.getByRole('checkbox')).toBeChecked();
await expect(page.getByRole('button')).toBeDisabled();
await expect(page.getByRole('button')).toBeEnabled();

// Count assertions (wait for correct number of elements)
await expect(page.getByRole('listitem')).toHaveCount(5);

// Attribute assertions
await expect(page.getByRole('link')).toHaveAttribute('href', '/contact');

// Class assertions
await expect(page.getByRole('button')).toHaveClass('primary');

// URL assertion
await expect(page).toHaveURL('/thank-you');
```

### Why Manual Waits Are Harmful

**Manual waits are the enemy of reliable tests.** They hide race conditions and make tests slower.

```javascript
// ❌ TERRIBLE: Manual wait (setTimeout)
await page.goto('/form');
await page.fill('input[name="email"]', 'test@example.com');
await page.waitForTimeout(2000); // Arbitrary wait!
await page.click('button'); // May still be too slow or too slow

// ❌ ALSO TERRIBLE: Explicit wait for URL/selector
await page.waitForSelector('.success-message', { timeout: 5000 });

// ✅ EXCELLENT: Web-first assertion with auto-wait
await page.goto('/form');
await page.getByLabel('Email').fill('test@example.com');
await page.getByRole('button', { name: 'Submit' }).click();
await expect(page).toHaveURL('/thank-you');
// All waits are implicit and automatic
```

**The Problems with Manual Waits:**
1. **Tests become slow** - wait for maximum timeout even if ready sooner
2. **Race conditions** - arbitrary delays don't guarantee synchronization
3. **Flakiness** - what works locally may fail in CI
4. **Maintenance burden** - adjusting waits requires tweaking timeouts
5. **Masking bugs** - they hide real synchronization issues

**The Playwright Guarantee:**
If you use proper locators and web-first assertions, you never need `waitForTimeout()` or `waitForSelector()`.

---

## Auto-Waiting and Actionability Checks

### What Playwright Auto-Waits For

When you perform an action (click, fill, etc.), Playwright automatically:

1. **Waits for element to be attached to DOM**
2. **Waits for element to be visible**
3. **Waits for element to be stable** (not animating)
4. **Waits for element to be enabled** (if applicable)
5. **Waits for element to be receiving pointer events** (no other element covering it)

This is done automatically—you don't write code for it.

```javascript
// ✅ All these waits happen automatically
await page.getByRole('button', { name: 'Submit' }).click();
// Playwright ensures:
// - Button is in DOM
// - Button is visible
// - Button is not covered by other elements
// - Button is enabled
// - Button receives pointer events

// If any condition fails, action waits up to 30 seconds
```

### Auto-Waiting in Assertions

The same applies to assertions:

```javascript
// ✅ Waits up to 5 seconds for element to be visible
await expect(page.getByText('Success!')).toBeVisible();

// ✅ Waits up to 5 seconds for element to be hidden
await expect(page.getByRole('dialog')).toBeHidden();

// ✅ Waits up to 5 seconds for value to match
await expect(page.getByLabel('Count')).toHaveValue('42');
```

### Custom Timeouts

You can adjust timeout for specific operations:

```javascript
// Set custom timeout for this assertion (wait 10 seconds)
await expect(page.getByText('Slow data')).toBeVisible({ timeout: 10000 });

// Set custom timeout for this action (wait 10 seconds)
await page.getByRole('button').click({ timeout: 10000 });

// Never override timeouts lightly—it indicates underlying issues
```

---

## Test Isolation and Independent Design

### Principle: Each Test Stands Alone

Tests should never depend on other tests, their execution order, or shared state. Each test should:

1. **Set up its own prerequisites** (login, data, etc.)
2. **Not rely on previous tests running first**
3. **Clean up after itself** (database, cookies, etc.)
4. **Be executable in any order**
5. **Be executable multiple times with same results**

```javascript
// ❌ BAD: Test depends on previous test state
test('user can edit profile', async ({ page }) => {
  // Assumes user is logged in from previous test!
  await page.goto('/profile');
  // ...
});

// ✅ GOOD: Each test sets up its own state
test('user can edit profile', async ({ page }) => {
  // Set up authentication first
  await loginAs(page, 'user@example.com');
  
  // Then do the test
  await page.goto('/profile');
  // ...
});
```

### Using beforeEach for Common Setup

Use `test.beforeEach()` to set up state that every test in a file needs:

```javascript
import { test, expect } from '@playwright/test';

test.describe('User Profile', () => {
  test.beforeEach(async ({ page }) => {
    // Every test in this suite will:
    // 1. Start at login page
    // 2. Log in as test user
    await page.goto('/login');
    await page.getByLabel('Email').fill('testuser@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Sign In' }).click();
    await expect(page).toHaveURL('/dashboard');
  });

  test('can view profile', async ({ page }) => {
    // User is already logged in
    await page.goto('/profile');
    await expect(page.getByRole('heading', { name: 'My Profile' })).toBeVisible();
  });

  test('can edit profile', async ({ page }) => {
    // User is already logged in
    await page.goto('/profile/edit');
    await page.getByLabel('Name').fill('John Doe');
    await page.getByRole('button', { name: 'Save' }).click();
    await expect(page.getByText('Profile updated')).toBeVisible();
  });
});
```

### Test Fixtures for Reusable Setup

Create custom fixtures for complex setup:

```javascript
import { test as base, expect } from '@playwright/test';

// Custom fixture: authenticated user
const test = base.extend({
  authenticatedPage: async ({ page }, use) => {
    await page.goto('/login');
    await page.getByLabel('Email').fill('testuser@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Sign In' }).click();
    await expect(page).toHaveURL('/dashboard');
    
    // Fixture is ready for test
    await use(page);
    
    // Cleanup (if needed)
    await page.getByRole('button', { name: 'Logout' }).click();
  },
});

test('can edit profile', async ({ authenticatedPage: page }) => {
  await page.goto('/profile/edit');
  await page.getByLabel('Name').fill('John Doe');
  await page.getByRole('button', { name: 'Save' }).click();
  await expect(page.getByText('Profile updated')).toBeVisible();
});
```

### Database Cleanup Between Tests

For tests that modify data, use `test.afterEach()` to clean up:

```javascript
test.describe('User Management', () => {
  test.afterEach(async ({ page }) => {
    // Clean up created test data
    // Use API calls for faster cleanup than UI
    const response = await page.request.delete('/api/test-users');
    expect(response.ok()).toBe(true);
  });

  test('can create user', async ({ page }) => {
    await page.goto('/admin/users');
    await page.getByRole('button', { name: 'Add User' }).click();
    
    const dialog = page.getByRole('dialog');
    await dialog.getByLabel('Email').fill('newuser@example.com');
    await dialog.getByLabel('Name').fill('New User');
    await dialog.getByRole('button', { name: 'Create' }).click();
    
    await expect(page.getByText('newuser@example.com')).toBeVisible();
  });
});
```

---

## Common Pitfalls and Anti-Patterns

### 1. Testing Implementation Instead of Behavior

```javascript
// ❌ WRONG: Testing internal state/props
test('counter increments', async ({ page }) => {
  await page.goto('/counter');
  const state = await page.evaluate(() => window.__state.counter);
  expect(state).toBe(1);
});

// ✅ CORRECT: Testing visible behavior
test('counter increments', async ({ page }) => {
  await page.goto('/counter');
  await expect(page.getByText('Count: 0')).toBeVisible();
  await page.getByRole('button', { name: 'Increment' }).click();
  await expect(page.getByText('Count: 1')).toBeVisible();
});
```

### 2. Not Waiting for Actual Conditions

```javascript
// ❌ WRONG: Checking immediately without waiting
test('data loads', async ({ page }) => {
  await page.goto('/users');
  const count = await page.locator('li').count();
  expect(count).toBeGreaterThan(0); // May fail if still loading!
});

// ✅ CORRECT: Wait for data to actually load
test('data loads', async ({ page }) => {
  await page.goto('/users');
  // Wait for at least one user item to be visible
  await expect(page.getByRole('listitem')).toHaveCount(3);
});
```

### 3. Overusing Test IDs

```javascript
// ❌ WRONG: Test ID instead of semantic locators
<button data-testid="submit-btn">Submit</button>

test('can submit', async ({ page }) => {
  await page.getByTestId('submit-btn').click();
});

// ✅ CORRECT: Use semantic HTML and accessible locators
<button>Submit</button>

test('can submit', async ({ page }) => {
  await page.getByRole('button', { name: 'Submit' }).click();
});
```

### 4. CSS and XPath Chains

```javascript
// ❌ TERRIBLE: Fragile CSS selectors
await page.locator('div.container > form > div:nth-child(2) input').fill('text');
await page.locator('.modal-content button.primary').click();

// ✅ GOOD: Semantic, accessible locators
await page.getByLabel('Email').fill('text');
await page.getByRole('button', { name: 'Submit' }).click();
```

### 5. Ignoring Errors and Flakiness

```javascript
// ❌ WRONG: Suppressing errors without investigation
test('form submits', async ({ page }) => {
  try {
    await page.getByRole('button', { name: 'Submit' }).click();
  } catch (e) {
    // Ignoring error!
  }
});

// ✅ CORRECT: Investigate and fix root cause
test('form submits', async ({ page }) => {
  await page.goto('/form');
  await page.getByLabel('Email').fill('test@example.com');
  await expect(page.getByRole('button', { name: 'Submit' })).toBeEnabled();
  await page.getByRole('button', { name: 'Submit' }).click();
  await expect(page).toHaveURL('/success');
});
```

### 6. Depending on Test Order

```javascript
// ❌ WRONG: Second test depends on first
test('login', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password');
  await page.getByRole('button', { name: 'Sign In' }).click();
});

test('view dashboard', async ({ page }) => {
  // Assumes previous test logged in!
  await page.goto('/dashboard');
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});

// ✅ CORRECT: Each test is independent
test('login', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password');
  await page.getByRole('button', { name: 'Sign In' }).click();
  await expect(page).toHaveURL('/dashboard');
});

test('view dashboard', async ({ page }) => {
  // Set up authentication independently
  await loginAs(page, 'user@example.com');
  
  // Now test the feature
  await page.goto('/dashboard');
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});
```

---

## Accessibility as a Side Benefit

### Why Semantic Locators = Accessibility Testing

By using `getByRole`, `getByLabel`, and other semantic locators, you're automatically testing accessibility:

```javascript
// ✅ Using getByRole automatically tests:
// - Proper heading hierarchy
// - Correct button semantics
// - Proper form structure
// - Dialog accessibility
// - Tab order and focus management

await page.getByRole('heading', { level: 1, name: 'Welcome' });
// Verifies heading exists with proper level

await page.getByLabel('Email');
// Verifies form input has associated label (WCAG requirement)

await page.getByRole('button', { name: 'Submit' });
// Verifies button is properly marked as button role
```

### Screen Reader Equivalence

Your tests effectively verify what a screen reader user would experience:

```javascript
// ✅ This test validates:
// 1. Button has accessible name ("Save Profile")
// 2. Button is announced as button to screen readers
// 3. Button is keyboard accessible
// 4. Button has proper focus indicators

test('can save profile', async ({ page }) => {
  await page.goto('/profile/edit');
  
  // Screen reader user can find and interact with this
  await page.getByLabel('Full Name').fill('John Doe');
  await page.getByRole('button', { name: 'Save Profile' }).click();
  
  // Screen reader user hears success message
  await expect(page.getByRole('status')).toContainText('Profile saved');
});
```

### Accessibility Testing Checklist

By following this skill, you automatically verify:

- ✅ **Semantic HTML structure** - Using roles ensures proper elements
- ✅ **Form labels** - `getByLabel` requires proper `<label>` elements
- ✅ **Heading hierarchy** - Using heading roles ensures proper nesting
- ✅ **Button accessibility** - Buttons are actually buttons, not divs
- ✅ **Dialog structure** - Dialogs are marked as role="dialog"
- ✅ **ARIA attributes** - Tests verify they're present and correct
- ✅ **Keyboard navigation** - Actions don't use mouse-only interactions
- ✅ **Focus management** - Interactive elements are focusable
- ✅ **Alt text for images** - Images have meaningful alt attributes

---

## Complete Test Structure Example

```javascript
import { test, expect } from '@playwright/test';

// Setup helper
async function loginAs(page, email) {
  await page.goto('/login');
  await page.getByLabel('Email').fill(email);
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: 'Sign In' }).click();
  await expect(page).toHaveURL('/dashboard');
}

test.describe('User Dashboard', () => {
  test.beforeEach(async ({ page }) => {
    // Ensure every test starts with logged-in user
    await loginAs(page, 'testuser@example.com');
  });

  test('displays user greeting', async ({ page }) => {
    await page.goto('/dashboard');
    
    // Verify greeting is visible with correct name
    await expect(page.getByRole('heading', { name: 'Welcome, Test User' }))
      .toBeVisible();
  });

  test('displays recent activities', async ({ page }) => {
    await page.goto('/dashboard');
    
    // Wait for activity list to load and have items
    await expect(page.getByRole('list')).toBeVisible();
    await expect(page.getByRole('listitem')).toHaveCount(5);
    
    // Verify first activity content
    const firstActivity = page.getByRole('listitem').first();
    await expect(firstActivity).toContainText('Profile updated');
  });

  test('can filter activities', async ({ page }) => {
    await page.goto('/dashboard');
    
    // Open filter dialog
    await page.getByRole('button', { name: 'Filter' }).click();
    
    // Filter by activity type
    await page.getByLabel('Activity Type').fill('Login');
    await page.getByRole('button', { name: 'Apply' }).click();
    
    // Verify only login activities shown
    const activities = page.getByRole('listitem');
    await expect(activities).toHaveCount(2);
    
    await expect(activities.first()).toContainText('Logged in');
  });

  test('can logout', async ({ page }) => {
    await page.goto('/dashboard');
    
    // Click logout button
    await page.getByRole('button', { name: 'Logout' }).click();
    
    // Verify redirected to login
    await expect(page).toHaveURL('/login');
    
    // Verify dashboard is no longer accessible
    await page.goto('/dashboard');
    await expect(page).toHaveURL('/login');
  });
});
```

---

## Key Takeaways

1. **Use semantic locators** - `getByRole`, `getByLabel`, `getByAltText`
2. **Never use CSS/XPath** - They're brittle and don't verify accessibility
3. **Embrace web-first assertions** - `await expect()` handles all waiting
4. **Avoid manual waits** - `setTimeout` and `waitForTimeout` are anti-patterns
5. **Test user behavior** - Not implementation details or internal state
6. **Isolate tests** - Each test should be completely independent
7. **Accessibility is free** - Semantic locators automatically verify it
8. **Playwright auto-waits** - Use it; don't fight against it

Follow these principles and your tests will be:
- **Reliable** - No flakiness, no manual waits needed
- **Maintainable** - Refactoring HTML won't break tests
- **Accessible** - Tests verify WCAG compliance
- **Fast** - No arbitrary delays, tests complete when ready
- **Clear** - Tests read like user stories

---

## Debugging Failing Tests

Sometimes your code appears correct, your test appears correct, but the test still fails. This scenario requires a systematic debugging approach rather than guessing.

The key is to **manually replicate what the test does**, then **analyze logs** to identify the root cause. Most failures fall into five categories: Playwright issues (test problems), Code issues (application bugs), Environment issues (setup/config), Timing/race conditions, or Data issues.

**The workflow:**

1. **Manually replicate** - Follow the exact test steps manually in your browser
2. **Analyze logs** - Check Playwright traces, console logs, and network activity
3. **Categorize** - Identify if it's a Playwright, Code, Environment, Timing, or Data issue
4. **Fix** - Apply the appropriate fix for that category
5. **Verify** - Ensure the test passes and there are no side effects

Common patterns you'll encounter:
- "Element not found" but it exists (locator mismatch)
- Timeout waiting for element (timing issue or element slow to appear)
- Text doesn't match (whitespace or dynamic content)
- Click succeeds but nothing happens (preconditions not met)
- Test passes locally but fails in CI (environment difference)
- Test passed yesterday but fails today (flaky test)

For detailed guidance on debugging failing tests, including specific solutions for each pattern and category, see **[DEBUGGING-FAILING-TESTS.md](./DEBUGGING-FAILING-TESTS.md)** which covers:
- The problem scenario and why it happens
- Step-by-step debugging workflow
- How to categorize the issue
- Solutions for 6 common patterns
- When to escalate to the user
- Tools and commands reference
- Complete debugging checklist
