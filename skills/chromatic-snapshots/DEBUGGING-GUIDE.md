# Chromatic Snapshot Debugging Guide

Full step-by-step process for diagnosing and resolving unexpected snapshot failures.

## 9-Step Debugging Workflow

Follow these steps in order when a snapshot fails unexpectedly:

### 1. Rerun the Build

Click the **Rerun** button in Chromatic to confirm whether the change is real or flaky.

- **Identical changes in both runs** → real UI change, proceed with review
- **Different changes between runs** → inconsistency in your code, continue debugging

### 2. Open the Trace Viewer

In the rerun build, click the browser button in the "Traces" column. The Trace Viewer provides a full replay of the capture environment including network, console, and DOM state.

### 3. Check the Network Tab

Look for failed or slow resource loads:
- Fonts returning 404 or timing out
- CSS files not loading
- Images from external CDNs failing
- API calls (MSW or otherwise) not resolving

### 4. Inspect the DOM

Verify element structure, applied styles, and layout at capture time:
- Are expected elements present in the DOM?
- Are CSS classes applied correctly?
- Is any element hidden by `overflow: hidden` or `display: none`?
- Are portals rendering outside the expected container?

### 5. Review Screenshot Metadata

Check viewport size and clip rectangle for positioning issues:
- Does the viewport match your expected dimensions?
- Is the clip rectangle capturing the full component?
- Are `position: fixed` or `position: absolute` elements outside bounds?

### 6. Identify the Category

Match the symptom to these categories:
- **Blank/missing content** → resource loading, animation timing, or container sizing
- **Shifted/cut off layout** → overflow, viewport sizing, or font metrics
- **Flaky diffs** → non-determinism, animations, or timing (see `FLAKY-SNAPSHOTS.md`)
- **Wrong data displayed** → MSW handler issues (see MSW section below)

### 7. Apply the Fix

Based on the category, apply the appropriate fix:
- Use `delay` or assertion-based waits for timing issues
- Serve resources as static files for loading issues
- Preload fonts in `preview-head.html` for font timing
- Disable or await animations for animation issues
- Use `chromatic-ignore` for intentionally volatile regions

### 8. Verify Locally

Run your Storybook build locally and check for issues:

```bash
npm run build-storybook && npx http-server storybook-static -o
```

Check the browser console for warnings, failed requests, and rendering errors.

### 9. Rerun Again

Push the fix and confirm the snapshot is now stable. If the snapshot still fails, return to step 2 with the new Trace Viewer data.

---

## Debugging Mocked Data Issues (MSW)

When stories using MSW don't snapshot correctly, follow this diagnostic path:

1. **Verify MSW initialization** — MSW must be initialized in `.storybook/preview.js` per [msw-storybook-addon docs](https://github.com/mswjs/msw-storybook-addon)
2. **Check package versions** — Ensure `msw` and `msw-storybook-addon` are up to date
3. **Build and serve locally** — Check console for MSW warnings (handler mismatches, unhandled requests)
4. **Match only the path** — In handler URLs, match the path only; access query params via `req.url.searchParams.get()`
5. **Preload required CSS** — Add any CSS dependencies to `.storybook/preview-head.html`
6. **Add interaction tests** — Assert mocked data is rendered before snapshot fires
7. **Use `delay` as last resort** — If async data needs extra time after assertions pass

---

## Known Limitations (No Workaround)

These are expected Chromatic behaviors — not bugs:

- **Emojis look different** — Linux capture environment uses a different emoji font than macOS/Windows
- **Videos not rendering** — Chromatic hides videos to prevent flakiness; poster image is shown if specified
- **Scrollbars missing** — Chromatic disables scrollbars intentionally; test scrollbar styling outside Chromatic

## When to Contact Chromatic Support

Escalate to Chromatic support if:
- The Trace Viewer shows correct rendering but the snapshot is still wrong
- Network requests succeed in Trace Viewer but content doesn't appear
- The issue reproduces only in Chromatic's infrastructure (not locally)
- You suspect a Chromatic infrastructure or browser version issue

Provide: build URL, story name, Trace Viewer screenshots, and local reproduction steps.
