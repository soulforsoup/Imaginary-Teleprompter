# Imaginary Teleprompter — Build Plan (Web)

Based on `SPEC.md`. Work top-to-bottom within each phase.

---

## Phase 0 — Audit (Before Any Code Changes)

- [ ] Run the app in browser, verify all features work (editor, save, prompt mode, timer, keyboard controls)
- [ ] Confirm which editor is active: CKEditor or TinyMCE (check `editor.js` for the toggle)
- [ ] Audit all loaded scripts — remove any dead/inactive libraries
- [ ] Check which fonts are actually referenced in CSS vs. bundled in `fonts/`
- [ ] Document current behavior of the save, scroll, and prompt mode flows

---

## Phase 1 — Dependency Updates (Foundation) ✅ DONE

- jQuery was already at v3.2.1 — no update needed
- Bootstrap 3→5 (v5.3.3) updated in css/bootstrap.min.css and js/bootstrap.min.js
- shelljs bumped to 0.8.5
- Commit: ea34968
- KNOWN ISSUE: Bootstrap 5 class migration still pending (done in Phase 2)

### Step 1.1 — jQuery
- [x] Already at v3.2.1 — no update needed

### Step 1.2 — Bootstrap
- [x] Updated `bootstrap.min.css` and `bootstrap.min.js` from 3.x → 5.3.3
- [ ] Bootstrap 5 class migration in HTML files — moved to Phase 2

### Step 1.3 — shelljs
- [x] Updated `package.json` shelljs from `0.8.1` → `0.8.5`

### Step 1.4 — Slider plugin
- [ ] Confirm `bootstrap-slider.min.js` compatibility with Bootstrap 5
- [ ] Replace with native HTML `<input type="range">` if possible — removes a dependency
- [ ] Update slider styling to match Bootstrap 5 form range styles

---

## Phase 2 — UX Fixes

### Step 2.1 — Save Button Toast
- [ ] Add toast CSS to `editor.css` (see SPEC Section 5.1 for styles)
- [ ] Add toast DOM element to `index.html` / `site.html`
- [ ] In `save()` function — append toast → add `.show` class → `setTimeout` 3s → remove
- [ ] Test: click save → toast appears → auto-dismisses after 3s

### Step 2.2 — Prompt Mode Scroll Reset
- [ ] In `submitTeleprompter` / `updatePrompterData` — set `session.scrollPosition = 0` before saving to IndexedDB
- [ ] In `teleprompter.js` `init()` — read `session.scrollPosition` (defaults to 0) and start from that position
- [ ] In `restoreEditor()` — call `animate(0, 0)` to reset position before closing
- [ ] Test: scroll teleprompter → exit → enter again → starts from top

---

## Phase 3 — UI Modernization

### Step 3.1 — Fonts
- [ ] Add Google Fonts `<link>` for `DM Sans` and `JetBrains Mono` to `index.html`, `site.html`
- [ ] Add `<link rel="preload">` for both fonts
- [ ] Update CSS variable declarations in `editor.css` with new font stack
- [ ] Update `.clock` and teleprompter display font to `JetBrains Mono`

### Step 3.2 — Color Variables
- [ ] Define CSS custom properties in `editor.css`:
  ```css
  :root {
    --bg-deep: #0d0d0f;
    --bg-panel: #1a1a1e;
    --bg-input: #232328;
    --accent: #e8a429;
    --accent-hover: #f5b83d;
    --text-primary: #f0f0f0;
    --text-muted: #888890;
    --border: #2e2e34;
    --success: #3ecf6e;
    --danger: #e84a4a;
  }
  ```
- [ ] Audit all hardcoded colors in `editor.css`, `teleprompter.css`, `teleprompter-themes.css`
- [ ] Replace hardcoded colors with CSS variables
- [ ] Apply `--bg-deep` to `body`, `--bg-panel` to sidebar/cards, `--accent` to buttons

### Step 3.3 — Button & Input Styling
- [ ] Update `.btn` base styles in `editor.css` — flat design, `--accent` background
- [ ] Update input/select styles to match dark theme
- [ ] Remove old Bootstrap 3 button shadows and gradients

### Step 3.4 — Icon Sprites → Inline SVGs
- [ ] Identify all icon sprite usage in `index.html` / `site.html`
- [ ] Replace sprite `<use>` references with inline SVGs or a modern icon CDN (Phosphor Icons)
- [ ] Update CSS to style icons consistently

### Step 3.5 — Toast Styles
- [ ] Add `.toast` styles to `editor.css` (slide-in from bottom-right, 3s auto-dismiss)
- [ ] Ensure toast matches dark theme (not default Bootstrap white)

---

## Phase 4 — Improvements

### Step 4.1 — Auto-Save Status Indicator
- [ ] Add a small `<span id="saveStatus">` to the editor toolbar
- [ ] On `blur`/`change` auto-save: show "Saving..." → show "Saved ✓" for 2s
- [ ] Style: muted text, then `--success` green

### Step 4.2 — Font Preloading
- [ ] Add `<link rel="preload">` for `DM Sans` and `JetBrains Mono` in all HTML pages

### Step 4.3 — Dead CKEditor Config Cleanup
- [ ] Audit `editor.js` toolbar config for plugins not in `ckeditor/`
- [ ] Remove references to uninstalled plugins

### Step 4.4 — Slider Bounds Validation
- [ ] Add `min`/`max` attributes to slider inputs in HTML
- [ ] Add JS bounds check in slider `onchange` handlers in `editor.js`

### Step 4.5 — Async Credits
- [ ] Replace synchronous `xmlhttp.open("GET", "credits.html", false)` with `fetch()` + async/await
- [ ] Add loading state while credits load

### Step 4.6 — `remoteControls()` Loop Fix
- [ ] Add a `stopRemoteControls` flag
- [ ] Store `setTimeout` ID and call `clearTimeout` when flag is set
- [ ] Call `stopRemoteControls` on `beforeunload`

### Step 4.7 — Keyboard Shortcut Customization
- [ ] Add a settings section in the sidebar or modal for shortcut remapping
- [ ] Store custom bindings in `IFTeleprompterSettings`
- [ ] Modify `document.onkeydown` to read from settings

### Step 4.8 — Script Export
- [ ] Add "Copy as plain text" button — strip HTML tags, copy to clipboard
- [ ] Add "Export as PDF" — use browser print dialog or `window.print()`
- [ ] Add buttons to editor toolbar

### Step 4.9 — Remove Unused Editor
- [ ] Confirm which editor (CKEditor or TinyMCE) is active
- [ ] Delete the unused editor's directory (`ckeditor/` or `tinymce/`)
- [ ] Verify no broken script references remain in HTML

### Step 4.10 — CSP Meta Tag
- [ ] Add `Content-Security-Policy` meta tag to `index.html`, `site.html`, `teleprompter.html`
- [ ] Test that all functionality works under restrictive CSP

### Step 4.11 — Mobile Remote UX
- [ ] Add connection status indicator (connected/disconnected dot) to editor header
- [ ] Simplify QR code flow with clearer instructions

### Step 4.12 — ARIA Labels
- [ ] Add `aria-label` to all toolbar buttons
- [ ] Add `role="status"` to `.clock` timer element
- [ ] Add `aria-live` to toast notifications

### Step 4.13 — IndexedDB Error Handling
- [ ] Wrap all `dataManager.getItem` / `setItem` calls in `try/catch`
- [ ] On error: show toast with error message

### Step 4.14 — Backspace Key Conflict
- [ ] In `document.onkeydown`: check if editor is focused before calling `resetTimer()` on Backspace
- [ ] Use `document.activeElement` or the `editorFocused` global variable to guard the handler

### Step 4.15 — Print Stylesheet
- [ ] Create `css/print.css`
- [ ] Style: hide toolbar, sidebar, controls; render script as clean black-on-white text
- [ ] Add `<link rel="stylesheet" href="css/print.css" media="print">` to all HTML pages
- [ ] Test: `window.print()` from editor — verify clean output

### Step 4.16 — Offline Image Handling
- [ ] On save: scan content for `<img src="https://...">` URLs
- [ ] If external images found and offline: show warning toast
- [ ] Optionally: add "Download images locally" button that fetches and converts to base64

### Step 4.17 — Anchor Key Validation
- [ ] In anchor creation dialog: show which keys are already shortcuts (WASD, arrows, space, F-keys)
- [ ] Reject those keys with a clear message
- [ ] Update the shortcut list in settings/help if custom shortcuts are added (Step 4.7)

### Step 4.18 — Script Version History
- [ ] Design schema: store `{ scriptId, version, content, savedAt }` in IndexedDB
- [ ] On every save: push new version snapshot (keep last 5)
- [ ] Add "History" button in sidebar: show version list, click to restore
- [ ] Keep it simple: no diff view in v1

---

## Phase 5 — Code Quality

### Step 5.1 — `var` → `let`/`const`
- [ ] Replace all `var` with `let`/`const` in `editor.js` and `teleprompter.js`
- [ ] Scope globals properly (wrap in IIFEs or use module pattern)

### Step 5.2 — Production Build
- [ ] Set up `esbuild` (or similar) for bundling JS assets
- [ ] Minify CSS with `csso` or similar
- [ ] Verify Electron app still works with minified assets

---

## Phase 6 — CKEditor 4 → 5 Migration

**Only attempt after Phase 1–5 are stable. CKEditor 5 is a full rewrite.**

### Step 6.1 — Install CKEditor 5
- [ ] Remove `ckeditor/` directory
- [ ] Add CKEditor 5 via npm or CDN
- [ ] Read CKEditor 5 docs for migration from v4

### Step 6.2 — Toolbar Migration
- [ ] Rewrite toolbar config — v5 uses `toolbar:` configuration, not v4 plugin format
- [ ] Test all toolbar buttons work (save, anchor, undo/redo, formatting)

### Step 6.3 — Plugin Compatibility
- [ ] Check which v4 plugins have v5 equivalents
- [ ] Replace or remove unsupported plugins (anchor/marker system likely needs custom implementation)

### Step 6.4 — Instance Management
- [ ] Replace `CKEDITOR.replace('prompt')` with `ClassicEditor.create()`
- [ ] Update `CKEDITOR.instances.prompt` references throughout
- [ ] Update save callback — v5 uses different event model

---

## Testing Checklist

After every phase:
- [ ] Save script → close → reopen → script restored
- [ ] Enter prompt mode → scroll → exit → re-enter → starts from top
- [ ] Keyboard controls (WASD, arrows, space, F-keys) all work
- [ ] Timer starts/stops/resets correctly
- [ ] Fullscreen on primary display
- [ ] Secondary display fullscreen (if multi-monitor)
- [ ] Mobile/responsive layout on small screens
- [ ] No console errors in DevTools
