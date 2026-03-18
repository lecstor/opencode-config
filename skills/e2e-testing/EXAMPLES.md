# Playwright E2E Testing Examples

Practical, production-ready examples following React Testing Library best practices.

---

## Form Interactions

### Simple Form with Validation

```javascript
import { test, expect } from '@playwright/test';

test('submit contact form with validation', async ({ page }) => {
  await page.goto('/contact');

  // Verify form is visible
  await expect(page.getByRole('heading', { name: 'Contact Us' })).toBeVisible();

  // Fill name field using getByLabel
  await page.getByLabel('Full Name').fill('John Doe');

  // Fill email field
  await page.getByLabel('Email Address').fill('john@example.com');

  // Fill message textarea
  await page.getByLabel('Message').fill('I have a question about your product');

  // Submit form using getByRole for button
  await page.getByRole('button', { name: 'Send Message' }).click();

  // Verify success message
  await expect(page.getByText('Thank you! We received your message.')).toBeVisible();

  // Verify form is cleared or redirected
  await expect(page).toHaveURL('/contact/thank-you');
});
```

### Form Validation Errors

```javascript
test('displays validation errors', async ({ page }) => {
  await page.goto('/signup');

  // Try to submit empty form
  await page.getByRole('button', { name: 'Create Account' }).click();

  // Verify error messages appear
  await expect(page.getByText('Email is required')).toBeVisible();
  await expect(page.getByText('Password is required')).toBeVisible();

  // Fill only email
  await page.getByLabel('Email').fill('john@example.com');
  await page.getByRole('button', { name: 'Create Account' }).click();

  // Only password error should remain
  await expect(page.getByText('Email is required')).not.toBeVisible();
  await expect(page.getByText('Password is required')).toBeVisible();

  // Fill password
  await page.getByLabel('Password').fill('SecurePassword123');
  await page.getByRole('button', { name: 'Create Account' }).click();

  // All errors gone, form submits
  await expect(page.getByText('Account created successfully')).toBeVisible();
});
```

### Checkbox and Radio Interactions

```javascript
test('form with checkboxes and radio buttons', async ({ page }) => {
  await page.goto('/preferences');

  // Check multiple checkboxes
  await page.getByLabel('Email Notifications').check();
  await page.getByLabel('SMS Notifications').check();
  await expect(page.getByLabel('Email Notifications')).toBeChecked();
  await expect(page.getByLabel('SMS Notifications')).toBeChecked();

  // Verify unchecked by default
  await expect(page.getByLabel('Push Notifications')).not.toBeChecked();

  // Select radio button option
  await page.getByLabel('Immediate').check();
  await expect(page.getByLabel('Immediate')).toBeChecked();

  // Radio buttons are mutually exclusive
  await page.getByLabel('Daily Digest').check();
  await expect(page.getByLabel('Daily Digest')).toBeChecked();
  await expect(page.getByLabel('Immediate')).not.toBeChecked();

  // Save preferences
  await page.getByRole('button', { name: 'Save Preferences' }).click();
  await expect(page.getByText('Preferences saved')).toBeVisible();
});
```

### Select Dropdown

```javascript
test('select from dropdown', async ({ page }) => {
  await page.goto('/billing');

  // Select option from dropdown
  await page.getByLabel('Billing Cycle').selectOption('annual');

  // Verify selection
  await expect(page.getByLabel('Billing Cycle')).toHaveValue('annual');

  // Verify price updated (side effect)
  await expect(page.getByText('$99/year')).toBeVisible();

  // Change selection
  await page.getByLabel('Billing Cycle').selectOption('monthly');
  await expect(page.getByLabel('Billing Cycle')).toHaveValue('monthly');
  await expect(page.getByText('$9.99/month')).toBeVisible();
});
```

### Password Field with Show/Hide Toggle

```javascript
test('password visibility toggle', async ({ page }) => {
  await page.goto('/login');

  const passwordField = page.getByLabel('Password');

  // Password is hidden by default
  await expect(passwordField).toHaveAttribute('type', 'password');

  // Type password
  await passwordField.fill('MySecretPassword123');

  // Click show password button/toggle
  await page.getByRole('button', { name: 'Show password' }).click();

  // Password field is now text type
  await expect(passwordField).toHaveAttribute('type', 'text');

  // Value is still there
  await expect(passwordField).toHaveValue('MySecretPassword123');

  // Toggle back to hidden
  await page.getByRole('button', { name: 'Hide password' }).click();
  await expect(passwordField).toHaveAttribute('type', 'password');
});
```

---

## List and Table Interactions

### Filtering a List

```javascript
test('filter product list', async ({ page }) => {
  await page.goto('/products');

  // Verify initial list has items
  await expect(page.getByRole('listitem')).toHaveCount(12);

  // Open filter menu
  await page.getByRole('button', { name: 'Filters' }).click();

  // Select filter options using getByLabel
  await page.getByLabel('Category').selectOption('Electronics');
  await page.getByLabel('Price Range').selectOption('100-500');

  // Apply filters using button role
  await page.getByRole('button', { name: 'Apply Filters' }).click();

  // Verify results are filtered
  await expect(page.getByRole('listitem')).toHaveCount(4);

  // Verify all items are in correct category
  const items = page.getByRole('listitem');
  await expect(items.first()).toContainText('Electronics');
});
```

### Sorting a List

```javascript
test('sort table by column', async ({ page }) => {
  await page.goto('/orders');

  // Get all order numbers before sorting
  const ordersBefore = await page
    .getByRole('cell', { name: /Order #/ })
    .allTextContents();

  // Click sort button for Date column
  await page.getByRole('columnheader', { name: 'Date' }).click();

  // Wait for sorting animation
  await page.waitForLoadState('networkidle');

  // Get order numbers after sorting
  const ordersAfter = await page
    .getByRole('cell', { name: /Order #/ })
    .allTextContents();

  // Verify order is different (sorted)
  expect(ordersBefore).not.toEqual(ordersAfter);

  // Verify first order is from most recent date
  const firstRow = page.getByRole('row').nth(1);
  await expect(firstRow).toContainText(new Date().toLocaleDateString());
});
```

### Pagination

```javascript
test('navigate table pages', async ({ page }) => {
  await page.goto('/reports');

  // Verify first page content
  await expect(page.getByRole('cell')).toContainText('Report 1');

  // Verify next button is enabled
  const nextButton = page.getByRole('button', { name: 'Next' });
  await expect(nextButton).toBeEnabled();

  // Click next page
  await nextButton.click();

  // Wait for new page to load
  await expect(page.getByRole('cell')).toContainText('Report 21');

  // Previous button should be enabled
  const prevButton = page.getByRole('button', { name: 'Previous' });
  await expect(prevButton).toBeEnabled();

  // Go to last page
  await page.getByRole('button', { name: 'Go to page 5' }).click();
  await expect(page.getByRole('cell')).toContainText('Report 101');

  // Next button should be disabled on last page
  await expect(nextButton).toBeDisabled();
});
```

### Editable Table Cells

```javascript
test('edit table cell inline', async ({ page }) => {
  await page.goto('/contacts');

  // Find specific row using getByRole
  const row = page
    .getByRole('row')
    .filter({ has: page.getByText('John Doe') });

  // Double-click cell to edit
  await row.getByRole('cell').nth(2).dblclick();

  // Input field appears
  const input = row.getByRole('textbox');
  await expect(input).toBeVisible();

  // Clear and enter new value
  await input.clear();
  await input.fill('john.doe@example.com');

  // Press Enter to save
  await input.press('Enter');

  // Verify change is saved
  await expect(row.getByRole('cell').nth(2)).toContainText('john.doe@example.com');

  // Verify save feedback
  await expect(page.getByText('Contact updated')).toBeVisible();
});
```

---

## Modal and Dialog Interactions

### Confirmation Dialog

```javascript
test('confirm delete action', async ({ page }) => {
  await page.goto('/items');

  // Find and click delete button
  const row = page.getByRole('row').filter({ has: page.getByText('Item to Delete') });
  await row.getByRole('button', { name: 'Delete' }).click();

  // Dialog appears
  const dialog = page.getByRole('alertdialog');
  await expect(dialog).toBeVisible();
  await expect(dialog).toContainText('Are you sure you want to delete this item?');

  // Cancel button
  await dialog.getByRole('button', { name: 'Cancel' }).click();
  await expect(dialog).not.toBeVisible();

  // Item still exists
  await expect(page.getByText('Item to Delete')).toBeVisible();

  // Delete again and confirm
  await row.getByRole('button', { name: 'Delete' }).click();
  await expect(dialog).toBeVisible();

  // Click confirm
  await dialog.getByRole('button', { name: 'Delete' }).click();

  // Dialog closes
  await expect(dialog).not.toBeVisible();

  // Item is removed
  await expect(page.getByText('Item to Delete')).not.toBeVisible();

  // Success message
  await expect(page.getByText('Item deleted successfully')).toBeVisible();
});
```

### Form Modal

```javascript
test('create new user via modal form', async ({ page }) => {
  await page.goto('/users');

  // Open modal via button
  await page.getByRole('button', { name: 'Add User' }).click();

  // Modal dialog appears
  const modal = page.getByRole('dialog');
  await expect(modal).toBeVisible();

  // Fill form in modal using getByLabel
  const nameInput = modal.getByLabel('Full Name');
  await expect(nameInput).toBeFocused(); // First field should be focused

  await nameInput.fill('Jane Smith');
  await modal.getByLabel('Email').fill('jane@example.com');
  await modal.getByLabel('Department').selectOption('engineering');

  // Submit form
  await modal.getByRole('button', { name: 'Create' }).click();

  // Modal closes
  await expect(modal).not.toBeVisible();

  // New user appears in list
  const row = page
    .getByRole('row')
    .filter({ has: page.getByText('jane@example.com') });
  await expect(row).toBeVisible();
  await expect(row).toContainText('Jane Smith');
  await expect(row).toContainText('Engineering');
});
```

### Modal with Tabs

```javascript
test('navigate tabs in modal', async ({ page }) => {
  await page.goto('/settings');

  // Open settings modal
  await page.getByRole('button', { name: 'Advanced Settings' }).click();

  const modal = page.getByRole('dialog');

  // Verify first tab is selected
  const tablist = modal.getByRole('tablist');
  await expect(tablist).toBeVisible();

  const generalTab = modal.getByRole('tab', { name: 'General' });
  await expect(generalTab).toHaveAttribute('aria-selected', 'true');

  // Verify first panel is visible
  await expect(modal.getByRole('tabpanel', { name: 'General' })).toBeVisible();

  // Click second tab
  await modal.getByRole('tab', { name: 'Advanced' }).click();

  // First tab is no longer selected
  await expect(generalTab).toHaveAttribute('aria-selected', 'false');

  // Second tab is selected
  const advancedTab = modal.getByRole('tab', { name: 'Advanced' });
  await expect(advancedTab).toHaveAttribute('aria-selected', 'true');

  // Second panel is visible
  await expect(modal.getByRole('tabpanel', { name: 'Advanced' })).toBeVisible();
});
```

---

## Error State Testing

### API Error Handling

```javascript
test('display error message on API failure', async ({ page }) => {
  await page.goto('/dashboard');

  // Mock API to return error
  await page.route('**/api/analytics/**', route => {
    route.abort('failed');
  });

  // Trigger data fetch
  await page.getByRole('button', { name: 'Refresh Analytics' }).click();

  // Error message appears
  const errorMessage = page.getByRole('alert');
  await expect(errorMessage).toBeVisible();
  await expect(errorMessage).toContainText('Failed to load analytics');

  // Error icon is visible
  await expect(page.getByAltText('Error icon')).toBeVisible();

  // Retry button is visible
  const retryButton = page.getByRole('button', { name: 'Retry' });
  await expect(retryButton).toBeVisible();

  // Unmock and retry
  await page.unroute('**/api/analytics/**');
  await retryButton.click();

  // Error message disappears
  await expect(errorMessage).not.toBeVisible();

  // Data loads successfully
  await expect(page.getByText('Analytics loaded')).toBeVisible();
});
```

### Network Timeout Error

```javascript
test('handle request timeout gracefully', async ({ page }) => {
  await page.goto('/reports');

  // Slow down network
  await page.route('**/api/reports/**', route => {
    setTimeout(() => route.continue(), 15000); // Longer than timeout
  });

  // Trigger report generation
  await page.getByRole('button', { name: 'Generate Report' }).click();

  // Timeout error appears
  await expect(page.getByRole('alert')).toContainText('Request timed out');

  // User can download sample report instead
  const downloadButton = page.getByRole('button', { name: 'Download Sample' });
  await expect(downloadButton).toBeVisible();
  await expect(downloadButton).toBeEnabled();
});
```

### Validation Error with Field Highlighting

```javascript
test('highlight invalid fields on form submission', async ({ page }) => {
  await page.goto('/checkout');

  // Submit without filling required fields
  await page.getByRole('button', { name: 'Place Order' }).click();

  // Name field has error state
  const nameField = page.getByLabel('Cardholder Name');
  await expect(nameField).toHaveClass(/error/);
  await expect(nameField.locator('~ span.error')).toContainText('This field is required');

  // Fill invalid email
  const emailField = page.getByLabel('Email');
  await emailField.fill('not-an-email');
  await page.getByRole('button', { name: 'Place Order' }).click();

  // Email field shows specific error
  await expect(emailField).toHaveClass(/error/);
  await expect(emailField.locator('~ span.error')).toContainText('Please enter a valid email');

  // Fix email
  await emailField.clear();
  await emailField.fill('customer@example.com');

  // Error disappears
  await expect(emailField).not.toHaveClass(/error/);
  await expect(emailField.locator('~ span.error')).not.toBeVisible();
});
```

---

## Success Message Verification

### Toast Notification

```javascript
test('show toast notification on success', async ({ page }) => {
  await page.goto('/settings');

  // Change setting
  await page.getByLabel('Theme').selectOption('dark');

  // Toast appears
  const toast = page.getByRole('status');
  await expect(toast).toBeVisible();
  await expect(toast).toContainText('Settings saved successfully');

  // Toast has correct icon
  await expect(toast.getByAltText('Success icon')).toBeVisible();

  // Toast auto-dismisses after 3 seconds
  await expect(toast).toBeHidden({ timeout: 4000 });
});
```

### Success Page

```javascript
test('show success page after checkout', async ({ page }) => {
  await page.goto('/checkout');

  // Fill and submit order
  await page.getByLabel('Card Number').fill('4242424242424242');
  await page.getByLabel('Expiry').fill('12/25');
  await page.getByLabel('CVC').fill('123');
  await page.getByRole('button', { name: 'Place Order' }).click();

  // Success page loads
  await expect(page).toHaveURL(/\/order-confirmation\/\d+/);

  // Success heading visible
  await expect(page.getByRole('heading', { name: 'Order Confirmed!' })).toBeVisible();

  // Order details displayed
  await expect(page.getByText('Order #12345')).toBeVisible();
  await expect(page.getByText('Total: $99.99')).toBeVisible();

  // Confirmation email message
  await expect(page.getByText(/confirmation email.*\d+.*@.*\.\w+/)).toBeVisible();

  // Next steps button
  const continueButton = page.getByRole('button', { name: 'Continue Shopping' });
  await expect(continueButton).toBeVisible();
  await expect(continueButton).toBeEnabled();
});
```

### Inline Success Message

```javascript
test('show inline success message after save', async ({ page }) => {
  await page.goto('/profile/edit');

  // Fill form
  await page.getByLabel('Bio').fill('Software engineer passionate about testing');

  // Save
  await page.getByRole('button', { name: 'Save Changes' }).click();

  // Success message appears
  const successMsg = page.getByRole('status', { name: 'Success' });
  await expect(successMsg).toBeVisible();
  await expect(successMsg).toContainText('Your profile has been updated');

  // Form is no longer in edit mode
  await expect(page.getByRole('button', { name: 'Edit Profile' })).toBeVisible();
  await expect(page.getByRole('button', { name: 'Save Changes' })).not.toBeVisible();
});
```

---

## Accessible Form Validation

### ARIA Live Region for Errors

```javascript
test('screen reader announces validation errors', async ({ page }) => {
  await page.goto('/signup');

  // Submit empty form
  await page.getByRole('button', { name: 'Create Account' }).click();

  // Error messages in live region (screen reader announces them)
  const liveRegion = page.locator('[aria-live="polite"][role="alert"]');
  await expect(liveRegion).toContainText('Email is required');
  await expect(liveRegion).toContainText('Password is required');

  // Invalid field gets aria-invalid attribute
  const emailField = page.getByLabel('Email');
  await expect(emailField).toHaveAttribute('aria-invalid', 'true');
  await expect(emailField).toHaveAttribute('aria-describedby', /.+/);

  // Error description is associated with field
  const errorId = await emailField.getAttribute('aria-describedby');
  const errorText = page.locator(`#${errorId}`);
  await expect(errorText).toContainText('Email is required');

  // Fill correct value
  await emailField.fill('valid@example.com');

  // aria-invalid is removed
  await expect(emailField).toHaveAttribute('aria-invalid', 'false');
});
```

### Loading State with ARIA

```javascript
test('accessible loading state', async ({ page }) => {
  await page.goto('/search');

  // Type search query
  await page.getByLabel('Search').fill('test');

  // Search button shows loading state
  const searchButton = page.getByRole('button', { name: 'Search' });
  await searchButton.click();

  // Button is disabled while loading
  await expect(searchButton).toBeDisabled();

  // Loading message is announced
  await expect(page.locator('[role="status"]')).toContainText('Searching...');

  // Results load
  await expect(searchButton).toBeEnabled();
  await expect(page.getByRole('list')).toBeVisible();
  await expect(page.locator('[role="status"]')).not.toBeVisible();
});
```

---

## Keyboard Navigation

### Tab Through Form

```javascript
test('navigate form with keyboard', async ({ page }) => {
  await page.goto('/contact');

  // Start at name field
  const nameInput = page.getByLabel('Name');
  await nameInput.focus();
  await expect(nameInput).toBeFocused();

  // Tab to email field
  await page.keyboard.press('Tab');
  await expect(page.getByLabel('Email')).toBeFocused();

  // Tab to subject field
  await page.keyboard.press('Tab');
  await expect(page.getByLabel('Subject')).toBeFocused();

  // Tab to message field
  await page.keyboard.press('Tab');
  await expect(page.getByLabel('Message')).toBeFocused();

  // Tab to submit button
  await page.keyboard.press('Tab');
  const submitButton = page.getByRole('button', { name: 'Submit' });
  await expect(submitButton).toBeFocused();

  // Press Enter to submit
  await page.keyboard.press('Enter');
  await expect(page.getByText('Thank you')).toBeVisible();
});
```

### Escape to Close Modal

```javascript
test('close modal with Escape key', async ({ page }) => {
  await page.goto('/items');

  // Open modal
  await page.getByRole('button', { name: 'Add Item' }).click();
  const modal = page.getByRole('dialog');
  await expect(modal).toBeVisible();

  // Verify focus is in modal
  const firstInput = modal.getByLabel('Name');
  await expect(firstInput).toBeFocused();

  // Press Escape
  await page.keyboard.press('Escape');

  // Modal closes
  await expect(modal).not.toBeVisible();

  // Focus returns to trigger button
  await expect(page.getByRole('button', { name: 'Add Item' })).toBeFocused();
});
```

---

## Complex Interactions

### Drag and Drop

```javascript
test('drag item to reorder list', async ({ page }) => {
  await page.goto('/tasks');

  // Get items in order
  const items = page.getByRole('listitem');
  const firstText = await items.nth(0).innerText();
  const secondText = await items.nth(1).innerText();

  // Drag first item to second position
  await items.nth(0).dragTo(items.nth(1));

  // Verify order changed
  const newOrder = await items.nth(0).innerText();
  expect(newOrder).toBe(secondText);

  // Verify persistence
  await page.reload();
  const reloadedItems = page.getByRole('listitem');
  const reloadedFirst = await reloadedItems.nth(0).innerText();
  expect(reloadedFirst).toBe(secondText);
});
```

### File Upload

```javascript
test('upload file with validation', async ({ page }) => {
  await page.goto('/upload');

  // Set up file input listener
  const fileInput = page.getByLabel('Choose file');

  // Upload file
  await fileInput.setInputFiles('/path/to/test-file.pdf');

  // File name appears
  await expect(page.getByText('test-file.pdf')).toBeVisible();

  // Verify file size displayed
  await expect(page.getByText('(245 KB)')).toBeVisible();

  // Submit
  await page.getByRole('button', { name: 'Upload' }).click();

  // Processing message
  await expect(page.getByText('Uploading...')).toBeVisible();

  // Success
  await expect(page.getByText('File uploaded successfully')).toBeVisible();
});
```

### Multi-Select with Search

```javascript
test('select multiple items with search', async ({ page }) => {
  await page.goto('/assign-users');

  // Click to open multi-select
  const selector = page.getByLabel('Assign Users');
  await selector.click();

  // Search for user
  const searchInput = page.getByPlaceholder('Search users...');
  await searchInput.fill('john');

  // Select matching option
  await page.getByRole('option', { name: 'John Doe' }).click();

  // Option is selected (has checkmark)
  await expect(page.getByRole('option', { name: 'John Doe' })).toHaveAttribute('aria-selected', 'true');

  // Search for another
  await searchInput.fill('jane');
  await page.getByRole('option', { name: 'Jane Smith' }).click();

  // Both selected
  const selected = page.locator('[aria-selected="true"]');
  await expect(selected).toHaveCount(2);

  // Close dropdown
  await page.keyboard.press('Escape');

  // Both appear as chips
  await expect(page.getByText('John Doe')).toBeVisible();
  await expect(page.getByText('Jane Smith')).toBeVisible();
});
```

---

## DO's and DON'Ts

### ✅ DO: Use semantic locators

```javascript
// ✅ GOOD
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByLabel('Email').fill('test@example.com');
await page.getByAltText('Profile picture').click();

// ❌ BAD
await page.locator('button.submit-btn').click();
await page.locator('input#email').fill('test@example.com');
await page.locator('img[alt="Profile picture"]').click();
```

### ✅ DO: Use web-first assertions

```javascript
// ✅ GOOD - waits for condition
await expect(page.getByText('Success')).toBeVisible();
await expect(page.getByLabel('Name')).toHaveValue('John');

// ❌ BAD - checks immediately
const isVisible = await page.locator('text=Success').isVisible();
expect(isVisible).toBe(true);
```

### ✅ DO: Test visible behavior

```javascript
// ✅ GOOD - test what user sees
test('form shows success', async ({ page }) => {
  await page.getByRole('button', { name: 'Save' }).click();
  await expect(page.getByText('Changes saved')).toBeVisible();
});

// ❌ BAD - test internal state
test('form updates state', async ({ page }) => {
  await page.getByRole('button', { name: 'Save' }).click();
  const state = await page.evaluate(() => window.__state.saved);
  expect(state).toBe(true);
});
```

### ✅ DO: Isolate tests

```javascript
// ✅ GOOD - each test is independent
test.beforeEach(async ({ page }) => {
  await loginAs(page, 'testuser@example.com');
});

test('can edit profile', async ({ page }) => {
  await page.goto('/profile/edit');
  // ...
});

// ❌ BAD - test depends on previous test
test('login', async ({ page }) => {
  // ...
});

test('can edit profile', async ({ page }) => {
  // Assumes login test ran first!
  await page.goto('/profile/edit');
});
```

### ✅ DO: Handle async operations

```javascript
// ✅ GOOD - await the operation
await page.getByRole('button', { name: 'Save' }).click();
await expect(page.getByText('Saved')).toBeVisible();

// ❌ BAD - fire and forget
page.getByRole('button', { name: 'Save' }).click(); // Not awaited!
// Next line may run before save completes
```

### ❌ DON'T: Use setTimeout

```javascript
// ❌ TERRIBLE - slow and unreliable
await page.goto('/data');
await new Promise(r => setTimeout(r, 2000));
await expect(page.getByText('Data loaded')).toBeVisible();

// ✅ GOOD - instant and reliable
await page.goto('/data');
await expect(page.getByText('Data loaded')).toBeVisible();
```

### ❌ DON'T: Use CSS chains

```javascript
// ❌ TERRIBLE - brittle selectors
await page.locator('.modal > .form > div:nth-child(2) input').fill('test');
await page.locator('button.primary.lg').click();

// ✅ GOOD - semantic locators
await page.getByLabel('Email').fill('test');
await page.getByRole('button', { name: 'Submit' }).click();
```

### ❌ DON'T: Test implementation details

```javascript
// ❌ WRONG - testing React props
const counter = await page.evaluate(() => {
  return document.querySelector('.Counter').props?.count;
});
expect(counter).toBe(5);

// ✅ CORRECT - testing visible output
await expect(page.getByText('Count: 5')).toBeVisible();
```

---

## Quick Reference

```javascript
// Locators (in priority order)
page.getByRole('button', { name: 'Submit' })
page.getByLabel('Email')
page.getByPlaceholder('Enter name')
page.getByAltText('Logo')
page.getByTitle('Help')
page.getByTestId('widget')
page.getByText('Click me')
// ❌ NEVER: page.locator('css or xpath')

// Actions (with auto-waiting)
await element.click()
await element.fill('text')
await element.check()
await element.selectOption('value')
await element.type('text')
await element.press('Enter')

// Assertions (with auto-waiting/retry)
await expect(element).toBeVisible()
await expect(element).toBeHidden()
await expect(element).toBeEnabled()
await expect(element).toBeDisabled()
await expect(element).toBeChecked()
await expect(element).toHaveValue('text')
await expect(element).toContainText('text')
await expect(element).toHaveCount(5)
await expect(element).toHaveClass('active')
await expect(element).toHaveAttribute('href', '/page')
await expect(page).toHaveURL('/page')
```
