# Chromatic Snapshot Examples

Real-world code examples for common snapshot issues across Storybook, Playwright, and Cypress.

## Delay Configuration

Use `delay` (milliseconds) to wait before capture:

```javascript
// Storybook — story or component level
parameters: {
  chromatic: { delay: 300 },
}
```

```javascript
// Playwright
test.use({ delay: 300 });
```

```javascript
// Cypress
it("test", { env: { delay: 300 } }, () => { /* ... */ });
```

## Assertion-Based Delay (Preferred Over Fixed Delay)

Wait for a specific UI state instead of guessing timing:

```javascript
// Storybook play function
play: async ({ canvas }) => {
  await waitFor(async () => {
    await expect(canvas.getByText("Loaded")).toBeInTheDocument();
  });
}
```

```javascript
// Playwright
await expect(page.getByText("Loaded")).toBeVisible({ timeout: 10000 });
```

```javascript
// Cypress
cy.get(".loaded-indicator", { timeout: 10000 }).should("exist");
```

## Animation Control

### Pause CSS Animation at First Frame

```javascript
// Storybook — story parameters
parameters: {
  chromatic: { pauseAnimationAtEnd: false },
}
```

### Disable JS Animations Globally

```javascript
// .storybook/preview.js
import { MotionGlobalConfig } from "framer-motion";
import isChromatic from "chromatic/isChromatic";

MotionGlobalConfig.skipAnimations = isChromatic();
```

```javascript
// Playwright
await page.addInitScript(() => {
  window.disableAnimations = true;
});
```

### Wait for Animation Completion via Play Function

```javascript
// Hook: track animation state
export const useAnimatedState = () => {
  const [isComplete, setIsComplete] = useState(false);
  const handleAnimationComplete = useCallback(() => setIsComplete(true), []);
  return {
    isComplete,
    animationProps: {
      onAnimationComplete: handleAnimationComplete,
      "data-animation": isComplete ? "complete" : "playing",
      "data-testid": "animated-element",
    },
  };
};

// Story: assert animation finished before snapshot
play: async ({ canvas }) => {
  await waitFor(
    async () => {
      await expect(canvas.getByTestId("animated-element"))
        .toHaveAttribute("data-animation", "complete");
    },
    { timeout: 5000 }
  );
}
```

## Ignoring Elements

Prevent specific regions from triggering diffs:

```html
<!-- CSS class -->
<div class="chromatic-ignore">volatile content</div>

<!-- Data attribute -->
<div data-chromatic="ignore">volatile content</div>
```

⚠️ Both baseline and new snapshot must keep the ignored element's **dimensions identical** or diffs will still appear.

## Handling Cross-Origin Errors

```javascript
// Wrap window.parent calls to avoid cross-origin errors in Chromatic's iframe
try {
  const parentData = window.parent.someProperty;
} catch (e) {
  // Fallback for Chromatic capture environment
  const parentData = defaultValue;
}
```

## Handling srcset Cache Collisions

```javascript
// Add unique query parameter per test to avoid browser cache collisions
const imageSrc = `https://example.com/image.jpg?cachebuster=${storyId}`;

<img
  srcSet={`${imageSrc}&w=400 400w, ${imageSrc}&w=800 800w`}
  sizes="(max-width: 600px) 400px, 800px"
  alt="Product"
/>
```

## Scrollable Container Content

```javascript
// Problem: content hidden inside scrollable container
// Solution: use a decorator to expand the container in Chromatic
import isChromatic from "chromatic/isChromatic";

const decorators = [
  (Story) => (
    <div style={{ height: isChromatic() ? "auto" : "400px", overflow: isChromatic() ? "visible" : "auto" }}>
      <Story />
    </div>
  ),
];
```

## Portal Components (Modals, Tooltips)

```javascript
// Problem: portal renders outside viewport
// Solution: adjust viewport via Chromatic modes or reposition
parameters: {
  chromatic: {
    modes: {
      default: { viewport: { width: 1200, height: 900 } },
    },
  },
}
```

## Common Pitfalls

| Pitfall | Problem | Solution |
|---|---|---|
| Using `100vh` for containers | Content clips in Chromatic's viewport | Use explicit pixel heights or `auto` |
| External CDN for fonts | Font may not load in time | Use `staticDirs` and preload |
| Outdated MSW version | Handler API changes break mocks | Keep `msw` and `msw-storybook-addon` current |
| No interaction tests | Snapshot captures loading state | Add `play` function with assertions |
| Fixed delay only | Timing varies across environments | Combine with assertion-based waits |

## Reference Links

- [Chromatic Delay](https://www.chromatic.com/docs/delay/)
- [Chromatic Animations](https://www.chromatic.com/docs/animations/)
- [Chromatic Ignoring Elements](https://www.chromatic.com/docs/ignoring-elements/)
- [Chromatic Resource Loading](https://www.chromatic.com/docs/resource-loading/)
- [Chromatic Font Loading](https://www.chromatic.com/docs/font-loading/)
- [Chromatic isChromatic()](https://www.chromatic.com/docs/ischromatic/)
- [MSW Storybook Addon](https://github.com/mswjs/msw-storybook-addon)
