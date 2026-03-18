# Accessibility for Testing: Why They're the Same Thing

> **Core Insight:** The accessibility tree is what Playwright uses to find and interact with elements. When you build accessible React code, E2E testing becomes trivial.

---

## The Accessibility Tree

Every webpage has two trees:
1. **DOM Tree** - How HTML is structured
2. **Accessibility Tree** - What assistive tech (and Playwright) see

Playwright doesn't look at CSS classes, ids, or `data-testid`. It queries the accessibility tree.

```javascript
// The DOM might look like this:
<div class="search-container">
  <div class="search-input-wrapper">
    <input class="search-box" placeholder="Search..." />
  </div>
  <div class="btn-primary-lg" onclick={...}>
    Go
  </div>
</div>

// The accessibility tree looks like this:
textbox "Search"  (from placeholder, no label)
button "Go"

// Playwright queries the accessibility tree, not the DOM:
await page.getByLabel('Search').fill('laptop');  // ❌ Fails - no label
await page.getByRole('textbox').fill('laptop');  // ⚠️ Works but fragile
await page.getByPlaceholderText('Search...').fill('laptop');  // Works but brittle
```

---

## Semantic HTML Builds the Right Tree

### Example: Form Input

```html
<!-- ❌ Bad - confuses the tree -->
<div>Enter your email:</div>
<input type="text" placeholder="Email" />

<!-- Accessibility tree sees:
text "Enter your email:"
textbox (no label!)
-->

<!-- ✅ Good - clear in the tree -->
<label for="email">Enter your email:</label>
<input id="email" type="email" />

<!-- Accessibility tree sees:
textbox "Enter your email:" (label is programmatically associated)
-->
```

**Why it matters for testing:**

```javascript
// With semantic HTML + label:
await page.getByLabel('Enter your email').fill('user@example.com');  // ✅ Works

// Without label:
await page.getByLabel('Enter your email').fill('user@example.com');  // ❌ Fails
```

### Example: Buttons

```html
<!-- ❌ Bad - div with click handler -->
<div onclick={handleClick}>Delete</div>

<!-- Accessibility tree sees:
text "Delete"  (not identified as button!)
-->

<!-- ✅ Good - semantic button -->
<button onclick={handleClick}>Delete</button>

<!-- Accessibility tree sees:
button "Delete"
-->
```

**Playwright test:**

```javascript
// With semantic button:
await page.getByRole('button', { name: /delete/i }).click();  // ✅ Works

// With div:
await page.getByRole('button', { name: /delete/i }).click();  // ❌ Fails
```

---

## ARIA Attributes Enhance the Tree

When semantic HTML isn't enough, ARIA attributes add information to the accessibility tree.

### aria-label: Custom Accessible Names

```javascript
// Problem: Icon button has no text
<button>🔍</button>

// Accessibility tree sees:
button ""  (empty name!)

// Solution: Add aria-label
<button aria-label="Search">🔍</button>

// Accessibility tree sees:
button "Search"
```

**Testing:**

```javascript
// With aria-label:
await page.getByRole('button', { name: /search/i }).click();  // ✅

// Without:
await page.getByRole('button', { name: /search/i }).click();  // ❌
```

### aria-describedby: Adding Descriptions

```javascript
// Input with description
<input
  id="email"
  type="email"
  aria-describedby="email-hint"
/>
<p id="email-hint">We'll never share your email</p>

// Accessibility tree includes:
textbox "Email" with description "We'll never share your email"

// Playwright can verify the hint:
const hint = page.getByText(/never share/i);
await expect(hint).toBeVisible();
```

### aria-hidden: Hiding Decorative Content

```javascript
// Decorative elements confuse the accessibility tree
<button>
  <span>❤️</span>
  Like
</button>

// Accessibility tree sees:
button "❤️ Like"  (emoji is read out!)

// Hide decorative content:
<button>
  <span aria-hidden="true">❤️</span>
  Like
</button>

// Accessibility tree sees:
button "Like"  (emoji is hidden from tree)
```

**Testing:**

```javascript
// With aria-hidden, button is easier to find:
await page.getByRole('button', { name: /like/i }).click();  // ✅
```

### aria-live: Announcing Dynamic Content

```javascript
// Content that updates dynamically
<div aria-live="polite">
  {isLoading && 'Loading...'}
  {results.length > 0 && `Found ${results.length} results`}
</div>

// Accessibility tree:
region "polite" with text "Found 5 results"

// Playwright can wait for live regions to update:
await page.getByText(/found 5 results/i).waitFor({ state: 'visible' });
```

**Testing async updates:**

```javascript
test('shows results after search', async ({ page }) => {
  await page.getByLabel('Search').fill('laptop');
  await page.getByRole('button', { name: /search/i }).click();

  // Wait for live region to announce results
  await expect(
    page.getByRole('region').filter({ hasText: /found.*results/i })
  ).toBeVisible();
});
```

### aria-expanded: Toggle States

```javascript
// Expandable section
<button
  aria-expanded={isOpen}
  aria-controls="menu-list"
>
  Menu
</button>
<ul id="menu-list" hidden={!isOpen}>
  <li><a href="/home">Home</a></li>
  <li><a href="/about">About</a></li>
</ul>

// Accessibility tree:
button "Menu" with aria-expanded="true"

// Playwright can verify state:
await expect(button).toHaveAttribute('aria-expanded', 'true');
```

**Testing:**

```javascript
test('menu expands and collapses', async ({ page }) => {
  const menuBtn = page.getByRole('button', { name: /menu/i });

  // Initially collapsed
  await expect(menuBtn).toHaveAttribute('aria-expanded', 'false');

  // Click to expand
  await menuBtn.click();
  await expect(menuBtn).toHaveAttribute('aria-expanded', 'true');

  // Links are now visible
  await expect(page.getByRole('link', { name: /home/i })).toBeVisible();
});
```

### aria-selected: Selection State

```javascript
// Tab component
<div role="tablist">
  <button role="tab" aria-selected={tab === 'overview'}>
    Overview
  </button>
  <button role="tab" aria-selected={tab === 'details'}>
    Details
  </button>
</div>

// Accessibility tree:
tablist
  tab "Overview" with aria-selected="true"
  tab "Details" with aria-selected="false"
```

**Testing:**

```javascript
test('can select different tabs', async ({ page }) => {
  const overviewTab = page.getByRole('tab', { name: /overview/i });
  const detailsTab = page.getByRole('tab', { name: /details/i });

  await expect(overviewTab).toHaveAttribute('aria-selected', 'true');

  await detailsTab.click();

  await expect(detailsTab).toHaveAttribute('aria-selected', 'true');
  await expect(overviewTab).toHaveAttribute('aria-selected', 'false');
});
```

---

## Semantic HTML Elements and Their Roles

Understanding which elements create which roles helps you build testable components.

### Text-Level Elements

```javascript
// Headings
<h1>Page Title</h1>           // role: heading, level: 1
<h2>Section</h2>             // role: heading, level: 2
<h3>Subsection</h3>          // role: heading, level: 3

// Paragraphs and text
<p>Paragraph text</p>        // (no special role)
<strong>Bold text</strong>   // (no special role)
<em>Emphasized text</em>     // (no special role)

// Links
<a href="/">Home</a>         // role: link

// Playground queries:
await page.getByRole('heading', { level: 1 });
await page.getByRole('link', { name: /home/i });
```

### Form Elements

```javascript
// Text inputs
<input type="text" />         // role: textbox
<input type="email" />        // role: textbox
<input type="password" />     // role: textbox
<input type="search" />       // role: searchbox
<textarea></textarea>        // role: textbox

// Selection
<input type="checkbox" />     // role: checkbox
<input type="radio" />        // role: radio
<select></select>            // role: combobox
<input type="range" />       // role: slider

// Buttons and submission
<button>Click</button>       // role: button
<input type="button" />      // role: button
<input type="submit" />      // role: button
<input type="reset" />       // role: button

// Playground queries:
await page.getByRole('textbox').fill('text');
await page.getByRole('button').click();
await page.getByRole('checkbox').check();
await page.getByRole('combobox').selectOption('Option');
```

### Grouping Elements

```javascript
// Lists
<ul><li>Item 1</li></ul>     // role: list, listitem
<ol><li>Item 1</li></ol>     // role: list, listitem

// Tables
<table></table>              // role: table
<thead></thead>              // (part of table)
<tr></tr>                    // role: row
<th>Header</th>              // role: rowheader or columnheader
<td>Cell</td>                // role: cell

// Forms
<form></form>                // role: form
<fieldset></fieldset>        // role: group (with <legend>)
<label></label>              // (connects to inputs)

// Playground queries:
await page.getByRole('list');
await page.getByRole('table');
await page.getByRole('form');
```

### Landmark Elements

```javascript
// Main regions
<main>Main content</main>       // role: main
<nav>Navigation</nav>           // role: navigation
<header>Header</header>         // role: banner
<footer>Footer</footer>         // role: contentinfo
<aside>Sidebar</aside>          // role: complementary
<section>Section</section>      // role: region

// Articles and document structure
<article>Article</article>      // role: article
<address>Address</address>      // role: (no special role)

// Playground queries:
await page.getByRole('main');
await page.getByRole('navigation');
await page.getByRole('complementary');
```

---

## Building Accessible (and Testable) Forms

Forms are where accessibility and testability are most tightly connected.

### Complete Form Example

```javascript
function RegistrationForm({ onSubmit }) {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    country: '',
    terms: false
  });
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    // Validation...
    await onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit} noValidate>
      {/* Fieldset groups related fields */}
      <fieldset>
        <legend>Personal Information</legend>

        <div>
          {/* Label is programmatically associated */}
          <label htmlFor="first-name">First Name</label>
          <input
            id="first-name"
            name="firstName"
            type="text"
            value={formData.firstName}
            onChange={handleChange}
            required
            aria-describedby={errors.firstName ? 'first-name-error' : undefined}
          />
          {errors.firstName && (
            <p id="first-name-error" role="alert">
              {errors.firstName}
            </p>
          )}
        </div>

        <div>
          <label htmlFor="last-name">Last Name</label>
          <input
            id="last-name"
            name="lastName"
            type="text"
            value={formData.lastName}
            onChange={handleChange}
            required
            aria-describedby={errors.lastName ? 'last-name-error' : undefined}
          />
          {errors.lastName && (
            <p id="last-name-error" role="alert">
              {errors.lastName}
            </p>
          )}
        </div>
      </fieldset>

      <fieldset>
        <legend>Contact Information</legend>

        <div>
          <label htmlFor="email">Email Address</label>
          <input
            id="email"
            name="email"
            type="email"
            value={formData.email}
            onChange={handleChange}
            required
            aria-describedby="email-hint"
            aria-invalid={!!errors.email}
          />
          <p id="email-hint">We'll use this for your account</p>
          {errors.email && (
            <p role="alert">{errors.email}</p>
          )}
        </div>

        <div>
          <label htmlFor="country">Country</label>
          <select
            id="country"
            name="country"
            value={formData.country}
            onChange={handleChange}
            required
          >
            <option value="">-- Select a country --</option>
            <option value="us">United States</option>
            <option value="ca">Canada</option>
            <option value="uk">United Kingdom</option>
          </select>
        </div>
      </fieldset>

      <div>
        {/* Checkbox label: label must wrap or reference the input */}
        <input
          id="terms"
          name="terms"
          type="checkbox"
          checked={formData.terms}
          onChange={handleChange}
          required
        />
        <label htmlFor="terms">
          I agree to the <a href="/terms">Terms of Service</a>
        </label>
      </div>

      <button type="submit">Create Account</button>
    </form>
  );
}
```

**Playwright tests:**

```javascript
test('user can fill and submit registration form', async ({ page }) => {
  // Find inputs by label
  await page.getByLabel('First Name').fill('John');
  await page.getByLabel('Last Name').fill('Doe');
  await page.getByLabel('Email Address').fill('john@example.com');
  
  // Find select by label
  await page.getByLabel('Country').selectOption('us');
  
  // Find checkbox by label
  await page.getByLabel(/i agree to the terms/i).check();
  
  // Submit form
  await page.getByRole('button', { name: /create account/i }).click();
  
  // Verify success
  await expect(page).toHaveURL('/dashboard');
});

test('shows validation error for invalid email', async ({ page }) => {
  await page.getByLabel('Email Address').fill('not-an-email');
  await page.getByLabel('First Name').fill('John');
  await page.getByLabel('Last Name').fill('Doe');
  await page.getByLabel('Country').selectOption('us');
  await page.getByLabel(/i agree to the terms/i).check();
  
  await page.getByRole('button', { name: /create account/i }).click();
  
  // Error is announced
  await expect(page.getByRole('alert')).toContainText(/invalid email/i);
});

test('shows description text for email field', async ({ page }) => {
  // Users see helpful hints
  await expect(
    page.getByText(/we'll use this for your account/i)
  ).toBeVisible();
});
```

---

## Keyboard Navigation Testing

Accessibility includes keyboard navigation. Testable components work with Tab, Arrow, Enter, and Escape.

```javascript
// Tab navigates through interactive elements
test('can navigate form with Tab key', async ({ page }) => {
  const firstNameInput = page.getByLabel('First Name');
  const lastNameInput = page.getByLabel('Last Name');
  const emailInput = page.getByLabel('Email Address');
  const submitBtn = page.getByRole('button', { name: /submit/i });

  // Focus first input
  await firstNameInput.focus();
  expect(await firstNameInput.evaluate(el => el === document.activeElement)).toBe(true);

  // Tab to next
  await page.keyboard.press('Tab');
  expect(await lastNameInput.evaluate(el => el === document.activeElement)).toBe(true);

  // Tab to next
  await page.keyboard.press('Tab');
  expect(await emailInput.evaluate(el => el === document.activeElement)).toBe(true);

  // Continue through form
  await page.keyboard.press('Tab');
  expect(await submitBtn.evaluate(el => el === document.activeElement)).toBe(true);

  // Submit with Enter
  await page.keyboard.press('Enter');
});

// Arrow keys navigate lists, menus, tabs
test('can navigate menu with arrow keys', async ({ page }) => {
  const menuBtn = page.getByRole('button', { name: /menu/i });
  await menuBtn.click();

  const firstItem = page.getByRole('menuitem').first();
  const secondItem = page.getByRole('menuitem').nth(1);

  // Focus first item
  await firstItem.focus();

  // Arrow down to next
  await page.keyboard.press('ArrowDown');
  expect(await secondItem.evaluate(el => el === document.activeElement)).toBe(true);

  // Arrow down again
  await page.keyboard.press('ArrowDown');

  // Escape closes menu
  await page.keyboard.press('Escape');
  await expect(page.getByRole('menu')).not.toBeVisible();
});
```

---

## Image and Alternative Text

Images need alt text for the accessibility tree:

```javascript
// ❌ Bad - no alt text
<img src="product.jpg" />

// ✅ Good - descriptive alt text
<img src="product.jpg" alt="Blue wireless headphones with noise cancellation" />

// ✅ Decorative - empty alt
<img src="decorative-line.svg" alt="" />

// ✅ Icon alternative
<img src="search-icon.svg" alt="Search" />
```

**Testing with images:**

```javascript
test('can find product by alt text', async ({ page }) => {
  const productImage = page.getByRole('img', {
    name: /blue wireless headphones/i
  });

  await expect(productImage).toBeVisible();
});
```

---

## Summary: Accessibility Checklist for Testing

- [ ] Use semantic HTML (`<button>`, `<label>`, `<form>`, `<input>`, etc.)
- [ ] Every form input has a `<label>` with `htmlFor`
- [ ] Icon buttons have `aria-label`
- [ ] Dynamic content uses `aria-live`
- [ ] Toggle buttons have `aria-expanded`
- [ ] Selected items have `aria-selected`
- [ ] Error messages have `role="alert"`
- [ ] Disabled elements have `disabled` attribute
- [ ] Images have meaningful `alt` text
- [ ] Color is not the only way to convey information
- [ ] Contrast is sufficient (7:1 for normal text, 4.5:1 minimum)
- [ ] Keyboard navigation works (Tab, Arrow keys, Enter, Escape)

---

## Next Steps

- See **SKILL.md** for full implementation patterns
- See **EXAMPLES.md** for complete accessible components
- See **ANTI-PATTERNS.md** for what breaks accessibility and testing
