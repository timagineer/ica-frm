# Forms System Prototype — Patient Portal Context

A mobile-first forms system prototype for a psychiatric patient intake flow. Demonstrates major input patterns needed for clinical forms, built with core web technologies and no frameworks or dependencies.

> **Scope note:** Covers part of the *Social History* section of the Standard Intake Form — not the full intake, but enough to demo every input pattern in the system.

---

## Summary

|                     |                                                                                                                                        |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------|
| **Pages**           | 11 (index + 10 form steps)                                                                                                             |
| **Input patterns**  | 15+ (text, date, email, tel, select, radio, checkbox, range, combobox, textarea, file upload, signature, switch, radio table, consent) |
| **Required fields** | 25                                                                                                                                     |
| **Dark mode**       | Full — via `light-dark()` at semantic token layer only                                                                                 |
| **Accessibility**   | WCAG 2.1 AA — 1.4.3 text contrast, 1.4.11 non-text contrast                                                                            |
| **Dependencies**    | None                                                                                                                                   |

---

## Architecture

### CSS Layers

```
@layer config, native, layouts, components, keyframes, utils
```

- **config** — design tokens (primitives → semantic → reset). Two sub-layers: `tokens-primitive` (color palettes, spacing, typography, etc.) and `tokens-semantic` (named aliases using `light-dark()` for color)
- **native** — base HTML element styles
- **layouts** — `cover`, `stack-x`, `stack-y`, `split-x`
- **components** — all UI components
- **keyframes** — `fade-in`
- **utils** — one-off overrides

`light-dark()` is used exclusively at the semantic layer — never on primitives. All dark mode support lives in one place with no duplicate rules.

### JavaScript (`app.js`)

Shared across all pages via `<script src="app.js" defer>`.

- **`<progress-bar>`** — shadow DOM custom element, animated fill, `value`/`max`/`label` attributes
- **Form persistence** — saves named fields to `sessionStorage('socialHistory')` on every `input`/`change`; restores on load
- **Progress** — `REQUIRED_FIELDS` map (25 fields); `calcProgress()` scores 5–95%
- **Validation** — `validate()` checks `aria-required` across all input types; scrolls to first error
- **`checkFormComplete()`** — promotes Next button to `.primary` when page is complete
- **Auto-advance** — defaults on; single-radio pages advance after 540ms
- **Slider repaint** — forces WebKit reflow on `input[type=range]` on color scheme change (WebKit bug workaround)

### View Transitions

```css
@media (prefers-reduced-motion: no-preference) {
    @view-transition { navigation: auto; }
}
```

Cross-document transitions handled entirely by the browser — no JS required.

---

## Pages & Input Patterns

| Page                                 | Pattern(s)                                                                              |
|--------------------------------------|-----------------------------------------------------------------------------------------|
| `index.html` — Dashboard             | Progress bar web component, auto-advance toggle switch                                  |
| `1-start.html` — Intro               | No inputs — context and time estimate                                                   |
| `2-demographics.html` — Demographics | Text, preferred name (mirrors first), date, email, tel, select, ZIP — 3-col grid layout |
| `3-employment.html` — Employment     | Radio group, auto-advance                                                               |
| `4-living.html` — Childhood          | Checkbox group                                                                          |
| `5-family.html` — Family Members     | Radio table (8 rows × 4 options, responsive)                                            |
| `6-history.html` — History           | Branching/conditional fields, dynamic `aria-required`                                   |
| `7-medications.html` — Medications   | Range slider, multi-select combobox, single-select combobox                             |
| `8-notes-upload.html` — Notes        | Textarea, drag-and-drop file upload with chip list                                      |
| `9-signature.html` — Signature       | Canvas signature pad (dark mode ink), typed name, consent checkbox                      |
| `10-confirm.html` — Confirmation     | Sets `sessionStorage('socialHistoryComplete')`, returns to dashboard                    |

---

## Design Tokens

Primitives are a perceptual 99-step grayscale prototyping palette (`---00`–`---100`) via [neuitral.timpish.com](https://neuitral.timpish.com) plus named brand, accent, and error tokens — all hex. Semantic tokens use `light-dark()` exclusively. The brand scale splits into `--brand-text` (text/focus/ghost) and `--brand-surface` (primary button) to satisfy WCAG 1.4.3 in both modes simultaneously.


### WCAG 2.1 AA
- **1.4.3** — all body text, labels, nav items, active states pass
- **1.4.11** — input borders, control borders, progress track all pass 3:1
- Required `*` markers use `font-weight: regular`
- Hover/active state contrast accepted as exempt per WCAG
