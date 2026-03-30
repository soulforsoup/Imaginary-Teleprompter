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

## Phase 2 — UX Fixes ✅ DONE

- Save button toast added (`js/editor.js`, `css/editor.css`)
- Prompt mode scroll reset implemented
- Bootstrap 5 class migration done in `index.html`
- Commit: `caa0a33`

### Step 2.1 — Save Button Toast
- [x] Toast CSS + `showSaveToast()` function added
- [x] `save()` triggers toast on every save

### Step 2.2 — Prompt Mode Scroll Reset
- [x] `updatePrompterData()` sets `scrollPosition: 0` on save
- [x] `teleprompter.js` reads `session.scrollPosition` on init
- [x] `closeInstance()` resets scroll to top before closing

---

## Phase 3 — UI Modernization ✅ DONE

- Google Fonts (DM Sans + JetBrains Mono) with preload
- Full CSS custom property block in `editor.css`
- Flat buttons with amber accent, inputs with dark styling
- Icon sprites replaced with Phosphor Icons CDN
- Toast now dark-theme compatible
- Commit: `68a6145`

---

## Phase 4 — Improvements ✅ DONE (13/18)
**Commit:** `2468436`

### Done:
- [x] 4.1 Auto-save status indicator
- [x] 4.2 Font preloading (confirmed from Phase 3)
- [x] 4.3 Dead CKEditor/TinyMCE config cleanup
- [x] 4.4 Slider bounds validation (Math.max/min clamping)
- [x] 4.5 Async credits (fetch() + then chain)
- [x] 4.6 remoteControls() loop fix (stop flag + clearTimeout)
- [x] 4.8 Script export (Copy + Print buttons)
- [x] 4.10 CSP meta tag
- [x] 4.12 ARIA labels
- [x] 4.13 IndexedDB error handling
- [x] 4.14 Backspace key conflict fix
- [x] 4.15 Print stylesheet + print button

### Skipped (need design or browser testing):
- [ ] 4.7 Keyboard shortcut customization
- [ ] 4.9 Remove unused editor
- [ ] 4.11 Mobile remote UX
- [ ] 4.16 Offline image handling
- [ ] 4.17 Anchor key validation
- [ ] 4.18 Script version history

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
