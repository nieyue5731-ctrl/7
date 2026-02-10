# Refactor Journal - Terraria Ultra Modular Refactoring

## Overview

Refactoring of `index (92).html` (24,673 lines, single monolithic HTML file) into a modular multi-file architecture.

---

## Phase 0: Baseline Snapshot

### Dependency Graph Summary

The original file has the following dependency chain (load order critical):

1. **CSS** (6 `<style>` blocks, ~2100 lines total)
2. **TU_Defensive IIFE** (~417 lines) - Global error handlers, TypeGuards, SafeMath, BoundaryChecks, InputValidator, SafeAccess, WorldAccess, PatchManager
3. **Phase 3 Modules** (between `</head>` and `<body>`) - EventManager, ParticlePool, PERF_MONITOR delegate, safe utils (safeGet, clamp, lerp), RingBuffer
4. **HTML Body** - Canvas, HUD elements, overlays, mobile controls
5. **Core Namespace** (~673 lines) - ObjectPool, VecPool, ArrayPool, MemoryManager, EventUtils, PerfMonitor, TextureCache, BatchRenderer, LazyLoader
6. **Naming Aliases** - TU namespace lazy aliases
7. **Loading Particles** - DOM particle init
8. **Utils & DOM** (~252 lines) - Utils object (clamp, lerp, hexToRgb, isMobile, etc.), DOM helper
9. **GameSettings** (~169 lines)
10. **Toast** (~22 lines)
11. **FullscreenManager** (~65 lines)
12. **AudioManager** (~120 lines)
13. **SaveSystem** (~308 lines)
14. **UX/UI Wiring** (~468 lines) - wireUXUI, applyInfoHintText, overlay logic
15. **Constants** (~341 lines) - CONFIG, BLOCK enum, BLOCK_DATA, lookup TypedArrays
16. **NoiseGenerator** (~63 lines)
17. **TextureGenerator** (~619 lines)
18. **Crafting Recipes** (~80 lines)
19. **WorldGenerator** (~2226 lines)
20. **ParticleSystem** (~114 lines)
21. **DroppedItem/Manager** (~390 lines)
22. **AmbientParticles** (~158 lines)
23. **Player** (~456 lines)
24. **TouchController** (~184 lines)
25. **Renderer** (~1077 lines)
26. **CraftingSystem** (~176 lines)
27. **UIManager** (~52 lines)
28. **QualityManager** (~330 lines)
29. **Minimap** (~491 lines)
30. **Hotbar Renderer** (~116 lines)
31. **UI Extensions** (~88 lines)
32. **InventoryUI** (~781 lines)
33. **InputManager** (~275 lines)
34. **InventorySystem** (~88 lines)
35. **Game** (~990 lines)
36. **Patch Layers** (~10,000 lines total, 18 separate patches)
37. **Bootstrap** (~30 lines)
38. **Health Check** (~65 lines)

### Patch Chain End-Version Locator Table

| Method/Function | Original Definition | Final Patch | Final Version Location |
|---|---|---|---|
| `Renderer.renderWorld` | Script 24 | patch-10, patch-12 (worker) | `js/patches/patch-12-worker-client.js` |
| `Renderer.renderSky` | Script 24 | patch-03 (render-sky) | `js/patches/patch-03-render-sky.js` |
| `Renderer._getSkyBucket` | Script 24 | patch-03 | `js/patches/patch-03-render-sky.js` |
| `Game._spreadLight` | Script 34 | patch-18 (final) | `js/patches/patch-18-spreadlight.js` |
| `Game.loop` | Script 34 | patch-01 (game-loop) | `js/patches/patch-01-game-loop.js` |
| `Game.init` | Script 34 | patch-12 (worker) | `js/patches/patch-12-worker-client.js` |
| `Game._writeTileFast` | Script 34 | patch-12 (worker) | `js/patches/patch-12-worker-client.js` |
| `Game._updateLight` | Script 34 | patch-12 (worker) | `js/patches/patch-12-worker-client.js` |
| `TouchController` | Script 23 | patch-11 | `js/patches/patch-11-touch-controller.js` |
| `WorldGenerator.generate` | Script 18 | patch-12 (worker) | `js/patches/patch-12-worker-client.js` |
| `SaveSystem.applyToWorld` | Script 12 | patch-12 (worker) | `js/patches/patch-12-worker-client.js` |
| `Renderer.drawTile` | Script 24 | patch-17 (runtime-opt) | `js/patches/patch-17-runtime-opt.js` |

### Behavior Baseline Checklist

- [ ] Loading screen displays correctly
- [ ] World generates without errors
- [ ] Player movement (WASD/touch)
- [ ] Mining (left click/touch button)
- [ ] Block placement (right click/touch button)
- [ ] Lighting propagation correct
- [ ] Water physics working
- [ ] UI overlays (pause/settings/help/inventory/crafting)
- [ ] Save/load functional
- [ ] Weather system active
- [ ] Audio plays on interaction
- [ ] Mobile controls work
- [ ] Minimap renders
- [ ] Fullscreen toggle works
- [ ] Toast notifications display
- [ ] No console errors (ReferenceError/TypeError)

---

## Phase 1: Safe Cleanup - Extraction & Modularization

### Actions Taken

1. **Renamed** `index (92).html` -> kept as reference; new `index.html` created
2. **Extracted 6 CSS style blocks** into organized files under `css/`
3. **Extracted 55 script blocks** into organized JS modules under `js/`
4. **Created modular `index.html`** with proper `<link>` and `<script>` references maintaining exact load order

### Dead Code Identification (Evidence-Based)

The following were identified as dead/unused code but **preserved in extraction** for safety:

| Symbol | Location | Evidence | Status |
|---|---|---|---|
| `BatchRenderer` | `js/performance/pools-and-cache.js` | No references found outside definition | Preserved (no callers) |
| `LazyLoader` | `js/performance/pools-and-cache.js` | No references found outside definition | Preserved (no callers) |
| `RingBuffer` | `js/core/event-manager.js` | Only `window.RingBuffer = RingBuffer` export, no usage | Preserved |
| `PERF_MONITOR` | `js/performance/perf-monitor-delegate.js` | Delegates to PerfMonitor; some patches may reference | Preserved |

### Utility Function Dedup Status

| Function | Copies Found | Canonical Location |
|---|---|---|
| `clamp` | SafeMath.clamp, BoundaryChecks.clamp, Utils.clamp, window.clamp | All preserved; window.clamp is guarded with `typeof === 'undefined'` |
| `lerp` | SafeMath (none), Utils.lerp, window.lerp, NoiseGenerator._lerp | Utils.lerp is canonical; window.lerp is guarded fallback |
| `safeGet` | SafeAccess.get, window.safeGet, BoundaryChecks.safeArrayAccess | window.safeGet is guarded; TU_Defensive versions have priority |

### Hot-Path Fixes (Already Present in Source)

The original code already contains these optimizations:
- `VecPool.release`: Uses `_pooled` tag (O(1)) instead of `includes()` (O(n))
- `ArrayPool.release`: Uses `_pooled` tag (O(1))
- `PerfMonitor.getMinFPS`: Still uses `Math.max(...validSamples)` - potential stack overflow for large arrays (preserved as-is; max 60 samples so safe)

---

## Phase 2: CSS Extraction

### Files Created

| File | Source Lines | Contents |
|---|---|---|
| `css/mobile-controls.css` | Line 17 (inline) | Mobile controls, toast, overlay base styles |
| `css/main.css` | Lines 18-1755 | Variables (:root), reset, HUD, hotbar, minimap, mining bar, crafting, inventory, overlays, mobile, responsive |
| `css/theme-frost.css` | Lines 1757-1977 | Frost/glass theme extensions |
| `css/loading.css` | Lines 1978-2056 | Loading screen styles |
| `css/performance.css` | Lines 2058-2098 | Performance debug styles |
| `css/low-perf.css` | Lines 2519-2522 | Low-performance mode utility classes |

### CSS Issues Preserved (Not Altered)

- Multiple `:root` blocks preserved in `main.css` (consolidation would require visual regression testing)
- `!important` usage preserved (removal requires specificity analysis with browser)
- `will-change: contents` preserved (invalid but harmless)
- `shimmer` animation uses `left` property (transform would be better but changes behavior)

---

## Phase 3: Monkey-Patch Preservation

All 18 patch layers are preserved in exact original form under `js/patches/`. They are loaded in the exact same order as the original file, ensuring the same prototype chain overrides apply.

**Rationale**: Merging patches into class definitions is extremely high risk without runtime testing. The patches are preserved separately to maintain 100% behavioral equivalence. Each patch file is independently identifiable and can be merged into canonical classes in a future phase with proper testing.

---

## Phase 4-6: Structural Notes

World data structures, render pipeline, and architecture remain unchanged in this extraction phase. The code has been organized into logical modules but internal logic is preserved exactly as-is.

---

## Phase 7: HTML Validity Improvements

- Moved all `<script>` tags to inside `<body>` (were between `</head>` and `<body>` in original)
- Added `role="progressbar"` to progress bars
- Added `aria-live="polite"` to toast container and loading status
- Added `aria-label` to health/mana bars
- All overlays retain `aria-hidden` attributes

---

## Phase 8: Final State

The project is now split into:
- 6 CSS files
- 42 JS files organized by domain
- 1 clean index.html
- Documentation in `docs/`
