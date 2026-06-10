# Forms System Prototype — Patient Portal Context

A prototype mobile-first form system for a psychiatric patient intake flow. Demonstrates major input patterns needed for clinical forms, built with vanilla HTML, CSS, and JavaScript — no frameworks or dependencies.

> **Scope note:** This prototype covers part of the *Social History* section of a larger Standard Intake Form. It is not the full intake — just enough to demo every input pattern in the design system.

---

## Architecture

### CSS Layers

Styles are organized using `@layer` for explicit cascade control:

```
@layer config, native, layouts, components, keyframes, utils
```

- **config** — design tokens (primitives → semantic → reset). Two sub-layers: `tokens-primitive` (raw scale: grayscale `---00`–`---100`, blue HSL scale, green/red) and `tokens-semantic` (named aliases using `light-dark()` for automatic dark mode: `--surface`, `--canvas`, `--brand-text`, `--brand-surface`, `--text-body`, `--text-muted`, etc.)
- **native** — base HTML element styles (`body`, `header`, `footer`, `h1`, `a`, etc.)
- **layouts** — structural layout utilities: `cover`, `stack-x`, `stack-y`, `split-x`
- **components** — all UI components: buttons, inputs, form grids, cards, combobox, chips, slider, switch, radio table, file upload, signature, header nav, breadcrumbs, etc.
- **keyframes** — animation definitions (`fade-in`)
- **utils** — one-off utility overrides (`.u-mbs30`, etc.)

### JavaScript (`app.js`)

Shared across all pages via `<script src="app.js" defer>`. Responsibilities:

- **`<progress-bar>` custom element** — shadow DOM web component with animated fill, accepts `value`, `max`, and `label` attributes
- **Form persistence** — saves all named fields to `sessionStorage` under key `socialHistory` on every `input`/`change` event; restores on page load
- **Progress calculation** — `REQUIRED_FIELDS` map defines 25 required fields; `calcProgress()` scores 5–95% (100% only after finish)
- **Validation** — `validate()` checks `aria-required` on inputs, selects, textareas, radio groups, and checkbox groups; shows/hides error messages; scrolls to first error
- **`checkFormComplete()`** — promotes the Next button to `.primary` when all required fields on the current page are filled
- **Auto-advance** — stored in `localStorage`, defaults to on; on single-radio pages, automatically clicks Next after 540ms
- **Menu toggle** — `menu()` toggles `aria-expanded` on the hamburger button with an animated hamburger → X path transition
- **Slider dark mode repaint** — forces a WebKit repaint on `input[type=range]` when the OS color scheme changes, working around a WebKit bug where slider pseudo-element colors don't update without a reflow

### View Transitions

Pages use the CSS View Transitions API for smooth page-to-page navigation:

```css
@media (prefers-reduced-motion: no-preference) {
    @view-transition {
        navigation: auto;
    }
}
```

No JavaScript required — the browser handles cross-document transitions automatically in supported browsers.

---

## Pages & Input Patterns

### `index.html` — Dashboard
Form card grid showing all intake sections with individual progress bars. Demonstrates:
- `<progress-bar>` custom element
- Toggle switch (`input[type=checkbox]`), defaults to on
- Responsive header nav (desktop) / hamburger (mobile)
- User menu button with Lucide chevron

### `1-start.html` — Intro
Landing page for the Social History section. No input patterns — sets context and estimated time.

### `2-demographics.html` — Demographics
Multi-column grid layout (`.form-grid-3`) responsive at 640px. Demonstrates:
- Text input (`input[type=text]`)
- Preferred name field that mirrors first name until manually edited (inline JS)
- Date input (`input[type=date]`)
- Email input (`input[type=email]`)
- Phone input (`input[type=tel]` + `inputmode="tel"`)
- Native select/dropdown (`<select>`)
- ZIP code input (`inputmode="numeric"`)
- Grid layout patterns: `.form-grid-3` (3-col equal), responsive to single column on mobile

### `3-employment.html` — Employment
Single required question. Demonstrates:
- Radio group (`.box.split-x` pattern)
- Auto-advance behavior (single radio group triggers next page after 540ms)

### `4-living.html` — Childhood Experience
Multi-select question. Demonstrates:
- Checkbox group (`.box.split-x` pattern)

### `5-family.html` — Family Members
Matrix question across 8 family member rows × 4 presence options. Demonstrates:
- Radio table (responsive grid → stacked mobile layout)

### `6-history.html` — History & Relationship
Branching logic and multiple question types. Demonstrates:
- Conditional/branching fields (Yes/No radio reveals hidden fields)
- Radio group
- Dynamic `aria-required` toggling on conditional fields

### `7-medications.html` — Medications & Mood
Rich input patterns for clinical data entry. Demonstrates:
- Range slider (`input[type=range]`) with brand-colored thumb, live value display
- Multi-select combobox (medication search with chip/tag management)
- Single-select combobox (pharmacy search, shows full list on focus)

### `8-notes-upload.html` — Notes & Documents
Optional supplemental information. Demonstrates:
- Textarea (multi-line text input)
- File upload (drag-and-drop + tap, multiple files, chip-based file list with truncation)

### `9-signature.html` — Signature & Consent
Legal confirmation before submission. Demonstrates:
- Canvas signature pad (mouse + touch drawing, ink color reads from computed CSS for dark mode support)
- Typed name confirmation (text input)
- Consent checkbox (`.consent-label` inline label pattern)

### `10-confirm.html` — Confirmation
End of flow. Marks the section complete in `sessionStorage` and returns to the dashboard.

---

## Design Tokens

Primitive tokens use a perceptual grayscale scale (`---00` through `---100`) generated with [neuitral.timpish.com](https://neuitral.timpish.com), plus a blue brand scale (`--blue-1` through `--blue-4d`), lime accent and error semantic tokens all stored as hex. The blue scale is split into `--brand-text` (text, focus rings, ghost buttons) and `--brand-surface` (primary button background) to satisfy WCAG 1.4.3 in both light and dark modes simultaneously.

Semantic tokens alias primitives via `light-dark()` exclusively at the semantic layer — never in primitives. This means all dark mode support is handled in one place with no separate dark mode overrides or duplicate rules anywhere in the codebase.

Spacing uses a stepped scale (`--space-00` through `--space-95`) with fluid `clamp()` values for responsive spacing (`--space-30-40`, `--space-40-60`, etc.).

### WCAG Compliance
- 1.4.3 (text contrast): all body text, labels, nav items, and active states pass AA
- 1.4.11 (non-text contrast): input borders (`---55` light / `---40` dark), control borders (`---56` light / `---44` dark), progress track all pass 3:1
- Required asterisks (`*`) use `font-weight: regular` — intentionally lighter than bold label text
- Hover/active states accepted as exempt per WCAG exception for state changes

