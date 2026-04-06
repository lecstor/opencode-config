---
name: chromatic-snapshots
description: Guide for diagnosing and fixing snapshot failures, flakiness, and comparison issues in Chromatic.
---

Stable snapshots are the foundation of visual regression testing. Every inconsistency you eliminate prevents a false positive from blocking your team.

## When to Use This Skill

Activate when you encounter:
- "Why is my snapshot blank / cut off / different from local?"
- Flaky visual diffs that appear and disappear between builds
- Resources (fonts, images) missing from snapshots
- Animations causing inconsistent captures
- Mocked data not appearing in snapshots
- Portal components (modals, tooltips) not captured
- Cross-browser rendering differences

## Common Snapshot Failures

| Symptom | Root Cause | Fix |
|---|---|---|
| Blank snapshot | `animateIn` effect or `position: fixed` without dimensions | Add `chromatic: { delay: 300 }` or wrap in a sized container |
| Missing images/fonts | Resource didn't load within 15s timeout | Serve as static files or use a placeholder service |
| Content cut off vertically | `overflow: hidden` or `height: 100vh` on ancestor | Remove overflow constraint or set explicit height via decorator |
| Portal not captured | Component renders outside viewport | Reposition component or adjust viewport size via modes |
| Mocked data not appearing | MSW handler mismatch or loading state captured | Add interaction test assertions; match API path only in handlers |
| Ignored elements still diff | Baseline and new snapshot have different dimensions | Ensure ignored elements keep identical dimensions across builds |

## Essential Best Practices

- ✅ Serve fonts, images, and assets as static files — never rely on external CDNs
- ✅ Use interaction test assertions to confirm UI state before capture (preferred over fixed `delay`)
- ✅ Use `seedrandom` or `mockdate` to eliminate non-determinism (dates, IDs, random data)
- ✅ Preload custom fonts in `.storybook/preview-head.html`
- ✅ Use `isChromatic()` to conditionally stabilize behavior only in Chromatic
- ❌ Don't leave JS animations running without disabling or awaiting completion
- ❌ Don't ignore flaky snapshots — find and fix the root cause
- ❌ Don't assume viewport-relative sizing (`100vh`) shows all content

## Quick Config Reference

| Option | Purpose |
|---|---|
| `chromatic: { delay: <ms> }` | Wait before capturing |
| `chromatic: { pauseAnimationAtEnd: false }` | Pause CSS animation at first frame |
| `chromatic: { diffThreshold: <0-1> }` | Sensitivity of pixel comparison |
| `.chromatic-ignore` / `data-chromatic="ignore"` | Exclude region from diff |
| `isChromatic()` | Detect Chromatic environment at runtime |
| `staticDirs` (Storybook config) | Serve local assets for stories |

## Useful Links

- [Chromatic Troubleshooting Snapshots](https://www.chromatic.com/docs/troubleshooting-snapshots/)
- [Animations](https://www.chromatic.com/docs/animations/)
- [Delay](https://www.chromatic.com/docs/delay/)
- [Font Loading](https://www.chromatic.com/docs/font-loading/)
- [Resource Loading](https://www.chromatic.com/docs/resource-loading/)
- [Ignoring Elements](https://www.chromatic.com/docs/ignoring-elements/)
- [isChromatic()](https://www.chromatic.com/docs/ischromatic/)
- [Disable Snapshots](https://www.chromatic.com/docs/disable-snapshots/)

## Supporting Files

For deeper guidance, load these companion files:

- `DEBUGGING-GUIDE.md` — Full 9-step debugging workflow, Trace Viewer usage, MSW diagnostics
- `FLAKY-SNAPSHOTS.md` — Flakiness causes, rerun workflow, prevention checklist
- `EXAMPLES.md` — Code examples for delay, animations, assertions, and ignoring elements
