# Fixing Flaky Chromatic Snapshots

Flakiness means the same code produces different snapshots across builds. Chromatic continuously captures screenshots until two match or timeout is reached.

## Rerun Workflow

Click the **Rerun** button in Chromatic. Compare the original and rerun builds:

- **Identical changes** → real UI change, proceed with review
- **Different changes** → inconsistency in your code, debug further

Always rerun before investigating — it separates real changes from flakiness.

## Flakiness Sources and Fixes

| Source | Solution |
|---|---|
| Randomness (IDs, data) | Use `seedrandom` or hard-code test data |
| Animations (CSS) | Chromatic pauses CSS animations automatically; set `pauseAnimationAtEnd: false` to pause at first frame |
| Animations (JS / framer-motion) | Disable via `MotionGlobalConfig.skipAnimations = isChromatic()` |
| Current date/time | Mock with `mockdate` or similar |
| Unpredictable resource hosts | Serve fonts/images as static files |
| Image CDN compression | Pin compression settings; serve images statically |
| Web font timing | Preload fonts with `<link rel="preload">` in `preview-head.html` |
| Slow UI settling | Add `delay` or use interaction test assertions |
| Intentional randomness | Use `isChromatic()` to stabilize only in Chromatic, or disable the snapshot |
| Iframes below viewport | Ignore the element or test the iframe in isolation |

## Code Examples

### Seeding Randomness

```javascript
// Storybook — preview.js
import seedrandom from "seedrandom";
import isChromatic from "chromatic/isChromatic";

if (isChromatic()) {
  Math.random = seedrandom("chromatic-seed");
}
```

### Mocking Dates

```javascript
// Storybook — preview.js
import MockDate from "mockdate";
import isChromatic from "chromatic/isChromatic";

if (isChromatic()) {
  MockDate.set("2024-01-15T12:00:00Z");
}
```

### Disabling JS Animations

```javascript
// Storybook — .storybook/preview.js
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

### Preloading Fonts

```html
<!-- .storybook/preview-head.html -->
<link rel="preload" href="/fonts/Inter-Regular.woff2" as="font" type="font/woff2" crossorigin />
```

## Prevention Checklist

Before adding a new story or visual test, verify:

- [ ] No `Date.now()` or `Math.random()` without seeding
- [ ] No JS animations running without disabling or awaiting
- [ ] Fonts served as static files and preloaded
- [ ] Images served locally, not from external CDNs
- [ ] MSW handlers match only the path, not full URLs
- [ ] Interaction tests assert data is rendered before snapshot
- [ ] No `100vh` or `100vw` assumptions that may clip content
- [ ] Intentionally volatile content uses `chromatic-ignore`
