# Common Playwright E2E Testing Anti-Patterns to Avoid

This guide highlights the most common mistakes when writing Playwright tests and why they're problematic.

---

## Anti-Pattern #1: Manual Waits (setTimeout)

### The Problem

```javascript
// ❌ ANTI-PATTERN: Using setTimeout
test('loads user data', async ({ page }) => {
  await page.goto('/user/123');
  
  // "Wait for data to load" (arbitrary!)
  await new Promise(r => setTimeout(r, 2000));
  
  // Check if data loaded
  expect(await page.locator('.user-name').textContent()).toBe('John Doe');
});
```

### Why It's Harmful

1. **Tests become slow** - You're waiting the full timeout even if data loads in 100ms
2. **False reliability** - Arbitrary delays don't guarantee synchronization
3. **Flaky in CI** - What works locally may fail in slow CI environments
4. **Hard to debug** - When tests fail, you don't know if it's the wait or the test
5. **Maintenance burden** - Need to adjust timeouts as app changes
6. **Race conditions** - Doesn't actually ensure the condition you care about

### The Right Way

```javascript
// ✅ CORRECT: Web-first assertion with auto-wait
test('loads user data', async ({ page }) => {
  await page.goto('/user/123');
  
  // Waits automatically until element appears (up to 5 seconds)
  // Or fails immediately if never appears
  await expect(page.getByText('John Doe')).toBeVisible();
});
```

**Why this works:**
- Passes as soon as data loads (fast!)
- Waits up to 5 seconds if data is slow
- Fails immediately and clearly if data never loads
- Works consistently in all environments

---

## Anti-Pattern #2: Using waitForTimeout/waitForSelector

### The Problem

```javascript
// ❌ ANTI-PATTERN: Explicit wait
test('modal closes', async ({ page }) => {
  await page.getByRole('button', { name: 'Delete' }).click();
  
  // Wait for modal to appear
  await page.waitForSelector('.modal-content');
  
  // Wait for it to close (arbitrary timeout)
  await page.waitForTimeout(3000);
  
  // Hope the modal is gone
  expect(await page.locator('.modal-content').isVisible()).toBe(false);
});
```

### Why It's Harmful

1. **Slow** - Waits full 3 seconds even if modal closes in 100ms
2. **Unreliable** - Timeout might not be enough in slow environments
3. **Not checking for the right thing** - Just waiting, not verifying condition
4. **Poor error messages** - Hard to know what failed

### The Right Way

```javascript
// ✅ CORRECT: Wait for specific condition
test('modal closes', async ({ page }) => {
  await page.getByRole('button', { name: 'Delete' }).click();
  
  // Wait for modal to appear
  const modal = page.getByRole('dialog');
  await expect(modal).toBeVisible();
  
  // Confirm delete
  await modal.getByRole('button', { name: 'Confirm' }).click();
  
  // Wait for modal to disappear
  await expect(modal).toBeHidden();
  // ^ Waits until actually hidden, no arbitrary delay
});
```

**Why this works:**
- Verifies modal actually appears
- Verifies modal actually closes
- Instant when conditions are met
- Clear error if modal doesn't appear/close

---

## Anti-Pattern #3: Test IDs Everywhere

### The Problem

```javascript
// ❌ ANTI-PATTERN: Over-relying on test IDs
<form>
  <input data-testid="email-input" />
  <button data-testid="submit-button">Submit</button>
</form>

test('submit form', async ({ page }) => {
  await page.getByTestId('email-input').fill('test@example.com');
  await page.getByTestId('submit-button').click();
  await expect(page.getByTestId('success-message')).toBeVisible();
});
```

### Why It's Harmful

1. **Test IDs are implementation details** - Tests don't verify accessibility
2. **HTML structure can be wrong** - Test IDs can exist on non-semantic elements
3. **Hides accessibility issues** - App might work for tests but fail for screen readers
4. **Maintenance burden** - Need to add test IDs throughout codebase
5. **Doesn't verify what users see** - Could be hidden, disabled, or covered

### The Right Way

```javascript
// ✅ CORRECT: Use semantic locators
<form>
  <label for="email">Email</label>
  <input id="email" type="email" />
  <button type="submit">Submit</button>
</form>

test('submit form', async ({ page }) => {
  // Tests verify: proper HTML structure, accessibility, and user interaction
  await page.getByLabel('Email').fill('test@example.com');
  await page.getByRole('button', { name: 'Submit' }).click();
  
  // If status message is important, give it proper ARIA role
  await expect(page.getByRole('status')).toContainText('Success');
});
```

**Why this works:**
- Verifies HTML is properly structured
- Verifies inputs have labels
- Verifies buttons are actual buttons
- Accessible to screen readers
- Tests break when accessibility breaks

---

## Anti-Pattern #4: CSS Selectors and XPath Chains

### The Problem

```javascript
// ❌ TERRIBLE: Brittle CSS selectors
test('add item to cart', async ({ page }) => {
  await page.locator('div.container > div.products-grid > div:nth-child(3) button.buy').click();
  await page.locator('form#checkout-form input[name="quantity"]').fill('2');
  await page.locator('//div[@class="modal-footer"]//button[@class="primary"]').click();
});

// Same problems with XPath
test('checkout', async ({ page }) => {
  await page.locator('//form[@id="checkout"]//*[@name="email"]').fill('test@example.com');
  await page.locator('//button[contains(text(), "Pay Now")]').click();
});
```

### Why It's Harmful

1. **Brittle selectors** - Break with any CSS refactor
2. **Deep coupling** - Changes to DOM structure break tests
3. **No accessibility verification** - Can select hidden/disabled elements
4. **Hard to maintain** - Selectors are hard to understand
5. **No semantic meaning** - Doesn't verify proper HTML structure
6. **Invisible elements selected** - CSS selectors don't check visibility

### Real Example of Brittleness

```javascript
// Test passes locally
await page.locator('div.form > div:nth-child(2) input').fill('test');

// Then designer adds a hidden loading spinner (also a div)
// Now your selector targets the wrong element!
// Or the test fails because DOM structure changed
```

### The Right Way

```javascript
// ✅ CORRECT: Semantic, accessible locators
test('add item to cart', async ({ page }) => {
  // Find button by what it says and that it's a button
  const product = page.getByRole('article').filter({ 
    has: page.getByText('Product Name') 
  });
  await product.getByRole('button', { name: 'Buy' }).click();
  
  // Input accessed by label
  await page.getByLabel('Quantity').fill('2');
  
  // Button by role and name
  await page.getByRole('button', { name: 'Complete Purchase' }).click();
});

test('checkout', async ({ page }) => {
  // Form input by label
  await page.getByLabel('Email Address').fill('test@example.com');
  
  // Button by role and visible text
  await page.getByRole('button', { name: 'Pay Now' }).click();
});
```

**Why this works:**
- Survives CSS refactoring
- Verifies HTML structure is correct
- Verifies elements are interactive (visible, enabled)
- Clear, maintainable selectors

---

## Anti-Pattern #5: Not Using Web-First Assertions

### The Problem

```javascript
// ❌ ANTI-PATTERN: Immediate assertions
test('user logs in', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: 'Sign In' }).click();
  
  // Check immediately - might not be redirected yet!
  const url = page.url();
  expect(url).toContain('/dashboard');
  
  // Check immediately - success message might not be loaded yet!
  const text = await page.locator('body').textContent();
  expect(text).toContain('Welcome back');
});
```

### Why It's Harmful

1. **Race conditions** - Checks before navigation/content complete
2. **Flaky tests** - Pass locally but fail in CI
3. **Timing dependent** - Tests fail randomly
4. **Hard to debug** - Don't know if check failed or timing failed
5. **No automatic retry** - Fails on first check

### The Right Way

```javascript
// ✅ CORRECT: Web-first assertions with auto-retry
test('user logs in', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('password123');
  await page.getByRole('button', { name: 'Sign In' }).click();
  
  // Waits for URL change (with retry)
  // Passes when redirected, fails if never redirects
  await expect(page).toHaveURL('/dashboard');
  
  // Waits for text to appear (with retry)
  // Passes when text appears, fails if never appears
  await expect(page.getByText('Welcome back')).toBeVisible();
});
```

**Why this works:**
- `await expect()` automatically retries for up to 5 seconds
- Passes as soon as condition is true
- Fails clearly if condition never becomes true
- No arbitrary waits

---

## Anti-Pattern #6: Testing Implementation Details

### The Problem

```javascript
// ❌ ANTI-PATTERN: Testing internal state/props
test('counter increases', async ({ page }) => {
  await page.goto('/counter');
  
  // Testing component state, not user behavior
  let state = await page.evaluate(() => {
    return window.__componentState.counter.count;
  });
  expect(state).toBe(0);
  
  // Testing React props
  await page.getByRole('button', { name: 'Increment' }).click();
  state = await page.evaluate(() => {
    return window.__componentState.counter.count;
  });
  expect(state).toBe(1);
});

// Testing localStorage directly
test('user preferences save', async ({ page }) => {
  const prefs = await page.evaluate(() => {
    return JSON.parse(localStorage.getItem('userPrefs'));
  });
  expect(prefs.theme).toBe('dark');
});
```

### Why It's Harmful

1. **Breaks when refactoring** - Internal structure changes break tests
2. **Doesn't test real behavior** - What users see might be different
3. **Tests become brittle** - Tightly coupled to implementation
4. **Not testing user workflows** - Missing real use cases
5. **False confidence** - Tests pass but app might be broken
6. **Accessibility ignored** - Internal state tests don't verify accessibility

### Real Example

```javascript
// Test passes because internal state is correct
// But button is actually hidden!
const state = await page.evaluate(() => window.__state.saved);
expect(state).toBe(true); // ✅ Test passes

// User can't see the success message!
// Real test would catch this:
await expect(page.getByText('Saved')).toBeVisible(); // ❌ Test fails
```

### The Right Way

```javascript
// ✅ CORRECT: Test user behavior
test('counter increases', async ({ page }) => {
  await page.goto('/counter');
  
  // Verify what user sees
  await expect(page.getByText('Count: 0')).toBeVisible();
  
  // User clicks button
  await page.getByRole('button', { name: 'Increment' }).click();
  
  // User sees updated count
  await expect(page.getByText('Count: 1')).toBeVisible();
});

// Test preferences through user interaction
test('user can change theme', async ({ page }) => {
  await page.goto('/settings');
  
  // User sees current theme
  const htmlTag = page.locator('html');
  await expect(htmlTag).toHaveAttribute('data-theme', 'light');
  
  // User changes theme
  await page.getByLabel('Dark mode').check();
  
  // User sees it updated
  await expect(htmlTag).toHaveAttribute('data-theme', 'dark');
  
  // Persistence verified by reload
  await page.reload();
  await expect(htmlTag).toHaveAttribute('data-theme', 'dark');
});
```

**Why this works:**
- Tests actual user workflows
- Breaks when real functionality breaks
- Survives refactoring (internal state changes)
- Tests both functionality AND UI

---

## Anti-Pattern #7: Flaky Selectors That Break Easily

### The Problem

```javascript
// ❌ ANTI-PATTERN: Selectors that break with small changes
test('can submit form', async ({ page }) => {
  // If heading text changes, test breaks
  await page.locator('h1').fill('input[name="email"]', 'test@example.com');
  
  // If someone adds a span inside the button, selector breaks
  await page.locator('button:has(span)').click();
  
  // If error message text changes, test breaks
  await expect(page.getByText('Form submitted successfully')).toBeVisible();
});

// Even worse: brittle text selectors
await page.getByText('Click the button below:').click(); // VERY brittle
```

### Why It's Harmful

1. **Maintainability nightmare** - Small changes break many tests
2. **False negatives** - Tests fail for wrong reasons
3. **High maintenance cost** - Constant test updates
4. **Brittle text** - Text changes break tests

### Comparison Table

```javascript
// ❌ BRITTLE
await page.getByText('Click Here').click()
// Breaks if text changes to "Click Here Now"

await page.getByText('Form submitted successfully').toBeVisible()
// Breaks if message changes to "Success! Form was submitted"

await page.locator('button:nth-child(3)').click()
// Breaks if a new button is added

// ✅ RESILIENT
await page.getByRole('button', { name: 'Submit' }).click()
// Survives: text changes to "Submit Form", styling changes, etc.

await expect(page.getByRole('status')).toContainText('submitted')
// Survives: exact message changes, as long as it contains "submitted"

await page.getByRole('button', { name: 'Save' }).click()
// Works regardless of where button is in DOM
```

### The Right Way

```javascript
// ✅ CORRECT: Resilient selectors
test('can submit form', async ({ page }) => {
  // Use semantic locators that survive changes
  await page.getByLabel('Email').fill('test@example.com');
  
  // Find by role and name, not text or position
  await page.getByRole('button', { name: 'Submit' }).click();
  
  // Use partial text match for messages
  await expect(page.getByRole('status')).toContainText('submitted');
});
```

**Why this works:**
- Survives text changes (using role + name)
- Survives CSS changes (not using nth-child)
- Survives layout changes (not using position)
- Resilient to refactoring

---

## Anti-Pattern #8: Test Interdependence

### The Problem

```javascript
// ❌ ANTI-PATTERN: Tests depend on each other
test('create user', async ({ page }) => {
  await page.goto('/admin/users');
  await page.getByRole('button', { name: 'Add User' }).click();
  // ... creates "testuser@example.com"
});

test('edit user', async ({ page }) => {
  // Assumes previous test created the user!
  await page.goto('/admin/users');
  const row = page.getByRole('row').filter({ 
    has: page.getByText('testuser@example.com') 
  });
  // ... test fails if "create user" test doesn't run first
});

test('delete user', async ({ page }) => {
  // Depends on both previous tests!
  // If they run in different order, fails
});
```

### Why It's Harmful

1. **Fragile** - Tests fail if run in different order
2. **Hard to debug** - Failures cascade from first test
3. **Slow** - Tests must run sequentially
4. **Unreliable in parallel** - CI test runners often run in parallel
5. **Bad isolation** - Can't run single test independently

### The Right Way

```javascript
// ✅ CORRECT: Each test is independent
test.beforeEach(async ({ page }) => {
  // Every test starts with clean setup
  await loginAs(page, 'admin@example.com');
});

test('create user', async ({ page }) => {
  // Set up user creation
  await page.goto('/admin/users');
  await page.getByRole('button', { name: 'Add User' }).click();
  
  // ... creates user
  // This test is now independent
});

test('edit user', async ({ page }) => {
  // Set up user for this test independently
  const userId = await createTestUser(page, 'testuser@example.com');
  
  // Now test editing it
  await page.goto(`/admin/users/${userId}/edit`);
  // ... test can run independently
});

test('delete user', async ({ page }) => {
  // Set up user for this test independently
  const userId = await createTestUser(page, 'testuser@example.com');
  
  // Now test deleting it
  await page.goto(`/admin/users/${userId}`);
  // ... test can run independently
});

// Helper function (can use API for speed)
async function createTestUser(page, email) {
  const response = await page.request.post('/api/admin/users', {
    data: { email, password: 'test123' }
  });
  const user = await response.json();
  return user.id;
}
```

**Why this works:**
- Each test runs independently
- Can run tests in any order
- Can run single test without running others
- Supports parallel test execution
- Faster (API calls for setup instead of UI)

---

## Anti-Pattern #9: Ignoring Errors and Exceptions

### The Problem

```javascript
// ❌ ANTI-PATTERN: Silently ignoring errors
test('submit form', async ({ page }) => {
  try {
    await page.getByRole('button', { name: 'Submit' }).click();
  } catch (e) {
    // Swallowing the error!
    console.log('Error:', e);
  }
  
  // Test passes even though click failed!
  expect(true).toBe(true);
});

// Ignoring assertion failures
test('loads data', async ({ page }) => {
  try {
    await expect(page.getByText('Data loaded')).toBeVisible();
  } catch {
    // Test continues even though data didn't load!
  }
  
  // This line runs even if assertion failed
  await page.getByRole('button', { name: 'Next' }).click();
});
```

### Why It's Harmful

1. **False positive tests** - Tests pass when they should fail
2. **Masking bugs** - Real issues go undetected
3. **Hard to debug** - Don't know what's actually failing
4. **Invalid test results** - Test report is unreliable

### The Right Way

```javascript
// ✅ CORRECT: Let errors propagate
test('submit form', async ({ page }) => {
  // No try-catch - let it fail if something is wrong
  await page.getByRole('button', { name: 'Submit' }).click();
  
  // If click failed, test stops here and reports the error
  await expect(page.getByText('Thank you')).toBeVisible();
});

// Only catch errors if you're testing error handling
test('handles submission error', async ({ page }) => {
  // Simulate API error
  await page.route('**/api/submit', route => route.abort());
  
  // Try to submit - this will fail
  await page.getByRole('button', { name: 'Submit' }).click();
  
  // Expect error message to appear
  await expect(page.getByRole('alert')).toContainText('Failed to submit');
});
```

**Why this works:**
- Errors are reported clearly
- Tests fail when they should
- Easy to debug failures
- No hidden issues

---

## Anti-Pattern #10: Using page.locator Without Checking Visibility

### The Problem

```javascript
// ❌ ANTI-PATTERN: Selecting elements without checking visibility
test('can delete item', async ({ page }) => {
  // Gets delete button even if hidden off-screen
  const deleteBtn = page.locator('button.delete');
  
  // Clicking might fail or do nothing
  await deleteBtn.click();
});

// Not checking if element is actually interactive
test('fill hidden field', async ({ page }) => {
  // This might succeed even if field is display:none
  await page.locator('input[name="secret"]').fill('value');
  
  // Filed wasn't actually filled for user
});
```

### Why It's Harmful

1. **Not testing real user interactions** - User can't interact with hidden elements
2. **Tests pass for wrong reasons** - Element exists but isn't usable
3. **Flaky in different environments** - Visibility depends on screen size, browser
4. **False confidence** - Tests pass but feature doesn't work

### The Right Way

```javascript
// ✅ CORRECT: Playwright actions verify actionability
test('can delete item', async ({ page }) => {
  // Playwright automatically:
  // 1. Waits for element to exist
  // 2. Waits for element to be visible
  // 3. Waits for element to not be covered
  // 4. Waits for element to be enabled
  // 5. Only then clicks
  await page.getByRole('button', { name: 'Delete' }).click();
  // ^ Fails clearly if button is not clickable
});

// Web-first assertions also check visibility
test('displays feedback', async ({ page }) => {
  await page.getByRole('button', { name: 'Submit' }).click();
  
  // Fails if message is hidden (display:none, opacity:0, etc)
  await expect(page.getByText('Submitted!')).toBeVisible();
  // ^ Won't pass for hidden elements
});
```

**Why this works:**
- Actions verify element is actually clickable
- Assertions verify element is actually visible
- Tests fail if element can't be used by real users
- Better error messages

---

## Summary: Red Flags in Your Tests

If you see any of these patterns, refactor immediately:

```javascript
// 🚩 setTimeout or waitForTimeout
await page.waitForTimeout(1000);

// 🚩 waitForSelector or waitForFunction
await page.waitForSelector('.message');

// 🚩 CSS selector chains
page.locator('div.modal > form > input:nth-child(2)')

// 🚩 XPath
page.locator('//div[@class="item"]')

// 🚩 Immediate assertions (not awaited)
const text = await element.textContent();
expect(text).toContain('...');

// 🚩 Testing internal state
window.__state.value

// 🚩 Try-catch swallowing errors
try { await ...; } catch(e) { }

// 🚩 Tests depending on other tests
// Test relies on global state from previous test

// 🚩 Over-using test IDs
data-testid="..."  (when semantic option exists)

// 🚩 Not isolating tests
// No beforeEach, shared database state
```

**Replace with:**
- `await expect()` web-first assertions
- `getByRole()`, `getByLabel()`, `getByAltText()`
- Let errors fail the test (don't catch)
- Independent test setup in `beforeEach()`
- Semantic selectors, never CSS/XPath
- Test actual user behavior
