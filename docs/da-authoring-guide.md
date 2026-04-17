# DA Authoring Guide: Multi-Theme Site Setup & Theme Selector

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Site Setup in DA](#site-setup-in-da)
4. [Setting a Default Page Theme](#setting-a-default-page-theme)
5. [Adding the Theme Switcher Block](#adding-the-theme-switcher-block)
6. [Available Themes](#available-themes)
7. [Page-Level vs Site-Level Theming](#page-level-vs-site-level-theming)
8. [Theme Switcher in the Navigation](#theme-switcher-in-the-navigation)
9. [Content Authoring with Themed Blocks](#content-authoring-with-themed-blocks)
10. [Testing Your Theme](#testing-your-theme)
11. [Troubleshooting](#troubleshooting)

---

## Overview

This site supports a **multi-theme system** that allows:

- **Authors** to set a default theme for any page via Page Metadata
- **Visitors** to switch themes at runtime using the Theme Switcher block
- All blocks (Cards, Hero, Columns, etc.) automatically adapt their styles to the active theme

Themes work by swapping a set of **CSS design tokens** (colors, shadows, border radii, etc.) without changing any block markup or JavaScript.

---

## Prerequisites

Before authoring, ensure:

- You have access to the DA (Document Authoring) environment for this project
- The site repository has been set up with the theme files (`styles/themes/`)
- The `theme-switcher` block is available in the codebase (`blocks/theme-switcher/`)

---

## Site Setup in DA

### Step 1: Access Document Authoring

1. Navigate to [https://da.live](https://da.live)
2. Sign in with your Adobe ID
3. Open the organisation and repository for this project (e.g., `skumawat-adobe/multi-theme-demo`)

### Step 2: Understand the Site Structure

Your site in DA will have the following key areas:

```
/ (root)
├── nav                  ← Navigation page (contains header links + optionally Theme Switcher)
├── footer               ← Footer page
├── index                ← Home page
├── en/
│   ├── page-one         ← Content pages
│   └── page-two
└── metadata             ← (optional) Site-wide metadata
```

### Step 3: Verify Block Availability

The following blocks are available for theming:

| Block Name       | DA Table Header  | Purpose                        |
| ---------------- | ---------------- | ------------------------------ |
| `cards`          | `Cards`          | Card grid with themed styles   |
| `hero`           | `Hero`           | Full-width hero with overlay   |
| `columns`        | `Columns`        | Multi-column layout            |
| `theme-switcher` | `Theme Switcher` | Runtime theme selection widget |

---

## Setting a Default Page Theme

Every page can have a default theme set via **Page Metadata**. This tells the site which theme to load when the page is first visited.

### Steps

1. Open any page in DA (e.g., `index`)
2. Scroll to the **bottom** of the document
3. Add a **Metadata** table:

   | Metadata |             |
   | -------- | ----------- |
   | Theme    | theme-ocean |

   In DA block table format:

   ```
   --- Metadata ---
   | Theme | theme-ocean |
   ----------------
   ```

4. Save and publish the page

> **Important:** The theme value must exactly match one of the available theme names (see [Available Themes](#available-themes)). The value is case-insensitive but must include the `theme-` prefix.

### Valid Theme Values for the Metadata field

| Metadata Value | Theme Applied       |
| -------------- | ------------------- |
| `theme-ocean`  | Ocean (cool blues)  |
| `theme-forest` | Forest (greens)     |
| `theme-sunset` | Sunset (warm tones) |

### What Happens Under the Hood

When the page loads:

1. EDS reads the `Theme` metadata → adds `theme-ocean` class to `<body>`
2. `scripts.js` detects the class and dynamically loads `/styles/themes/theme-ocean.css`
3. All blocks on the page pick up the new CSS tokens automatically

---

## Adding the Theme Switcher Block

The **Theme Switcher** block gives site visitors a control to change the theme at runtime. Their choice is saved in `localStorage` and remembered on future visits.

### Option A: Add to a Content Page

1. Open any content page in DA
2. Place your cursor where you want the switcher (e.g., top of page, after the hero)
3. Insert a new block table:

   ```
   | Theme Switcher |
   | ocean          |
   | forest         |
   | sunset         |
   ```

   Each row after the header is one available theme option shown to the visitor.

4. Save and preview

### Option B: Add to the Navigation (Recommended)

Adding the Theme Switcher to the `nav` page means it appears on **every page** of the site.

1. Open the `nav` page in DA
2. At the end of the nav content, add the Theme Switcher block:

   ```
   | Theme Switcher |
   | ocean          |
   | forest         |
   | sunset         |
   ```

3. Save and publish the `nav` page

> **Tip:** Placing it in the nav ensures consistent theme switching across the entire site without needing to author it on every page.

### Theme Switcher Block Structure in Detail

The block table in DA maps to the following:

| Row        | Purpose                                              |
| ---------- | ---------------------------------------------------- |
| Header row | Identifies the block as `Theme Switcher`             |
| Data rows  | Each row = one theme option available to the visitor |

The block will render as a set of buttons (one per theme) in the header area. Clicking a button:

1. Removes any existing `theme-*` class from `<body>`
2. Adds the selected `theme-{name}` class
3. Loads `/styles/themes/theme-{name}.css` if not already loaded
4. Saves the selection to `localStorage` under the key `selected-theme`

---

## Available Themes

### 🌊 theme-ocean

A cool, calming palette inspired by ocean blues.

| Token              | Value                        |
| ------------------ | ---------------------------- |
| Background         | `#e8f4f8` (light blue-white) |
| Text               | `#1a2e3a` (deep navy)        |
| Links              | `#0077b6` (ocean blue)       |
| Link Hover         | `#023e8a` (deep blue)        |
| Light accent       | `#caf0f8` (pale cyan)        |
| Dark accent        | `#03045e` (midnight blue)    |
| Card border        | `#90e0ef`                    |
| Card shadow        | `rgba(0, 119, 182, 0.15)`    |
| Card border radius | `12px`                       |

### 🌲 theme-forest

An earthy, natural palette inspired by forest greens.

| Token              | Value                        |
| ------------------ | ---------------------------- |
| Background         | `#f0f7f0` (pale green-white) |
| Text               | `#1a2e1a` (deep forest)      |
| Links              | `#2d6a4f` (forest green)     |
| Link Hover         | `#1b4332` (dark green)       |
| Light accent       | `#d8f3dc` (mint)             |
| Dark accent        | `#081c15` (near black green) |
| Card border        | `#95d5b2`                    |
| Card shadow        | `rgba(45, 106, 79, 0.15)`    |
| Card border radius | `8px`                        |

### 🌅 theme-sunset

A warm, vibrant palette inspired by sunset tones.

| Token              | Value                       |
| ------------------ | --------------------------- |
| Background         | `#fff8f0` (warm white)      |
| Text               | `#2d1b0e` (deep brown)      |
| Links              | `#e85d04` (burnt orange)    |
| Link Hover         | `#9c2c0a` (dark red-orange) |
| Light accent       | `#fde8d0` (peach)           |
| Dark accent        | `#6a1a00` (deep rust)       |
| Card border        | `#f4a261`                   |
| Card shadow        | `rgba(232, 93, 4, 0.15)`    |
| Card border radius | `4px`                       |

---

## Page-Level vs Site-Level Theming

| Scenario                     | How to Set Up                                                                                 |
| ---------------------------- | --------------------------------------------------------------------------------------------- |
| **One theme for all pages**  | Add Theme Switcher to `nav` page; set `Theme` metadata on the home page only                  |
| **Different theme per page** | Set `Theme` metadata individually on each page                                                |
| **Let users choose freely**  | Add Theme Switcher block to `nav`; omit `Theme` metadata (defaults to no theme / base styles) |
| **Lock a page to one theme** | Set `Theme` metadata on the page; do NOT include the Theme Switcher block on that page        |

---

## Theme Switcher in the Navigation

### Full Nav Page Example

Here is a complete example of a `nav` page in DA with the Theme Switcher included:

```
| Navigation |
| ---------- |
| Logo | [link to homepage] |

---

| Nav Sections |
| About | Products | Contact |

---

| Theme Switcher |
| ocean          |
| forest         |
| sunset         |
```

The Theme Switcher will be rendered by the header block and positioned appropriately in the navigation bar.

---

## Content Authoring with Themed Blocks

All standard EDS blocks automatically respond to the active theme. No special authoring is required — just use blocks as normal.

### Cards Block

```
| Cards |
| [image] | Card Title \n Card description text \n **[Read More](#)** |
| [image] | Card Title \n Card description text \n **[Read More](#)** |
```

With a theme active, cards will show the theme's `--card-border-color`, `--card-bg-color`, `--card-radius`, and `--card-shadow` automatically.

### Hero Block

```
| Hero |
| [background image] | # Page Headline \n Subtitle text \n ***[Get Started](#)*** |
```

The hero overlay color and text color are controlled by `--hero-overlay-color` and `--hero-text-color` from the active theme.

### Columns Block

```
| Columns |
| Column 1 content | Column 2 content | Column 3 content |
```

Column dividers and spacing adapt to the active theme tokens.

---

## Testing Your Theme

### Preview in DA

1. Open your page in DA
2. Click the **Preview** button (top right)
3. The page opens in a new tab — the theme set in metadata is applied automatically
4. To test the Theme Switcher: look for the theme buttons in the header and click them

### Test with URL Parameter (Dev/Staging)

You can force a theme via the URL for testing without changing metadata:

```
https://your-site.aem.page/en/page?theme=theme-forest
```

> Note: URL parameter theme override requires the optional enhancement in `scripts.js` — confirm with your developer if this is enabled.

### Check localStorage

Open browser DevTools → Application → Local Storage → your domain:

- Key: `selected-theme`
- Value: `ocean`, `forest`, or `sunset`

Delete this key to reset to the page's default theme.

---

## Troubleshooting

### Theme is not applying

**Symptoms:** Page loads with default styles, no theme visible.

**Check:**

1. Metadata table spelling — value must be exactly `theme-ocean`, `theme-forest`, or `theme-sunset`
2. Ensure the page has been **saved and published** in DA after adding the metadata
3. Open browser DevTools → check if the `<body>` tag has the theme class (e.g., `class="appear theme-ocean"`)
4. Check Network tab for the theme CSS file request (e.g., `/styles/themes/theme-ocean.css`) — verify it returns 200

### Theme Switcher not appearing

**Symptoms:** No theme buttons visible in the nav/page.

**Check:**

1. Block table header must be exactly `Theme Switcher` (two words, capitalized)
2. Each subsequent row must contain only one cell with a plain theme name (`ocean`, `forest`, `sunset`)
3. Verify the `theme-switcher` block files exist at `blocks/theme-switcher/theme-switcher.js` and `blocks/theme-switcher/theme-switcher.css`
4. Check browser DevTools Console for JavaScript errors

### Theme reverts on page navigation

**Symptoms:** User selects a theme, navigates to another page, theme resets.

**Check:**

1. Ensure `scripts.js` has the `localStorage` restore logic in `loadEager()`
2. The restore code reads `localStorage.getItem('selected-theme')` and applies it before the page renders

### Wrong theme on a specific page

**Symptoms:** Page shows ocean theme but forest was selected.

**Explanation:** Page-level metadata theme takes priority over `localStorage` on initial load. If you want user preference to always win, this requires a code change — raise with your developer.

---

## Quick Reference Card

| Task                    | DA Action                                                 |
| ----------------------- | --------------------------------------------------------- |
| Set page default theme  | Add `Metadata` table with `Theme` = `theme-ocean`         |
| Add user theme switcher | Add `Theme Switcher` block with theme names as rows       |
| Site-wide switcher      | Add `Theme Switcher` block to `nav` page                  |
| Remove theme from page  | Delete the `Theme` row from the page's `Metadata` table   |
| Add a new theme         | Ask developer to create `styles/themes/theme-newname.css` |

---

_Last updated: April 2026 | Multi-Theme Demo Project_
