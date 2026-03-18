# Playwright E2E Testing - Quick Start Template

This file provides templates to get you started quickly with your first test file.

## Basic Test Template

```javascript
import { test, expect } from '@playwright/test';

test.describe('Feature Name', () => {
  // Setup run before each test
  test.beforeEach(async ({ page }) => {
    // Navigate to page
    await page.goto('/feature');
    
    // Login if needed
    // await loginAs(page, 'user@example.com');
  });

  // Cleanup run after each test (optional)
  test.afterEach(async ({ page }) => {
    // Clean up test data if needed
    // await deleteTestData(page);
  });

  test('should do something', async ({ page }) => {
    // Arrange: Element is visible
    await expect(page.getByRole('button', { name: 'Action' })).toBeVisible();

    // Act: User interacts
    await page.getByRole('button', { name: 'Action' }).click();

    // Assert: Result is visible
    await expect(page.getByText('Success')).toBeVisible();
  });

  test('should handle error', async ({ page }) => {
    // Setup error condition
    await page.route('**/api/**', route => route.abort());

    // Trigger action
    await page.getByRole('button', { name: 'Save' }).click();

    // Verify error shown
    await expect(page.getByRole('alert')).toContainText('Error');
  });
});
```

---

## Form Testing Template

```javascript
import { test, expect } from '@playwright/test';

test('form submission', async ({ page }) => {
  await page.goto('/form');

  // Fill form using semantic locators
  await page.getByLabel('Full Name').fill('John Doe');
  await page.getByLabel('Email').fill('john@example.com');
  await page.getByLabel('Message').fill('Test message');
  
  // Check optional checkbox
  await page.getByLabel('Subscribe').check();

  // Select dropdown option
  await page.getByLabel('Priority').selectOption('high');

  // Submit form
  await page.getByRole('button', { name: 'Submit' }).click();

  // Verify success
  await expect(page.getByText('Thank you!')).toBeVisible();
  await expect(page).toHaveURL('/success');
});

test('form validation', async ({ page }) => {
  await page.goto('/form');

  // Submit empty form
  await page.getByRole('button', { name: 'Submit' }).click();

  // Verify errors appear
  await expect(page.getByText('Name is required')).toBeVisible();
  await expect(page.getByText('Email is required')).toBeVisible();

  // Fill one field
  await page.getByLabel('Full Name').fill('John');

  // Verify error cleared for that field only
  await expect(page.getByText('Name is required')).not.toBeVisible();
  await expect(page.getByText('Email is required')).toBeVisible();
});
```

---

## Modal Testing Template

```javascript
import { test, expect } from '@playwright/test';

test('modal interactions', async ({ page }) => {
  await page.goto('/items');

  // Open modal
  await page.getByRole('button', { name: 'Add Item' }).click();

  // Modal appears
  const modal = page.getByRole('dialog');
  await expect(modal).toBeVisible();

  // Fill modal form
  await modal.getByLabel('Item Name').fill('New Item');
  await modal.getByLabel('Description').fill('Test description');

  // Submit modal
  await modal.getByRole('button', { name: 'Create' }).click();

  // Modal closes
  await expect(modal).not.toBeVisible();

  // Verify item added
  await expect(page.getByText('New Item')).toBeVisible();
});

test('cancel modal', async ({ page }) => {
  await page.goto('/items');

  // Open modal
  await page.getByRole('button', { name: 'Add Item' }).click();

  // Type something
  const modal = page.getByRole('dialog');
  await modal.getByLabel('Item Name').fill('Temporary');

  // Click cancel
  await modal.getByRole('button', { name: 'Cancel' }).click();

  // Modal closes
  await expect(modal).not.toBeVisible();

  // Item was not added
  await expect(page.getByText('Temporary')).not.toBeVisible();
});

test('escape closes modal', async ({ page }) => {
  await page.goto('/items');

  // Open modal
  await page.getByRole('button', { name: 'Add Item' }).click();

  // Press escape
  await page.keyboard.press('Escape');

  // Modal closes
  await expect(page.getByRole('dialog')).not.toBeVisible();
});
```

---

## List/Table Testing Template

```javascript
import { test, expect } from '@playwright/test';

test('list with filtering', async ({ page }) => {
  await page.goto('/items');

  // Verify initial list
  await expect(page.getByRole('listitem')).toHaveCount(10);

  // Open filter menu
  await page.getByRole('button', { name: 'Filter' }).click();

  // Apply filter
  await page.getByLabel('Category').selectOption('electronics');
  await page.getByRole('button', { name: 'Apply' }).click();

  // Verify filtered results
  await expect(page.getByRole('listitem')).toHaveCount(3);

  // Verify all items match filter
  const items = page.getByRole('listitem');
  for (let i = 0; i < 3; i++) {
    await expect(items.nth(i)).toContainText('Electronics');
  }
});

test('table sorting', async ({ page }) => {
  await page.goto('/table');

  // Click sort button
  await page.getByRole('columnheader', { name: 'Date' }).click();

  // Wait for sort
  await page.waitForLoadState('networkidle');

  // Verify first row is most recent
  const firstRow = page.getByRole('row').nth(1);
  await expect(firstRow).toContainText(new Date().toLocaleDateString());
});

test('pagination', async ({ page }) => {
  await page.goto('/items');

  // Verify on first page
  await expect(page.getByText('Item 1')).toBeVisible();

  // Go to next page
  await page.getByRole('button', { name: 'Next' }).click();

  // Verify on second page
  await expect(page.getByText('Item 1')).not.toBeVisible();
  await expect(page.getByText('Item 11')).toBeVisible();

  // Verify previous button works
  await page.getByRole('button', { name: 'Previous' }).click();
  await expect(page.getByText('Item 1')).toBeVisible();
});
```

---

## Error Handling Template

```javascript
import { test, expect } from '@playwright/test';

test('handles API error', async ({ page }) => {
  // Mock API to return error
  await page.route('**/api/data', route => {
    route.abort('failed');
  });

  await page.goto('/data');

  // Trigger request
  await page.getByRole('button', { name: 'Load' }).click();

  // Verify error shown
  const alert = page.getByRole('alert');
  await expect(alert).toBeVisible();
  await expect(alert).toContainText('Failed to load');

  // Retry button available
  const retryBtn = page.getByRole('button', { name: 'Retry' });
  await expect(retryBtn).toBeVisible();
});

test('handles timeout', async ({ page }) => {
  // Slow down request
  await page.route('**/api/slow', route => {
    setTimeout(() => route.continue(), 15000);
  });

  await page.goto('/data');
  await page.getByRole('button', { name: 'Load' }).click();

  // Verify timeout error
  await expect(page.getByRole('alert')).toContainText('timeout');
});

test('validation errors', async ({ page }) => {
  await page.goto('/form');

  // Submit invalid data
  await page.getByLabel('Email').fill('not-an-email');
  await page.getByRole('button', { name: 'Submit' }).click();

  // Error shows on specific field
  const emailField = page.getByLabel('Email');
  await expect(emailField).toHaveClass(/error/);
  
  // Error message appears
  await expect(page.getByText('Invalid email format')).toBeVisible();
});
```

---

## Complex Interactions Template

```javascript
import { test, expect } from '@playwright/test';

test('drag and drop', async ({ page }) => {
  await page.goto('/tasks');

  // Get items
  const items = page.getByRole('listitem');
  const first = items.nth(0);
  const second = items.nth(1);

  // Drag first to second position
  await first.dragTo(second);

  // Verify order changed
  const newFirst = items.nth(0);
  const newSecond = items.nth(1);
  
  const firstText = await first.innerText();
  const secondText = await second.innerText();
  const newFirstText = await newFirst.innerText();
  
  expect(newFirstText).toBe(secondText);
});

test('file upload', async ({ page }) => {
  await page.goto('/upload');

  // Set file
  const fileInput = page.getByLabel('Choose file');
  await fileInput.setInputFiles('/path/to/file.pdf');

  // Verify file shown
  await expect(page.getByText('file.pdf')).toBeVisible();

  // Submit
  await page.getByRole('button', { name: 'Upload' }).click();

  // Verify success
  await expect(page.getByText('Uploaded')).toBeVisible();
});

test('keyboard navigation', async ({ page }) => {
  await page.goto('/form');

  // Start at first field
  const nameInput = page.getByLabel('Name');
  await nameInput.focus();
  await expect(nameInput).toBeFocused();

  // Tab to next field
  await page.keyboard.press('Tab');
  await expect(page.getByLabel('Email')).toBeFocused();

  // Shift+Tab to go back
  await page.keyboard.press('Shift+Tab');
  await expect(nameInput).toBeFocused();

  // Type in field
  await page.keyboard.type('John Doe');
  await expect(nameInput).toHaveValue('John Doe');

  // Tab to button and press Enter
  await page.keyboard.press('Tab');
  await page.keyboard.press('Tab');
  const submitBtn = page.getByRole('button', { name: 'Submit' });
  await expect(submitBtn).toBeFocused();
  await page.keyboard.press('Enter');

  // Verify submission
  await expect(page.getByText('Success')).toBeVisible();
});
```

---

## With Custom Fixtures Template

```javascript
import { test as base, expect } from '@playwright/test';

// Create custom fixture for authenticated user
const test = base.extend({
  authenticatedPage: async ({ page }, use) => {
    // Setup: Login
    await page.goto('/login');
    await page.getByLabel('Email').fill('testuser@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Sign In' }).click();
    await expect(page).toHaveURL('/dashboard');
    
    // Use in test
    await use(page);
    
    // Cleanup: Logout
    await page.getByRole('button', { name: 'Logout' }).click();
  },
});

test('authenticated user can edit profile', async ({ authenticatedPage: page }) => {
  // No need to login - fixture handles it
  await page.goto('/profile/edit');
  
  await page.getByLabel('Name').fill('John Doe');
  await page.getByRole('button', { name: 'Save' }).click();
  
  await expect(page.getByText('Saved')).toBeVisible();
});

test('authenticated user can view dashboard', async ({ authenticatedPage: page }) => {
  // No need to login - fixture handles it
  await page.goto('/dashboard');
  
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible();
});
```

---

## Configuration Template (playwright.config.ts)

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  
  reporter: 'html',
  
  use: {
    // Base URL for all tests
    baseURL: 'http://localhost:3000',
    
    // Take screenshot on failure
    screenshot: 'only-on-failure',
    
    // Record video on failure
    video: 'retain-on-failure',
    
    // Trace on failure
    trace: 'on-first-retry',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
  ],

  // Web Server
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

---

## Common Helper Functions

```javascript
// Login helper
async function loginAs(page, email, password = 'password123') {
  await page.goto('/login');
  await page.getByLabel('Email').fill(email);
  await page.getByLabel('Password').fill(password);
  await page.getByRole('button', { name: 'Sign In' }).click();
  await expect(page).toHaveURL('/dashboard');
}

// Create test user (via API)
async function createTestUser(page, email) {
  const response = await page.request.post('/api/users', {
    data: { email, password: 'test123' },
  });
  return await response.json();
}

// Delete test data (via API for speed)
async function deleteTestData(page) {
  await page.request.delete('/api/test-data');
}

// Fill form
async function fillForm(page, data) {
  for (const [label, value] of Object.entries(data)) {
    await page.getByLabel(label).fill(value);
  }
}

// Get table row by text
function getTableRowByText(page, text) {
  return page.getByRole('row').filter({ has: page.getByText(text) });
}

// Wait for network idle
async function waitForData(page) {
  await page.waitForLoadState('networkidle');
}
```

---

## Best Practices Checklist

Before submitting tests, verify:

- ✅ Using `await expect()` not `expect()`
- ✅ Using `getByRole()` / `getByLabel()` not CSS/XPath
- ✅ Each test is independent (no shared state)
- ✅ Using `beforeEach()` for common setup
- ✅ Testing user behavior (not implementation)
- ✅ No `setTimeout` or `waitForTimeout`
- ✅ Clear test names describing what is tested
- ✅ Using `Arrange-Act-Assert` pattern
- ✅ Proper error handling tested
- ✅ Accessibility verified (labels, roles, ARIA)

---

## Running Tests

```bash
# Run all tests
npx playwright test

# Run specific file
npx playwright test tests/auth.spec.ts

# Run specific test
npx playwright test -g "can login"

# Run in UI mode
npx playwright test --ui

# Run in headed mode (see browser)
npx playwright test --headed

# Debug mode
npx playwright test --debug

# View HTML report
npx playwright show-report
```

---

## Next Steps

1. Copy one of these templates
2. Replace example with your actual app
3. Run tests: `npx playwright test`
4. Check EXAMPLES.md for specific patterns you need
5. Verify against ANTI-PATTERNS.md
6. Expand test suite with more tests

For detailed guidance, see SKILL.md and EXAMPLES.md.
