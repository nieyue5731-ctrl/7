# Risk Register - Terraria Ultra Refactoring

## Active Risks

### R1: Script Load Order Dependency (HIGH)
- **Description**: The original monolithic file had implicit load order guarantees. Splitting into separate `<script>` files introduces potential race conditions if browser caching or network issues cause out-of-order loading.
- **Mitigation**: All scripts use synchronous `<script src>` tags (not `defer`/`async`), preserving sequential execution order identical to inline scripts. Load order in `index.html` exactly matches original file order.
- **Status**: MITIGATED

### R2: Global Namespace Pollution (MEDIUM)
- **Description**: Original code uses 30+ `window.*` assignments. Extraction preserves all of them, but splitting files means each assignment must be visible to subsequent files.
- **Mitigation**: All `window.TU`, `window.ObjectPool`, `window.VecPool`, etc. assignments are preserved in their original positions. Load order ensures definitions precede usage.
- **Status**: MITIGATED

### R3: Monkey-Patch Chain Integrity (HIGH)
- **Description**: 18 patch layers override class prototypes in a specific chain. The final behavior depends on the last override winning. Incorrect ordering would break rendering, input, physics, etc.
- **Mitigation**: Patches are loaded in exact original order (verified by line number analysis). Each patch file is extracted verbatim without modification.
- **Status**: MITIGATED

### R4: CSS Specificity Changes (MEDIUM)
- **Description**: Moving CSS from inline `<style>` blocks to external `<link>` files does not change specificity, but changes load timing. CSS loaded via `<link>` in `<head>` should be equivalent to inline `<style>` in `<head>`.
- **Mitigation**: All CSS links are in `<head>`, same as original `<style>` blocks. External CSS files maintain same selector order.
- **Status**: MITIGATED

### R5: Inline Style on Line 17 (LOW)
- **Description**: The original file had a single-line minified `<style>` block. Extraction preserves the content but it was on one very long line.
- **Mitigation**: Content extracted verbatim. CSS is valid regardless of line breaks.
- **Status**: MITIGATED

### R6: Worker Inline Blob (MEDIUM)
- **Description**: The WorldWorkerClient builds a Worker from an inline Blob URL using string concatenation. This code is inside patch-12-worker-client.js and was extracted verbatim.
- **Mitigation**: No changes to worker construction logic. The `parts.push()` pattern is preserved exactly.
- **Status**: MITIGATED

### R7: PatchManager.once() Guard Flags (LOW)
- **Description**: Several patches use `PatchManager.once('flag_name')` or `__tu_xxx` flags to prevent double-application. With separate files, these flags still work correctly since scripts execute sequentially.
- **Mitigation**: No change to guard logic. Sequential loading preserves idempotency.
- **Status**: MITIGATED

### R8: HTML Structure Between head/body (LOW)
- **Description**: Original file had `<script>` tags between `</head>` and `<body>`, which is invalid HTML. These scripts (EventManager, ParticlePool, PERF_MONITOR) are now loaded inside `<body>`.
- **Mitigation**: Browser behavior is identical for scripts in `<body>` vs between head/body. DOM elements referenced by these scripts are defined later, but the scripts only define classes/objects, not query the DOM.
- **Status**: MITIGATED

## Closed Risks

None yet - all risks are being actively monitored.
