# Debugging Failing Playwright Tests: When Code and Tests Appear Correct

## The Problem Scenario

You're confident your code is correct. You're confident your test is correct. Yet the test fails. You read the error message, check the test logic, verify the implementation—everything looks right. But something isn't working as expected.

This is one of the most frustrating situations in test development. The test failure doesn't point to an obvious bug in either the code or the test. The issue might be:

- **Timing-related** (element appears later than expected)
- **Environment-related** (works locally but fails in CI)
- **Data-related** (mock data doesn't match expectations)
- **Locator-related** (element matches different HTML than expected)
- **Visibility-related** (element exists but isn't visible)
- **Race-condition-related** (test runs before app finishes initialization)

When faced with this situation, you need a systematic debugging approach. This guide provides that methodology.

---

## Debugging Workflow: A Systematic Approach

The key to debugging failing tests is to separate concerns and methodically eliminate possibilities. Follow this workflow:

### Step 1: Manual Replication

**The Action**: Manually replicate exactly what the test is trying to do.

This is the most important step and often reveals the issue immediately.

**How to do it:**

1. **Run the application** in your development environment
2. **Follow the exact same steps** that your test script performs
   - If the test navigates to a URL, go to that URL manually
   - If the test fills a form, fill that form manually in the browser
   - If the test clicks a button, click that button manually
   - If the test expects text to appear, watch for that text manually
3. **Document what actually happens** vs. what you expected
4. **Pay close attention to**:
   - How long elements take to appear
   - If elements are visible or hidden
   - What text actually displays
   - Whether clicks work as expected
   - If data loads correctly
   - Any console errors

**Why this works:**

Playwright may have timing issues, locator mismatches, or environment differences that aren't obvious from just reading the code. By manually performing the exact same steps, you verify whether the application actually works as your test expects.

**Common discoveries:**

- **Element takes longer to appear** than the test allows
- **Text is different** (typos, extra whitespace, dynamic content)
- **Element exists but isn't visible** (opacity: 0, display: none, overflow hidden)
- **Element is disabled** when the test tries to click it
- **Data isn't loaded** when the test expects it
- **Redirect happens** that the test doesn't account for

**Example workflow:**

```
Test expects: User can submit form and see success message

Manual replication:
1. Open the form manually
   ✓ Form loads correctly
2. Fill in the fields manually
   ✓ Fields accept input
3. Click submit button manually
   ✓ Button responds to click
4. Wait for success message
   ⚠️ DISCOVERY: Message appears after 2 seconds, not immediately
   
Debug finding: Test is checking for success message too quickly
```

---

### Step 2: Log Analysis

**The Action**: Examine all available logs to find what actually happened.

After manual replication, check the logs that Playwright and your application generated.

**What to check:**

1. **Playwright debug logs**
   ```bash
   # Run test with debug mode enabled
   PWDEBUG=1 npx playwright test test-name.spec.ts
   # Or with trace
   npx playwright test --trace on
   npx playwright show-trace trace.zip
   ```

2. **Browser console logs**
   - Open Developer Tools (F12)
   - Check the Console tab for errors, warnings, or custom logs
   - Application might log timing information

3. **Network activity logs**
   - Check Network tab in Developer Tools
   - Look for failed requests
   - Check response times
   - Verify API responses match expected data

4. **Application logs**
   - Check server console output
   - Check database query logs
   - Look for errors in backend services

5. **Test output details**
   ```bash
   # Run with verbose reporter to see detailed output
   npx playwright test --reporter=verbose
   
   # Generate HTML report
   npx playwright test --reporter=html
   npx playwright show-report
   ```

**Common findings from logs:**

- **Locator timeout**: "Timeout of 5000ms exceeded while waiting for locator"
  - Element never appeared in the DOM
  - Locator doesn't match any element
  - Element is dynamically created with different structure than expected

- **Text mismatch**: Expected "Sign In" but element contains "SIGN IN"
  - Case sensitivity
  - Extra whitespace or formatting
  - Dynamic content rendering differently

- **Element visibility issue**: Element found but not visible
  - CSS visibility: hidden
  - CSS opacity: 0
  - CSS display: none
  - Element overflow outside visible area

- **Network failure**: API didn't return
  - Network request failed
  - Response took too long
  - Wrong endpoint or method

- **Race condition**: Network request still pending
  - Test runs before API response arrives
  - Element renders after test checked for it

- **State issue**: Data not in expected state
  - Database not seeded
  - Mock data incomplete
  - Previous test modified state

**Example log analysis:**

```
Test output: "Timeout of 5000ms exceeded while waiting for locator..."

Checking Playwright trace:
→ Found navigation to /login at 100ms
→ Form appears at 150ms
→ Input fields appear at 200ms
→ But test is looking for role="button", name="Sign In"
→ Button doesn't appear until 3500ms (API call to get button text)

Finding: Test is checking for button before API populates it
Solution: Add explicit wait for the API data
```

---

### Step 3: Identify the Category

Once manual replication and logs reveal the actual issue, categorize it to determine the fix.

#### Category 1: Playwright Issues (Test File Problem)

**Symptoms:**
- Locator doesn't match the element you know exists
- Test times out waiting for element that's clearly there
- Assertion passes locally but fails in CI
- Test passes inconsistently (flaky)

**Common causes:**
- **Locator mismatch**: Selector matches wrong element or no element
- **Timing expectations wrong**: Element takes longer to appear than expected
- **Interaction assumptions wrong**: Click works manually but not in test
- **Assertion too strict**: Assertion expects exact match when content varies
- **Assertion too loose**: Assertion passes when it shouldn't
- **Test isolation issue**: Test depends on state from previous test
- **Environment difference**: Page loads differently in test vs manual

**How to fix:**

1. **Debug the locator**:
   ```javascript
   // Log what the locator actually matches
   const elements = await page.locator('button', { name: 'Sign In' }).all();
   console.log(`Found ${elements.length} elements matching the locator`);
   
   // Check first element
   if (elements.length > 0) {
     const text = await elements[0].textContent();
     console.log(`First element text: "${text}"`);
   }
   ```

2. **Use Playwright Inspector**:
   ```bash
   npx playwright test --debug test-name.spec.ts
   ```
   This opens the Playwright Inspector where you can:
   - Inspect the page DOM
   - Test locators in real-time
   - Step through test execution
   - See exactly what the page looks like at each step

3. **Update the locator** to match reality:
   ```javascript
   // Wrong - looking for text that doesn't exist
   page.getByRole('button', { name: 'Sign In' })
   
   // Right - looking for text that's actually there
   page.getByRole('button', { name: 'SIGN IN' })
   
   // Or use regex if text varies
   page.getByRole('button', { name: /sign in/i })
   ```

4. **Fix timing expectations**:
   ```javascript
   // Instead of arbitrary wait
   await page.waitForTimeout(1000); // ❌ Wrong
   
   // Wait for the actual element
   await expect(page.getByRole('button', { name: 'Sign In' })).toBeVisible();
   ```

5. **Isolate the test** from others:
   ```javascript
   test.beforeEach(async ({ page }) => {
     // Ensure clean state before each test
     await page.goto('/');
     // Don't rely on state from previous test
   });
   ```

---

#### Category 2: Code Issues (Application Code Problem)

**Symptoms:**
- Manual replication shows the application doesn't behave as expected
- Test is correct, but code doesn't implement the feature
- Element doesn't render when it should
- Element is disabled when it should be enabled
- Text content is wrong
- Event handler doesn't fire

**Common causes:**
- **Element not rendered**: Conditional rendering is false when it should be true
- **Element hidden**: CSS hides element that should be visible
- **Event handler missing**: Button click doesn't trigger expected action
- **State not updating**: Component state doesn't change as expected
- **Logic error**: Business logic contains a bug
- **Incorrect text**: Display string is wrong
- **Wrong attributes**: Element is disabled/readonly when shouldn't be

**How to fix:**

1. **Fix the application code**:
   ```javascript
   // Example: Button not showing due to bad condition
   
   // Wrong
   {showButton === false && <button>Submit</button>}
   
   // Right
   {showButton === true && <button>Submit</button>}
   ```

2. **Verify the fix**:
   - Run the test again
   - Manually verify behavior
   - Check no side effects from the fix

3. **Add tests if needed**:
   - Test might reveal a genuine code issue
   - Fix the issue and test passes
   - Prevents future regressions

---

#### Category 3: Environment Issues (Setup/Config Problem)

**Symptoms:**
- Test passes locally but fails in CI
- Test passes in one environment but not another
- Mock data doesn't match expectations
- API responses are different
- Database doesn't have expected data
- Port/URL is different

**Common causes:**
- **API mocking not set up**: Test expects mocked response but gets real API
- **Seed data missing**: Database doesn't have test data
- **Network stubbing incomplete**: Only some requests are mocked
- **Browser configuration wrong**: Different headless behavior
- **Base URL wrong**: Test connects to wrong server
- **Environment variables missing**: Feature flag not set
- **Race condition in setup**: Test starts before setup completes

**How to fix:**

1. **Verify API mocking** (if using MSW or similar):
   ```javascript
   // Ensure mock setup runs before test
   test.beforeEach(async ({ page }) => {
     // Initialize mock handlers
     server.listen(); // or resetHandlers()
   });
   ```

2. **Verify seed data**:
   ```javascript
   // Reset database before each test
   test.beforeEach(async () => {
     await resetDatabase();
     await seedTestData();
   });
   ```

3. **Check environment variables**:
   ```bash
   # Verify variables are set in test environment
   echo $DATABASE_URL
   echo $API_BASE_URL
   ```

4. **Verify base URL** in playwright.config.ts:
   ```javascript
   webServer: {
     command: 'npm run dev',
     url: 'http://localhost:3000', // Verify this is correct
     reuseExistingServer: true,
   },
   ```

5. **Add explicit setup waits**:
   ```javascript
   test('can log in', async ({ page }) => {
     // Ensure API is responding
     await page.goto('/');
     
     // Wait for page to fully load
     await page.waitForLoadState('networkidle');
     
     // Now perform test
     await page.getByLabel('Email').fill('test@example.com');
   });
   ```

---

#### Category 4: Timing/Race Condition Issues

**Symptoms:**
- Test times out waiting for element
- Test passes when run alone, fails when run with others
- Test passes with --workers=1, fails with --workers=4
- Test passes when run slowly, fails when run quickly
- Console shows "Element is not attached to the DOM"

**Common causes:**
- **App loads slower in test environment**: Network slower, rendering slower
- **Test starts before initialization complete**: beforeEach finishes before app ready
- **Async operation not awaited**: Promise never resolves
- **Multiple requests racing**: API requests complete in wrong order
- **Re-render timing**: Element detaches and reattaches before test runs
- **Worker interference**: Tests running in parallel interfere with each other

**How to fix:**

1. **Use proper Playwright waiting** (not setTimeout):
   ```javascript
   // Wrong - arbitrary timeout
   await page.waitForTimeout(2000);
   
   // Right - wait for actual condition
   await page.waitForLoadState('networkidle');
   await expect(page.getByRole('button', { name: 'Submit' })).toBeVisible();
   ```

2. **Ensure setup completes** before test runs:
   ```javascript
   test.beforeEach(async ({ page }) => {
     // Navigate to page
     await page.goto('/dashboard');
     
     // Wait for critical content to load
     await expect(page.getByRole('heading')).toBeVisible();
     // Now test can proceed
   });
   ```

3. **Use auto-waiting assertions**:
   ```javascript
   // Playwright automatically waits up to 5 seconds
   await expect(page.getByRole('button', { name: 'Save' })).toBeEnabled();
   // Won't fail immediately if button is disabled
   // Will retry every 100ms until it's enabled or timeout
   ```

4. **Isolate tests** to prevent parallel interference:
   ```javascript
   test.describe.serial('database operations', () => {
     // These tests run sequentially, not in parallel
     test('can create record', async ({ page }) => {
       // ...
     });
     
     test('can read record', async ({ page }) => {
       // ...
     });
   });
   ```

5. **Use explicit wait for load state**:
   ```javascript
   test('can submit form', async ({ page }) => {
     await page.goto('/form');
     
     // Wait for page to fully load and stabilize
     await page.waitForLoadState('networkidle');
     
     // Now interact with page
     await page.getByLabel('Name').fill('John');
     await page.getByRole('button', { name: 'Submit' }).click();
   });
   ```

---

#### Category 5: Data Issues

**Symptoms:**
- Test expects data that doesn't exist
- Test expects specific data but gets different data
- Database has data locally but not in CI
- Mock response doesn't match actual API format
- Test data created by one test not available in another

**Common causes:**
- **Expected data missing**: Not created during setup
- **Data in wrong format**: Mock data doesn't match real API format
- **Database not seeded**: Test assumes data exists
- **Mock data incomplete**: Only mocking part of API response
- **Data cleanup wrong**: Previous test deleted data
- **Test isolation failure**: Tests share data

**How to fix:**

1. **Verify mock data setup**:
   ```javascript
   // Ensure mock data is created before test
   test.beforeEach(async () => {
     const user = await createTestUser({
       email: 'test@example.com',
       name: 'Test User',
     });
     
     // Verify it was created
     const retrieved = await getUser(user.id);
     expect(retrieved).toEqual(user);
   });
   ```

2. **Verify mock responses match API schema**:
   ```javascript
   // Mock should match what real API returns
   mock('/api/users/1', {
     status: 200,
     body: {
       id: 1,
       email: 'test@example.com',
       name: 'Test User',
       createdAt: '2024-01-01T00:00:00Z',
     }
   });
   ```

3. **Check test isolation**:
   ```javascript
   test.beforeEach(async () => {
     // Clean up before each test
     await deleteAllUsers();
     
     // Create fresh test data
     await createTestUser({ email: 'test@example.com' });
   });
   ```

4. **Verify database seeding** in CI:
   ```bash
   # In CI config, ensure database is seeded
   - name: Seed database
     run: npm run db:seed:test
   ```

---

## Common Patterns & Solutions

When debugging, you'll encounter recurring patterns. Here are the most common ones and how to solve them:

### Pattern 1: "Element not found" but Element Clearly Exists in Code

**What you see:**
```
Error: Timeout 5000ms exceeded waiting for locator
```

**What's happening:**
- Locator doesn't match the actual element
- Element exists in code but test is looking for something different

**Debug steps:**
1. Log actual element attributes:
   ```javascript
   const elements = await page.locator('button').all();
   for (const el of elements) {
     console.log(await el.textContent());
     console.log(await el.getAttribute('role'));
     console.log(await el.getAttribute('aria-label'));
   }
   ```

2. Check what the locator actually finds:
   ```javascript
   // In Playwright Inspector (--debug mode)
   // Type: page.getByRole('button', { name: 'Sign In' }).count()
   // This shows how many elements match
   ```

3. Adjust locator to match reality:
   ```javascript
   // If text has different case
   page.getByRole('button', { name: /sign in/i }) // Case-insensitive
   
   // If text has extra whitespace
   page.getByRole('button', { name: 'Sign In', exact: false })
   
   // If element has aria-label instead of visible text
   page.getByLabel('Sign In')
   ```

**Solution:**
Use Playwright Inspector to test locators in real-time until you find one that matches.

---

### Pattern 2: "Timeout waiting for element" but It Appears When Testing Manually

**What you see:**
```
Error: Timeout of 5000ms exceeded while waiting for locator to satisfy condition
```

**What's happening:**
- Playwright starts looking before element renders
- Element takes longer to appear than Playwright's default wait
- Network request hasn't completed yet

**Debug steps:**
1. Check when element actually appears:
   ```javascript
   const start = Date.now();
   await expect(page.getByRole('button', { name: 'Submit' })).toBeVisible();
   console.log(`Element appeared after ${Date.now() - start}ms`);
   ```

2. Verify app is fully loaded:
   ```javascript
   test('can submit form', async ({ page }) => {
     // Don't start test until app is ready
     await page.goto('/form');
     await page.waitForLoadState('networkidle');
     
     // Now element should exist
     await expect(page.getByRole('button', { name: 'Submit' })).toBeVisible();
   });
   ```

3. Check for network delays:
   - Open DevTools Network tab
   - See how long requests take
   - Increase timeout if needed

**Solution:**
- Ensure setup waits for critical elements
- Use `page.waitForLoadState('networkidle')` before test starts
- Trust Playwright's auto-waiting (don't add extra waits)

---

### Pattern 3: "Text doesn't match" Even Though Code Shows Correct Text

**What you see:**
```
AssertionError: expected 'Sign  In' to contain 'Sign In'
```

**What's happening:**
- Text content doesn't exactly match assertion
- Extra whitespace or formatting
- Dynamic content renders differently than expected

**Debug steps:**
1. Log the actual text with visible whitespace:
   ```javascript
   const text = await page.getByRole('button').first().textContent();
   console.log(`Text: [${text}]`); // Brackets show whitespace
   ```

2. Check for dynamic content:
   ```javascript
   // Log multiple times to see if text changes
   console.log(await element.textContent()); // First check
   await page.waitForTimeout(100);
   console.log(await element.textContent()); // Second check
   ```

3. Test with regex matching:
   ```javascript
   // Instead of exact match
   await expect(element).toContainText('Sign In');
   
   // Or with regex to be flexible
   await expect(element).toContainText(/sign\s+in/i);
   ```

**Solution:**
- Use `toContainText()` instead of `toHaveText()` for flexibility
- Use regex matching for content that might vary
- Normalize whitespace if comparing strings

---

### Pattern 4: "Click succeeded but nothing happened"

**What you see:**
```
✓ Button clicked successfully
✗ Expected modal to appear but it didn't
```

**What's happening:**
- Click succeeded but precondition wasn't met
- Element is clickable but doesn't do what expected
- Element exists but is disabled or hidden
- Wrong element was clicked

**Debug steps:**
1. Verify correct element is clickable:
   ```javascript
   const button = page.getByRole('button', { name: 'Open Modal' });
   
   // Check element properties
   console.log(await button.isEnabled()); // Should be true
   console.log(await button.isVisible()); // Should be true
   console.log(await button.isEditable()); // Should be true
   ```

2. Log what happened after click:
   ```javascript
   await button.click();
   
   // Wait a moment and check page state
   await page.waitForTimeout(500);
   console.log(await page.locator('[role="dialog"]').count()); // 0 = no modal
   ```

3. Check for JavaScript errors:
   ```javascript
   page.on('console', msg => {
     if (msg.type() === 'error') {
       console.log('JS Error:', msg.text());
     }
   });
   
   await button.click();
   ```

**Solution:**
- Verify button is enabled before clicking
- Add explicit wait for expected result after click
- Check console for JavaScript errors
- Verify click target is correct element

---

### Pattern 5: "Test passes locally but fails in CI"

**What you see:**
```
✓ Local: Test passes
✗ CI: Timeout waiting for element
```

**What's happening:**
- Environment difference (network, timing, configuration)
- CI runs headless (no display) which can affect rendering
- CI has different network conditions
- CI doesn't have same seed data

**Debug steps:**
1. Check CI logs in detail:
   ```bash
   # Look at CI runner output for errors
   # Check if setup steps ran correctly
   # Verify environment variables are set
   ```

2. Reproduce CI environment locally:
   ```bash
   # Run in headless mode like CI does
   HEADED=false npx playwright test test-name.spec.ts
   
   # Run with CI settings
   npx playwright test --reporter=github
   ```

3. Compare environments:
   - Local: What OS, browser version, network?
   - CI: What OS, browser version, network?
   - Are environment variables the same?
   - Is database seeded the same way?

4. Add explicit timing safety:
   ```javascript
   test.beforeEach(async ({ page }) => {
     await page.goto('/');
     // Extra safety wait for CI environments
     await page.waitForLoadState('networkidle');
     // Wait for critical element
     await expect(page.getByRole('main')).toBeVisible();
   });
   ```

**Solution:**
- Use `--debug` and check logs to see what's different
- Add explicit waits for critical content
- Verify all setup runs correctly
- Mock network requests consistently
- Use same browser version in CI and locally

---

### Pattern 6: "Test passed yesterday, fails today"

**What you see:**
```
✓ Yesterday: All tests passed
✗ Today: Random test fails
```

**What's happening:**
- Flaky test that sometimes passes, sometimes fails
- Code changed and test didn't account for change
- Timing issue that only happens sometimes
- Database state issue affecting test

**Debug steps:**
1. Check if test is flaky:
   ```bash
   # Run test multiple times
   npx playwright test --repeat-each=10 test-name.spec.ts
   # If it sometimes passes/fails, it's flaky
   ```

2. Check for recent code changes:
   ```bash
   git log --oneline -10
   # Look at changes to components being tested
   ```

3. Look for timing issues:
   ```javascript
   // Add explicit waits if test is flaky
   test('flaky test', async ({ page }) => {
     await page.goto('/');
     
     // Add explicit wait for critical content
     await expect(page.getByRole('main')).toBeVisible();
     
     // Test code
   });
   ```

4. Check database state:
   ```javascript
   test.beforeEach(async () => {
     // Ensure clean state each time
     await cleanupDatabase();
   });
   ```

**Solution:**
- Run test multiple times with `--repeat-each`
- Add explicit waits for timing issues
- Ensure clean database state
- Check for recent code changes
- Use fixtures to ensure consistent test setup

---

## When to Escalate to User

Sometimes you've done the debugging but still can't identify the root cause. At this point, it's time to get the user involved.

### What to Tell the User

Be specific and provide all the information they need to help debug:

1. **The specific test that's failing**:
   ```
   "The test 'can submit payment form' in payment.spec.ts is failing"
   ```

2. **What the test is trying to do**:
   ```
   "The test fills out a payment form with credit card details,
    clicks Submit, and expects to see a success message"
   ```

3. **What error/behavior is happening**:
   ```
   "The test times out waiting for the success message to appear.
    The error is: 'Timeout of 5000ms exceeded while waiting for locator'"
   ```

4. **What you've already verified**:
   ```
   ✓ Manually replicated the test steps - form works correctly
   ✓ Checked element exists in DOM - it does appear
   ✓ Checked Playwright logs - no obvious errors
   ✓ Verified test is correctly written - syntax and locators look right
   ✓ Checked for API errors - payment API returns success
   ```

5. **What debugging you've done**:
   ```
   - Ran test with PWDEBUG=1 to inspect execution
   - Checked browser console for errors - none found
   - Checked network tab - all requests successful
   - Ran test 5 times - fails consistently
   - Ran in local vs CI - fails in both
   ```

6. **What category you suspect**:
   ```
   "I suspect this is a Timing/Race Condition issue.
    The success message might be appearing after the test's
    default timeout, or there's a delay in the page update"
   ```

7. **Any logs or console output**:
   ```
   Full test output:
   ...
   
   Playwright trace (if available):
   ...
   
   Browser console logs:
   ...
   ```

8. **Next steps you'd like to try**:
   ```
   "Would you like me to:
    - Increase timeout to see if that helps?
    - Add more explicit waits?
    - Check if the API response is slower?
    - Look at the payment form implementation?"
   ```

### Questions to Ask the User

1. **"Can you verify [specific thing] on your end?"**
   ```
   "Can you manually test the payment form flow and confirm
    the success message appears correctly?"
   ```

2. **"Would you like me to [specific debugging action]?"**
   ```
   "Would you like me to increase the timeout to 10 seconds
    to see if that resolves it?"
   ```

3. **"Do you want to investigate [specific area]?"**
   ```
   "Should we look at the API response time for the payment endpoint?"
   ```

4. **"Has [something] changed recently?"**
   ```
   "Has the payment form component been modified recently?
    The test was passing before."
   ```

5. **"Can you run this test locally?"**
   ```
   "Can you run the failing test on your machine?
    Does it pass locally for you?"
   ```

### Provide Options

Give the user choices for how to proceed:

```
Here are some options for next steps:

Option 1: Debug the API
- Check if payment API is slow
- Add network logging to see response time
- Verify API returns success quickly

Option 2: Debug the UI rendering
- Check if form needs time to update
- Look at component re-render timing
- Add explicit wait for success message element

Option 3: Increase timeout
- Run with longer timeout to see if it helps
- Not a permanent solution but confirms timing issue

Option 4: Look at implementation
- Review payment form component code
- Check if success message has conditional rendering
- Verify state updates correctly

Which would you like to try first?
```

### Provide Failure Details

Include specific information:

```
FAILURE DETAILS:

Test Name: payment.spec.ts › can submit payment form
Error: Timeout of 5000ms exceeded while waiting for locator

What was expected:
- Form submits successfully
- API returns success response
- Success message appears on page

What actually happened:
- Form submitted
- API called (need to verify response)
- Success message did not appear (timeout)

Timing information:
- Test starts at 0ms
- Form fills at ~100ms
- Submit clicked at ~200ms
- Timeout occurs at 5200ms

Possible causes:
1. Success message element doesn't exist in DOM
2. Success message appears after 5 second timeout
3. API response delayed
4. Page redirect happens
5. Wrong element is being waited for
```

---

## Tools & Commands Reference

Keep these commands handy when debugging:

### Run Single Test with Debug Output

```bash
# Run a specific test with Playwright Inspector
npx playwright test test-name.spec.ts --debug

# Run with more verbose output
npx playwright test test-name.spec.ts --reporter=verbose

# Run with trace recording
npx playwright test test-name.spec.ts --trace on
```

### View Execution Traces

```bash
# After running with --trace on, view the trace
npx playwright show-trace trace.zip

# This shows:
# - Page state at each step
# - Network activity
# - Console logs
# - Timing information
# - Screenshots at each step
```

### Run with Environment Variables

```bash
# Run with debug mode enabled
PWDEBUG=1 npx playwright test test-name.spec.ts

# Run in headed mode (see the browser)
npx playwright test --headed test-name.spec.ts

# Run in slow motion (helpful for seeing what's happening)
npx playwright test --debug-on-failure test-name.spec.ts
```

### Generate Reports

```bash
# Generate HTML report
npx playwright test --reporter=html

# View the report
npx playwright show-report

# List all reporters available
npx playwright test --help
```

### Run Specific Test or Line

```bash
# Run all tests in a file
npx playwright test test-name.spec.ts

# Run specific test
npx playwright test test-name.spec.ts -g "can submit form"

# Run test starting at specific line
npx playwright test test-name.spec.ts:42
```

### Check Element with Page Inspector

```bash
# Generate test code by inspecting page
npx playwright codegen http://localhost:3000

# This opens a browser where you can:
# - Inspect elements
# - Click elements to generate locators
# - Type text to see selectors
# - Generate test code as you interact
```

### Debug Locators

In Playwright Inspector (running with `--debug`), you can:

```javascript
// Test a locator in the console
page.getByRole('button', { name: 'Submit' }).count()

// See what it matches
page.getByRole('button', { name: 'Submit' }).all()

// Get detailed info
page.getByRole('button', { name: 'Submit' }).first().textContent()
```

---

## Debugging Checklist

Quick reference checklist when you encounter a test failure:

### Immediate Checks (do these first)

- [ ] Manually replicate test steps in browser
  - Does the application work as expected?
  - What actually happens vs. what test expects?
  
- [ ] Check browser console for errors
  - Are there JavaScript errors?
  - Any warnings that might be relevant?
  
- [ ] Check network tab for failed requests
  - Did all API requests succeed?
  - Were responses what you expected?
  
- [ ] Verify element exists in DOM
  - Right-click → Inspect Element
  - Can you find the element manually?

### Element Checks

- [ ] Verify element is visible (not hidden/overflow)
  - Check CSS `display`, `visibility`, `opacity`
  - Check if element is covered by other element
  - Check if in scrollable area that's scrolled off
  
- [ ] Verify text content matches exactly
  - Check for whitespace differences
  - Check for case sensitivity
  - Check for dynamic content differences

- [ ] Check locator matches actual element
  - Use `--debug` mode to test locator
  - Verify it matches the right element
  - Adjust if needed

### Test Checks

- [ ] Verify test setup/beforeEach runs correctly
  - Does setup create needed test data?
  - Does setup navigate to right page?
  - Does setup wait for page to load?

- [ ] Check if test is isolated
  - Does test depend on other tests?
  - Would test pass if run in different order?
  - Would test pass if run in parallel?

- [ ] Run test multiple times (flakiness check)
  ```bash
  npx playwright test --repeat-each=10 test-name.spec.ts
  ```
  - Does it always pass? Always fail? Sometimes?

### Environment Checks

- [ ] Compare local vs CI behavior
  - Does test pass locally but fail in CI?
  - What's different about the environments?
  - Are environment variables set?

- [ ] Check Playwright version
  ```bash
  npx playwright --version
  ```

- [ ] Check browser version
  ```bash
  npx playwright install --with-deps
  npx playwright test --list --reporter=json | head -20
  ```

- [ ] Verify mock data is correct
  - Is seed data created?
  - Does mock response match real API?
  - Is test data fresh for each test?

### If Still Stuck

- [ ] Run with `--debug` and inspect step-by-step
- [ ] Enable trace recording and view trace
- [ ] Check Playwright logs for clues
- [ ] Review application source code
- [ ] Ask user to verify expected behavior
- [ ] Consider escalating with all gathered information

---

## Summary

When tests fail despite appearing correct, follow this systematic approach:

1. **Manually replicate** - Confirm what actually happens vs. expected
2. **Analyze logs** - Review Playwright traces, console logs, network activity
3. **Categorize the issue** - Playwright, Code, Environment, Timing, or Data
4. **Apply the fix** - Based on the category, fix the issue
5. **Verify the fix** - Ensure test passes and no side effects
6. **Escalate if needed** - Provide user with detailed information

The key is methodical debugging rather than guessing. With this approach, you'll find the root cause and fix the actual problem rather than applying band-aids.
