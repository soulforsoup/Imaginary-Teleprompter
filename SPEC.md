# Imaginary Teleprompter — SPEC.md

## 1. Concept & Vision

A professional-grade, free software teleprompter application for presenters, journalists, and content creators. Runs entirely in the browser with no backend, no accounts, and no data leaving the device. Two modes: a rich-text editor for writing scripts, and a distraction-free fullscreen teleprompter that reads those scripts with smooth hardware-accelerated scrolling.

The project has been running since 2015. This spec covers the modernization effort: updated dependencies, refreshed UI, and targeted UX fixes.

---

## 2. Design Language

### Current State
Outdated Bootstrap 3 UI, Courier monospace default font, flat gray buttons, modal dialogs from 2015. Functional but visually stale.

### Design Direction (Modernized)
Keep the purpose-driven, utilitarian feel — this is a production tool, not a marketing page. Clean up the visual noise, keep the functionality density, make it feel like professional broadcast software rather than a 2015 admin panel.

**Reference:** OBS Studio, vMix, Premiere Pro — dark production-tool aesthetic.

### Color Palette
| Token | Value | Use |
|-------|-------|-----|
| `--bg-deep` | `#0d0d0f` | Main background |
| `--bg-panel` | `#1a1a1e` | Cards, panels, sidebar |
| `--bg-input` | `#232328` | Inputs, editor area |
| `--accent` | `#e8a429` | Buttons, highlights, active states (warm amber — broadcast feel) |
| `--accent-hover` | `#f5b83d` | Hover state for accent |
| `--text-primary` | `#f0f0f0` | Primary text |
| `--text-muted` | `#888890` | Labels, secondary info |
| `--border` | `#2e2e34` | Dividers, input borders |
| `--success` | `#3ecf6e` | Save confirmation, success states |
| `--danger` | `#e84a4a` | Error, close, destructive actions |

### Typography
- **Primary:** `'DM Sans'` (Google Fonts) — clean, modern, legible at small sizes
- **Monospace:** `'JetBrains Mono'` (Google Fonts) — for the teleprompter display, clock, velocity readouts
- **Fallbacks:** `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`

### Spacing
- Base unit: `4px`
- Panel padding: `16px` (4 units)
- Component gaps: `8px` / `12px`
- Section gaps: `24px`

### Motion
- Transitions: `150ms ease` for interactive states
- Toast notifications: slide in from bottom-right, auto-dismiss after 3s
- No heavy animations — this is a production tool, not a landing page

---

## 3. Project Structure

```
imaginary-teleprompter/
├── index.html              # Landing / entry point
├── site.html               # Standalone web version (editor + teleprompter combined)
├── instance.html           # Iframe instance wrapper
├── teleprompter.html       # Full teleprompter view (in-frame or popup)
├── credits.html             # Credits page
├── blank.html              # Empty iframe target
├── css/
│   ├── reset.css           # CSS reset
│   ├── bootstrap.min.css   # Bootstrap 5 (updated from v3)
│   ├── editor.css          # Editor layout styles
│   ├── teleprompter.css   # Teleprompter view styles
│   ├── teleprompter-themes.css  # Theme options (mirrored, focused, etc.)
│   └── prompt.css          # Prompt/content styles
├── js/
│   ├── editor.js           # Main editor logic (2,000+ lines — target for refactor)
│   ├── teleprompter.js     # Prompter scrolling/controls (1,283 lines)
│   ├── data.manager.js     # IndexedDB/localStorage abstraction
│   ├── sidebar.js          # Script sidebar
│   ├── jquery.min.js       # jQuery 3.x (updated from 1.x)
│   ├── jquery.timer.js     # Timer plugin
│   ├── bootstrap.min.js     # Bootstrap JS (updated)
│   ├── bootstrap-slider.min.js  # Range slider control
│   ├── pep.min.js          # Pointer events polyfill
│   ├── jscolor.min.js      # Color picker
│   ├── jquery.qrcode.js    # QR code for remote control
│   ├── mobilecheck.js      # Mobile detection
│   └── teleprompter-themes.js  # Theme switching logic
├── img/                    # Static images, favicon
├── fonts/                  # Local font files
├── ckeditor/               # CKEditor 5 (updated from CKEditor 4)
├── tinymce/                # TinyMCE (alternative editor — keep for now)
├── package.json            # For Electron packaging
├── main.js                # Electron main process entry
├── SPEC.md                # This file
└── README.md
```

---

## 4. Dependencies — Current vs. Target

### Critical Outdated Dependencies

| Package | Current | Target | Risk |
|---------|---------|--------|------|
| jQuery | 1.x | 3.x | Low — v3 is API-compatible |
| Bootstrap | 3.x | 5.x | **Medium** — class changes, modal JS API changed |
| Electron | 1.8.8 (2018) | Latest stable (34.x) | **High** — breaking API changes, Squirrel replaced by NSIS |
| CKEditor | 4.x | 5.x | **Medium** — full rewrite of editor integration |
| electron-builder | 20.14.7 | Latest | **Medium** — config format changes |
| shelljs | 0.8.1 | 0.8.5 | Low |

### Update Strategy
1. **Phase 1:** Update jQuery 1.x → 3.x, Bootstrap 3 → 5, shelljs. Test editor + teleprompter still work.
2. **Phase 2:** CKEditor 4 → 5. The toolbar plugin API changed significantly.
3. **Phase 3:** Electron 1.8.8 → latest. Requires full electron-builder config rewrite.

**Important:** The web version (`site.html`) must remain fully functional without Electron. Electron updates only affect the desktop packaging.

---

## 5. UX Fixes

### 5.1 Save Button Feedback (Wave Button)

**Current behavior:** Clicking the wave/save button in the toolbar saves content to IndexedDB with no visual feedback. User doesn't know if save succeeded.

**Fix:** After a successful save, show a transient toast notification:
- Bottom-right corner
- Text: "Saved" with a checkmark icon
- Style: `--success` green background, white text
- Auto-dismiss after **3 seconds**
- Slide-in animation from bottom

```css
/* Toast styles */
.toast {
  position: fixed;
  bottom: 24px;
  right: 24px;
  background: var(--success);
  color: white;
  padding: 10px 16px;
  border-radius: 6px;
  font-family: 'DM Sans', sans-serif;
  font-size: 0.875rem;
  font-weight: 500;
  z-index: 9999;
  transform: translateY(80px);
  opacity: 0;
  transition: transform 200ms ease, opacity 200ms ease;
}
.toast.show {
  transform: translateY(0);
  opacity: 1;
}
```

**Implementation:** In `save()` function in `editor.js`, append the toast to `document.body`, add `.show` class, and remove it after 3 seconds via `setTimeout`.

---

### 5.2 Teleprompter Scroll Position on Enter/Exit Prompt Mode

**Current behavior:** When entering prompt mode, the teleprompter starts wherever it last was. When exiting, the scroll position stays wherever it ended. No reset to top on enter or exit.

**Fix — Two behaviors:**

1. **When entering prompt mode (`submitTeleprompter`):** The teleprompter should start from **the top of the script** (`scroll position = 0`). This ensures every prompting session starts fresh from the beginning, regardless of where the previous session ended.

2. **When exiting prompt mode (`restoreEditor`):** The teleprompter scroll position should **reset to top** before returning to the editor view. The editor view should also scroll to top.

**Implementation in `teleprompter.js`:**
- Before `submitTeleprompter` opens the teleprompter, reset `session.scrollPosition = 0` in IndexedDB
- In `restoreEditor()`, call `animate(0, 0)` (or equivalent reset) before closing the prompter
- The teleprompter's `init()` should read `session.scrollPosition` and start from that position — defaults to `0` (top)

```javascript
// In submitTeleprompter / updatePrompterData — reset scroll to top
session.scrollPosition = 0;
dataManager.setItem('IFTeleprompterSession', session, 1);

// In teleprompter.js init() — start from top (or saved position)
var startPosition = session.scrollPosition || 0;
if (flipV) {
    animate(0, -promptHeight + screenHeight);
} else {
    animate(0, startPosition);
}
```

---

## 6. UI Modernization

### Key Visual Changes

1. **Dark theme by default** — Matches the teleprompter's dark aesthetic. The editor should feel like a production tool, not a 2015 admin panel.
2. **Modern font stack** — DM Sans for UI, JetBrains Mono for readouts
3. **Cleaner button styling** — Flat, minimal buttons with `--accent` amber highlight
4. **Updated iconography** — Replace old 2015 icon sprites with inline SVG or a modern icon set (Phosphor Icons via CDN, or local SVGs)
5. **Toast notifications** — Replace any `alert()` or bare DOM manipulation for feedback with styled toast
6. **Responsive cleanup** — The current responsive handling is Bootstrap 3 era. Bootstrap 5 has better responsive utilities.

### Anti-Patterns to Remove
- Inline `style=""` attributes in HTML (move to CSS files)
- Table-based layouts (replace with flexbox/CSS grid)
- `Courier New` as default font (use the new font stack)
- `bootstrap.min.css` at v3 (update to v5)

---

## 7. Web Worker Considerations

The editor and teleprompter are purely synchronous DOM manipulation + IndexedDB. No NLP or heavy computation that would benefit from a Web Worker. **No Web Workers needed for this project.**

---

## 8. Persistence (IndexedDB via data.manager.js)

Current storage: `data.manager.js` wraps IndexedDB with a simple key-value API (`getItem`, `setItem`, `removeItem`).

Stored data:
- `IFTeleprompterSettings` — user preferences (speed, font size, flip settings, style, etc.)
- `IFTeleprompterSession` — current script HTML + scroll position
- `IFTeleprompterVersion` — migration tracking
- `IFTeleprompterSideBar` — script list for sidebar

**No changes needed to persistence layer.** Data stays local, no backend.

---

## 9. File Size & Performance

The project currently bundles a lot of libraries (`ckeditor/`, `tinymce/`, `fonts/`, `img/`). Many of these are referenced but may not be actively used.

**Before modernizing dependencies:**
- Audit which assets are actually referenced and loaded
- Remove dead CSS/JS (especially old IE fallbacks)
- Audit `ckeditor/` and `tinymce/` — both are loaded in some modes. Only one should be active.

---

## 10. Additional Improvements

### 10.1 Empty `save()` Stub
The `save()` function at line 388 in `editor.js` is an empty stub — dead code that misleads anyone reading the source. Either remove it or fill it with actual save confirmation logic.

### 10.2 Auto-Save Status Indicator
The editor auto-saves on `blur` and `change` events (lines 1410–1420), but gives no visual feedback. Add a small status indicator in the editor corner:
- "Saving..." (muted text, brief)
- "Saved" (green checkmark, persists 2s)
No behavioral change — purely informational.

### 10.3 Font Preloading
`DM Sans` and `JetBrains Mono` are loaded via Google Fonts. First render causes a flash of fallback text. Fix: add `<link rel="preload">` in `<head>` for both fonts to eliminate render-block.

### 10.4 Dead CKEditor Config Cleanup
The toolbar configuration at lines 321–323 references `save` and `spellchecker` plugins that may not be loaded. Dead config that causes confusion. Audit and remove any plugin references that aren't actually bundled.

### 10.5 Keyboard Shortcut Customization
All keyboard shortcuts are currently hardcoded in `teleprompter.js` (`document.onkeydown`). Add a settings panel to remap keys (WASD, arrows, F-keys, Space, ESC). Store custom bindings in `IFTeleprompterSettings`.

### 10.6 Script Export
No way to export a script for use on other teleprompter hardware. At minimum:
- **Copy as plain text** — one-click button, copies editor content without HTML formatting
- **PDF export** — via browser print-to-PDF or a lightweight library like `html2pdf.js`

### 10.7 Remove Unused Editor
Both CKEditor and TinyMCE are bundled in `ckeditor/` and `tinymce/` directories. Only one runs at a time (confirmed by the conditional in `editor.js`). Removing the unused editor saves ~2–3MB of JS. Audit which one is the default and remove the other.

### 10.8 Content Security Policy
No CSP headers are currently set. Add a basic `Content-Security-Policy` meta tag to harden the app against XSS, especially important since users paste content from unknown sources into the editor:
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; script-src 'self' 'unsafe-inline'; img-src 'self' data: https:;">
```

### 10.9 Mobile Remote UX
The QR code remote control setup works but the flow is unintuitive. Streamline:
- Show connection status on the editor screen (connected/disconnected indicator)
- Clearer instructions for first-time setup
- Consider WebSocket-based connection instead of polling (current approach is unclear from code)

### 10.10 Accessibility (ARIA)
No ARIA labels on any interactive elements, no screen reader support. For a single-user broadcast tool this is low priority, but the gap should be documented:
- Add `aria-label` to all toolbar buttons
- Add `role="status"` to the timer display
- Ensure focus indicators are visible (CSS `outline`)

### 10.11 Slider Input Validation
Speed, font size, and acceleration sliders have no `min`/`max` enforcement. Users can set negative speed or extreme font sizes with no guard. Add proper bounds checking in the slider `onchange` handlers.

### 10.12 Synchronous XHR in `credits()`
`internalCredits()` uses `xmlhttp.open("GET", "credits.html", false)` — a **synchronous** XHR that blocks the entire browser thread until the file loads. Replace with an async `fetch()` + promise-based approach.

### 10.13 Unbounded `remoteControls()` Loop
`remoteControls()` calls itself with `setTimeout(remoteControls, 0)` in an infinite loop when `isMobileApp` is true. No cancellation, no backoff — runs forever. Add:
- A cancellation mechanism (e.g., store timeout ID and `clearTimeout` on cleanup)
- A backoff or polling interval instead of immediate recursion

### 10.14 Velocity Formula Opacity
The velocity calculation `Math.pow(Math.abs(x), sensitivity) * (x>=0?1:-1)` with `sensitivity=1.2` produces speeds that aren't easily predictable or tunable. The relationship between slider value and actual pixels-per-second is opaque. Document the formula and expose it in settings so users understand what they're adjusting.

### 10.15 IndexedDB Error Handling
All `dataManager.getItem` and `setItem` calls have no `try/catch` and fail silently on error. Wrap calls in error handling and surface failures to the user (e.g., via the toast notification system).

### 10.16 `var` → `let`/`const` Modernization
The codebase uses `var` throughout (2015 vintage). Modernizing to `let`/`const` would improve readability, prevent accidental hoisting bugs, and make the code easier to reason about. Also addresses several global variable declarations that should be scoped.

### 10.17 Production Build Minification

### 10.18 Backspace Key Conflict in Editor
`document.onkeydown` catches Backspace and calls `resetTimer()`. When the editor is focused and the user is typing, Backspace also deletes text — both actions fire simultaneously. The handler should check whether the editor is focused before triggering the timer reset.

### 10.19 Print Stylesheet
No `print.css` exists. When users print a script via `window.print()`, it outputs raw HTML with all broadcast styling. A dedicated print stylesheet would render clean plain-text output suitable for reading on paper.

### 10.20 Offline Image Handling
Images pasted from the web are stored as `<img src="https://...">`. When offline, these images break. Options:
- **Warn on save:** detect external image URLs and alert the user before saving
- **Download option:** "Download all images" button that caches them locally to IndexedDB or as base64
- **Offline indicator:** show a warning if the app is offline and external images are detected

### 10.21 Anchor Key Conflict with Keyboard Shortcuts
The anchor system accepts any single character as a jump key, but WASD, Space, Arrow keys, and F-keys are already keyboard shortcuts. If a user sets an anchor on `w` or `Space`, the shortcut fires instead of the jump. Add validation to the anchor creation dialog: reject keys already assigned to shortcuts.

### 10.22 Script Version History
The sidebar stores scripts but has no version control. Overwriting a script loses the previous version with no way to revert. For a production tool where scripts are iterated on before events, this is a meaningful gap. Options:
- **Simple:** keep last N versions (e.g., last 5 saves) in IndexedDB
- **Manual snapshots:** "Save version" button with a name, separate from auto-save
- **Diff view:** show what changed between versions (lower priority)
The project ships unminified jQuery, Bootstrap, TinyMCE, and CKEditor. For the Electron build, adding a bundling/minification step (e.g., `esbuild` or `rollup`) would reduce install size. Can be done as part of the dependency update work.

---

## 11. Known Issues

- `save()` function at line 388 in `editor.js` is an empty stub — actual save logic is in the second `save()` at line 1336 (sidebar-related)
- The `wave` button in the toolbar triggers TinyMCE/CKEditor's built-in save, which has no user feedback
- Scroll position is not reset when entering or exiting prompt mode
- Electron 1.8.8 is vulnerable (no longer receiving security patches since ~2020)
- jQuery 1.x + Bootstrap 3 is a known vulnerability surface
