# Playwright E2E Testing Skill - Content Index

## Quick Navigation Guide

### 📋 What to Read When

**I'm new to Playwright:**
1. Start with **README.md** (overview)
2. Read **SKILL.md** sections:
   - "Core Philosophy" (foundational mindset)
   - "Locator Strategy" (the critical hierarchy)
   - "Web-First Assertions" (how it works)
3. Check **EXAMPLES.md** "Form Interactions" (simplest case)

**My tests are flaky/slow:**
1. Read **SKILL.md** → "Why Manual Waits Are Harmful"
2. Check **ANTI-PATTERNS.md** → "Anti-Pattern #1" (setTimeout)
3. Replace all setTimeout with web-first assertions from **EXAMPLES.md**

**I need specific examples:**
1. Search **EXAMPLES.md** for your scenario:
   - Forms → "Form Interactions"
   - Lists → "List and Table Interactions"
   - Modals → "Modal and Dialog Interactions"
   - Errors → "Error State Testing"
   - Success → "Success Message Verification"

**I'm making mistakes in my tests:**
1. Check **ANTI-PATTERNS.md** for the red flag you see
2. Read "Why It's Harmful" section
3. Study "The Right Way" code example
4. Apply the pattern to your tests

---

## 📑 File-by-File Breakdown

### README.md (8.1 KB)
**Purpose**: Overview and quick start guide

**Sections**:
- Contents summary
- Key principles (locator hierarchy, web-first testing, etc.)
- Common scenarios (quick snippets)
- Getting started guide
- Red flags (anti-patterns overview)
- By the numbers (stats about the skill)

**Use when**: You need a quick reference or overview

---

### SKILL.md (24 KB, 780 lines)
**Purpose**: Comprehensive philosophy and deep dive

**Sections**:

1. **Overview** (philosophy & why it matters)
2. **Core Philosophy** (user-centric testing)
3. **Locator Strategy: Priority Order** (8 levels, from best to never)
   - getByRole (most preferred)
   - getByLabel
   - getByPlaceholder
   - getByAltText
   - getByTitle
   - getByTestId (use sparingly)
   - getByText (fallback)
   - CSS/XPath (NEVER)

4. **Web-First Assertions with Retry-Ability** (auto-waiting explained)
   - How auto-waiting works
   - Key assertions
   - Why manual waits are harmful

5. **Auto-Waiting and Actionability Checks** (what Playwright does for you)
   - What auto-waits for
   - Auto-waiting in assertions
   - Custom timeouts

6. **Test Isolation and Independent Design** (avoiding test interdependence)
   - Test isolation principle
   - Using beforeEach
   - Creating fixtures
   - Database cleanup

7. **Common Pitfalls and Anti-Patterns** (6 key mistakes)
   - Testing implementation instead of behavior
   - Not waiting for actual conditions
   - Overusing test IDs
   - CSS and XPath chains
   - Ignoring errors
   - Test order dependency

8. **Accessibility as a Side Benefit** (free accessibility testing)
   - Why semantic locators = accessibility
   - Screen reader equivalence
   - Accessibility testing checklist

9. **Complete Test Structure Example** (production-ready test)

10. **Key Takeaways** (summary)

**Use when**: You want to understand the philosophy deeply

---

### EXAMPLES.md (26 KB, 954 lines)
**Purpose**: Practical, copy-paste-ready code examples

**Sections** (with 50+ complete examples):

1. **Form Interactions**
   - Simple form with validation
   - Form validation errors
   - Checkbox and radio interactions
   - Select dropdown
   - Password field with show/hide toggle

2. **List and Table Interactions**
   - Filtering a list
   - Sorting a list
   - Pagination
   - Editable table cells

3. **Modal and Dialog Interactions**
   - Confirmation dialog
   - Form modal
   - Modal with tabs

4. **Error State Testing**
   - API error handling
   - Network timeout error
   - Validation error with field highlighting

5. **Success Message Verification**
   - Toast notification
   - Success page
   - Inline success message

6. **Accessible Form Validation**
   - ARIA live region for errors
   - Loading state with ARIA

7. **Keyboard Navigation**
   - Tab through form
   - Escape to close modal

8. **Complex Interactions**
   - Drag and drop
   - File upload
   - Multi-select with search

9. **DO's and DON'Ts**
   - Side-by-side comparisons
   - ✅ GOOD vs ❌ BAD for each pattern

10. **Quick Reference**
    - Cheat sheet of locators
    - Cheat sheet of actions
    - Cheat sheet of assertions

**Use when**: You need working code examples for your specific scenario

---

### ANTI-PATTERNS.md (21 KB, 738 lines)
**Purpose**: Detailed analysis of what NOT to do

**Sections** (10 detailed anti-patterns):

1. **Manual Waits (setTimeout)**
   - The problem
   - Why it's harmful
   - The right way
   - Real example

2. **Using waitForTimeout/waitForSelector**
   - The problem
   - Why it's harmful
   - The right way

3. **Test IDs Everywhere**
   - The problem
   - Why it's harmful
   - The right way

4. **CSS Selectors and XPath Chains**
   - The problem
   - Why it's harmful
   - Real example of brittleness
   - The right way

5. **Not Using Web-First Assertions**
   - The problem
   - Why it's harmful
   - The right way

6. **Testing Implementation Details**
   - The problem
   - Why it's harmful
   - Real example
   - The right way

7. **Flaky Selectors That Break Easily**
   - The problem
   - Why it's harmful
   - Comparison table
   - The right way

8. **Test Interdependence**
   - The problem
   - Why it's harmful
   - The right way
   - Helper function example

9. **Ignoring Errors and Exceptions**
   - The problem
   - Why it's harmful
   - The right way

10. **Using page.locator Without Visibility Checks**
    - The problem
    - Why it's harmful
    - The right way

**Final Section**: Summary with red flags checklist

**Use when**: You see a test pattern that seems wrong, or you want to avoid common mistakes

---

### DEBUGGING-FAILING-TESTS.md (29 KB, 890+ lines)
**Purpose**: Systematic guide for debugging tests that fail despite appearing correct

**Sections**:

1. **The Problem Scenario** (when code and test appear correct but test fails)
2. **Debugging Workflow** (step-by-step systematic approach)
   - **Step 1: Manual Replication** (replicate test steps manually)
   - **Step 2: Log Analysis** (examine Playwright traces, console logs, network activity)
   - **Step 3: Identify the Category** (5 categories of issues)
     - Playwright Issues (test problems - locators, timing, assertions)
     - Code Issues (application bugs - rendering, state, events)
     - Environment Issues (setup/config - mocking, seeding, variables)
     - Timing/Race Conditions (async synchronization issues)
     - Data Issues (missing or incorrect test data)

3. **Common Patterns & Solutions** (6 frequently encountered patterns)
   - "Element not found" but element exists (locator mismatch)
   - "Timeout waiting" but appears manually (timing issue)
   - "Text doesn't match" despite correct code (whitespace/dynamic content)
   - "Click succeeded but nothing happened" (preconditions)
   - "Test passes locally but fails in CI" (environment difference)
   - "Test passed yesterday, fails today" (flaky test)

4. **When to Escalate to User** (how to inform user with all relevant details)
   - What to tell the user
   - Questions to ask the user
   - How to provide options
   - What failure details to include

5. **Tools & Commands Reference** (debugging tools and commands)
   - Run with debug mode
   - View traces
   - Generate reports
   - Test locators
   - Run specific tests

6. **Debugging Checklist** (quick reference)
   - Immediate checks
   - Element checks
   - Test checks
   - Environment checks
   - If still stuck

**Use when**: A test is failing even though the code and test appear correct, and you need a systematic debugging approach

---

## 🔍 Finding Specific Topics

### By Scenario
**Testing a form:**
- SKILL.md → "Locator Strategy" (for form fields)
- EXAMPLES.md → "Form Interactions" (complete examples)

**Testing a button click:**
- EXAMPLES.md → "Modal and Dialog Interactions" (button examples)
- SKILL.md → "Auto-Waiting and Actionability Checks"

**Testing a list:**
- EXAMPLES.md → "List and Table Interactions"
- SKILL.md → "Test Isolation" (for setup)

**Testing an API error:**
- EXAMPLES.md → "Error State Testing"
- ANTI-PATTERNS.md → "Anti-Pattern #1" (what NOT to do)

**Testing success messages:**
- EXAMPLES.md → "Success Message Verification"

**Keyboard testing:**
- EXAMPLES.md → "Keyboard Navigation"

### By Problem

**Tests are flaky:**
1. SKILL.md → "Why Manual Waits Are Harmful"
2. ANTI-PATTERNS.md → Patterns 1-2, 5
3. Replace with examples from EXAMPLES.md

**Tests are slow:**
1. SKILL.md → "Auto-Waiting and Actionability Checks"
2. ANTI-PATTERNS.md → "Anti-Pattern #1"
3. Remove all manual waits

**Tests break on refactoring:**
1. ANTI-PATTERNS.md → "Anti-Pattern #4" (CSS selectors)
2. ANTI-PATTERNS.md → "Anti-Pattern #3" (test IDs)
3. Replace with SKILL.md locator hierarchy

**Can't find elements:**
1. SKILL.md → "Locator Strategy: Priority Order"
2. EXAMPLES.md → Find matching scenario

**Tests depend on each other:**
1. ANTI-PATTERNS.md → "Anti-Pattern #8"
2. SKILL.md → "Test Isolation and Independent Design"

**False positive tests (pass when they shouldn't):**
1. ANTI-PATTERNS.md → "Anti-Pattern #9" (ignoring errors)
2. ANTI-PATTERNS.md → "Anti-Pattern #5" (immediate assertions)

**Test fails despite code and test appearing correct:**
1. DEBUGGING-FAILING-TESTS.md → "Debugging Workflow" (step-by-step approach)
2. DEBUGGING-FAILING-TESTS.md → "Step 1: Manual Replication" (verify actual behavior)
3. DEBUGGING-FAILING-TESTS.md → "Step 2: Log Analysis" (examine traces and logs)
4. DEBUGGING-FAILING-TESTS.md → "Step 3: Identify the Category" (categorize the issue)
5. DEBUGGING-FAILING-TESTS.md → "Common Patterns & Solutions" (find your pattern)
6. DEBUGGING-FAILING-TESTS.md → "When to Escalate" (if needed, inform user with details)

### By Technology/Feature

**Forms and validation:**
- EXAMPLES.md → "Form Interactions" + "Accessible Form Validation"
- SKILL.md → "Locator Strategy" → getByLabel section

**Tables and lists:**
- EXAMPLES.md → "List and Table Interactions"

**Dialogs and modals:**
- EXAMPLES.md → "Modal and Dialog Interactions"
- SKILL.md → "Locator Strategy" → getByRole (dialog role)

**File uploads:**
- EXAMPLES.md → "Complex Interactions" → File Upload

**Drag and drop:**
- EXAMPLES.md → "Complex Interactions" → Drag and Drop

**Accessibility:**
- SKILL.md → "Accessibility as a Side Benefit"
- EXAMPLES.md → "Accessible Form Validation"
- EXAMPLES.md → "Keyboard Navigation"

**API mocking:**
- EXAMPLES.md → "Error State Testing" (route examples)

---

## 📊 Content Statistics

| Metric | Value |
|--------|-------|
| Total Lines | 2,700+ |
| Total Size | 80 KB |
| Number of Files | 4 |
| Code Examples | 50+ |
| Anti-Patterns Covered | 10 |
| Locator Types | 8 |
| Assertion Types | 12+ |
| Scenarios Covered | 30+ |

---

## 🎯 Learning Path

### Beginner (2-3 hours)
1. Read README.md (10 min)
2. Read SKILL.md sections:
   - "Core Philosophy" (10 min)
   - "Locator Strategy" (30 min)
   - "Web-First Assertions" (20 min)
3. Read EXAMPLES.md:
   - "Form Interactions" (20 min)
   - "Quick Reference" (10 min)

### Intermediate (3-5 hours)
1. Read complete SKILL.md (1.5 hours)
2. Read complete EXAMPLES.md (2 hours)
3. Practice writing one test file

### Advanced (5+ hours)
1. Read ANTI-PATTERNS.md completely (1 hour)
2. Review existing test suite for anti-patterns
3. Refactor problematic tests
4. Implement skill patterns project-wide

---

## 💾 How to Use This Skill

### Step 1: Choose Your Path
- **Need examples?** → Go to EXAMPLES.md
- **Need philosophy?** → Go to SKILL.md
- **Need guidance on mistakes?** → Go to ANTI-PATTERNS.md
- **Not sure?** → Start with README.md

### Step 2: Find Your Scenario
Use the content index above to locate your specific use case

### Step 3: Read the Guidance
Each section explains:
- ❌ What NOT to do (and why)
- ✅ What to do instead
- 💡 Why it works

### Step 4: Copy and Adapt
All code examples are production-ready and can be adapted to your app

### Step 5: Verify Against Anti-Patterns
Check your code against ANTI-PATTERNS.md to ensure you're not:
- Using manual waits
- Using CSS/XPath
- Testing implementation details
- Creating test dependencies
- Ignoring errors

---

## 🔗 Cross-References

**Topic**: Waiting for elements
- SKILL.md → "Web-First Assertions with Retry-Ability"
- SKILL.md → "Auto-Waiting and Actionability Checks"
- ANTI-PATTERNS.md → "Anti-Pattern #1" (manual waits)
- ANTI-PATTERNS.md → "Anti-Pattern #2" (waitForSelector)
- EXAMPLES.md → All examples use proper waiting

**Topic**: Form testing
- EXAMPLES.md → "Form Interactions" (complete section)
- EXAMPLES.md → "Accessible Form Validation"
- SKILL.md → "Locator Strategy" → getByLabel section
- EXAMPLES.md → "DO's and DON'Ts" → form examples

**Topic**: Selector choice
- SKILL.md → "Locator Strategy: Priority Order" (complete guide)
- ANTI-PATTERNS.md → "Anti-Pattern #3" (test IDs)
- ANTI-PATTERNS.md → "Anti-Pattern #4" (CSS/XPath)
- ANTI-PATTERNS.md → "Anti-Pattern #7" (flaky selectors)

**Topic**: Test design
- SKILL.md → "Test Isolation and Independent Design"
- ANTI-PATTERNS.md → "Anti-Pattern #8" (test interdependence)
- EXAMPLES.md → All examples show proper isolation

**Topic**: Accessibility
- SKILL.md → "Accessibility as a Side Benefit"
- EXAMPLES.md → "Accessible Form Validation"
- EXAMPLES.md → "Keyboard Navigation"

---

**Total Coverage**: This skill provides comprehensive coverage of Playwright E2E testing following React Testing Library best practices, with 50+ production-ready examples and 10 detailed anti-pattern explanations.

**Time to Proficiency**: 
- Basic understanding: 2-3 hours
- Practical proficiency: 8-10 hours
- Expert level: 20+ hours
