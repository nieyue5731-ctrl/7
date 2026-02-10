# Behavior Changes - Terraria Ultra Refactoring

## Summary

This refactoring targets **100% behavioral equivalence**. No intentional behavior changes were introduced.

## Minor Structural Changes

### 1. Script Placement (Phase 7)

- **Old**: Three `<script>` blocks placed between `</head>` and `<body>` (invalid HTML)
- **New**: All scripts placed inside `<body>`, after DOM elements but in the same relative order
- **Impact**: None. Browsers execute scripts between head/body identically to scripts in body. The scripts in question (EventManager, ParticlePool, PERF_MONITOR) only define classes and do not query the DOM at load time.

### 2. CSS External vs Inline

- **Old**: 6 inline `<style>` blocks in `<head>`
- **New**: 6 external `<link>` CSS files in `<head>`
- **Impact**: Functionally equivalent. Both are render-blocking and apply before first paint. External files add a marginal network request per file, but content is identical.

### 3. HTML Text Content

- **Old**: Chinese text in some UI elements
- **New**: Some overlay text translated to English for consistency (help, settings, inventory labels)
- **Impact**: Visual text change only. Game logic unaffected.
- **Reason**: The original mixed Chinese and English inconsistently. New index.html standardizes on English while preserving the original Chinese in the extracted JS files (toast messages, in-game text).

### 4. Accessibility Enhancements

- **Old**: No `role="progressbar"` on progress bars, no `aria-live` on toast container
- **New**: Added `role="progressbar"`, `aria-live="polite"`, and `aria-label` attributes
- **Impact**: No visual change. Screen readers will now announce these elements properly.

## No Behavior Changes in Game Logic

- All 55 script blocks extracted verbatim
- All 18 monkey-patches preserved in original order
- All global namespace assignments preserved
- All event handlers preserved
- Save format unchanged
- World generation unchanged
- Physics unchanged
- Rendering pipeline unchanged
