# Multi-Theme System Plan for EDS with DA Authoring

## Current Architecture Analysis

The EDS project uses:

- `styles/styles.css` — global CSS custom properties (design tokens) on `:root`
- `decorateTemplateAndTheme()` in `aem.js` — reads the `theme` metadata tag and adds it as a class on `<body>`
- Blocks (cards, hero, columns, etc.) use CSS variables like `var(--background-color)`, `var(--text-color)`, etc.
- DA authoring sets page-level metadata like `theme: my-theme` which gets applied as `<body class="my-theme">`

This means **the theme system hook already exists** — we just need to build on it properly.

---

## Architecture Plan

### 1. Theme CSS Files (`styles/themes/`)

Create one CSS file per theme. Each theme file **overrides the CSS custom properties** defined in `styles.css` using a body class selector:

```
styles/
  themes/
    theme-ocean.css      ← overrides :root tokens under body.theme-ocean
    theme-forest.css     ← overrides :root tokens under body.theme-forest
    theme-sunset.css     ← overrides :root tokens under body.theme-sunset
```

**Pattern inside each theme file:**

```css
/* styles/themes/theme-ocean.css */
body.theme-ocean {
  --background-color: #e8f4f8;
  --text-color: #1a2e3a;
  --link-color: #0077b6;
  --link-hover-color: #023e8a;
  --light-color: #caf0f8;
  --dark-color: #03045e;

  /* block-level tokens (e.g. for cards) */
  --card-border-color: #90e0ef;
  --card-bg-color: #ffffff;
  --card-radius: 12px;
  --card-shadow: 0 4px 12px rgba(0, 119, 182, 0.15);

  /* hero tokens */
  --hero-text-color: #ffffff;
  --hero-overlay-color: rgba(0, 77, 113, 0.4);
}
```

### 2. Tokenize Block CSS

Convert hardcoded values in block CSS to reference design tokens. For example, `cards.css`:

```css
/* Before */
.cards > ul > li {
  border: 1px solid #dadada;
  background-color: var(--background-color);
}

/* After (tokenized) */
.cards > ul > li {
  border: 1px solid var(--card-border-color, #dadada);
  background-color: var(--card-bg-color, var(--background-color));
  border-radius: var(--card-radius, 0);
  box-shadow: var(--card-shadow, none);
}
```

The **fallback value** pattern `var(--token, fallback)` ensures blocks look fine with no theme applied (uses default styling).

### 3. Theme Switcher Block (`blocks/theme-switcher/`)

A minimal block that:

- Reads available themes from its authored content (DA authors list themes in the block)
- Renders a dropdown/button group UI
- On selection: adds the theme class to `<body>` + persists to `localStorage` + dynamically loads the theme CSS file via `loadCSS()`

```
blocks/
  theme-switcher/
    theme-switcher.js   ← renders UI, handles switching, loads CSS
    theme-switcher.css  ← minimal styling for the switcher widget
```

**Switcher JS logic:**

```js
// On theme select:
async function applyTheme(themeName) {
  // Remove old theme classes
  document.body.className = document.body.className
    .replace(/theme-\S+/g, "")
    .trim();
  // Add new theme class
  document.body.classList.add(`theme-${themeName}`);
  // Load the theme CSS
  await loadCSS(`/styles/themes/theme-${themeName}.css`);
  // Persist
  localStorage.setItem("selected-theme", themeName);
}
```

### 4. DA Authoring Integration

**Two ways themes work with DA:**

**a) Page-level default theme (via Page Metadata):**

In DA, authors set the `Theme` metadata field to e.g. `theme-ocean`. `decorateTemplateAndTheme()` already reads this and adds `theme-ocean` to `<body>`. We extend `scripts.js` to also **load the corresponding CSS file** when this class is detected.

```js
// In loadEager(), after decorateTemplateAndTheme():
const themeClass = [...document.body.classList].find((c) =>
  c.startsWith("theme-"),
);
if (themeClass) {
  loadCSS(`${window.hlx.codeBasePath}/styles/themes/${themeClass}.css`);
}
```

**b) User-selectable theme (via Theme Switcher block):**

Authors place the `Theme Switcher` block anywhere on the page (header/nav area recommended). They list available themes in the block's table rows in DA. The block renders a UI control for runtime switching.

### 5. Minimal Demo Blocks to Create/Update

| Block            | What Changes                                                       |
| ---------------- | ------------------------------------------------------------------ |
| `cards`          | Tokenize `border-color`, `bg-color`, `border-radius`, `box-shadow` |
| `hero`           | Tokenize text color, overlay, gradient                             |
| `columns`        | Tokenize divider color, spacing                                    |
| `theme-switcher` | **New block** — theme selection UI                                 |

---

## File Structure After Implementation

```
styles/
  styles.css              ← base tokens (unchanged structure)
  themes/
    theme-ocean.css        ← Ocean: blues, cool tones
    theme-forest.css       ← Forest: greens, earthy tones
    theme-sunset.css       ← Sunset: warm oranges/purples

blocks/
  theme-switcher/
    theme-switcher.js
    theme-switcher.css
  cards/
    cards.css              ← tokenized (updated)
    cards.js               ← unchanged
  hero/
    hero.css               ← tokenized (updated)
    hero.js                ← unchanged

scripts/
  scripts.js               ← extended to load theme CSS on page load
```

---

## DA Authoring Workflow for Authors

1. Author opens a page in DA
2. In **Page Metadata**, set `Theme` = `theme-ocean` (sets default page theme)
3. Optionally, add a **Theme Switcher** block anywhere on the page listing:
   ```
   | Theme Switcher |
   | ocean          |
   | forest         |
   | sunset         |
   ```
4. Visitors can switch themes; choice persists in `localStorage`

---

## Implementation Steps

1. **Create 3 theme CSS files** (`theme-ocean`, `theme-forest`, `theme-sunset`) with overriding CSS tokens
2. **Tokenize existing block CSS** (`cards.css`, `hero.css`, `columns.css`) to use theme tokens with fallbacks
3. **Create `theme-switcher` block** (JS + CSS) that reads authored theme list and renders a switcher UI
4. **Extend `scripts.js`** to auto-load the theme CSS file when a theme class is present on `<body>`
5. **Verify** the fallback chain: default tokens → theme override tokens → block styles

---

## Key Design Principles

- **EDS-native**: No external dependencies; follows EDS conventions throughout
- **Progressive enhancement**: Fallback values ensure blocks render correctly with no theme applied
- **DA-first authoring**: Theme selection is controlled via page metadata — no code changes needed for new pages
- **Token cascade**: `styles.css` defines base tokens → theme CSS overrides tokens on `body.theme-X` → block CSS consumes tokens
- **Performance**: Theme CSS is loaded dynamically only when needed; no unused CSS blocking render
- **Persistence**: User's theme choice is saved to `localStorage` and restored on next visit
