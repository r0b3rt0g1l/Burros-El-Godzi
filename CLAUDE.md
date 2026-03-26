# CLAUDE.md — Burros El Godzi

This file provides context for AI assistants working in this repository.

---

## Project Overview

**Burros El Godzi** is a lightweight, single-page web application (SPA) that serves as an online menu and ordering interface for a Mexican street-food restaurant. Customers browse the menu, build a cart, and submit their order directly via WhatsApp.

- **Language**: HTML5 + CSS3 + Vanilla JavaScript — no frameworks, no build tools, no external packages
- **Entry point**: `index.html` is the entire application (~1,111 lines)
- **Audience/locale**: Spanish-speaking customers; UI, comments, and variable names are all in Spanish
- **Deployment**: GitHub Pages via GitHub Actions (auto-deploys on push to `main`)

---

## Repository Structure

```
Burros-El-Godzi/
├── index.html                     # Complete application (HTML markup + embedded CSS + embedded JS)
├── README.md                      # Minimal title placeholder
└── .github/
    └── workflows/
        └── static.yml             # GitHub Actions workflow — deploys to GitHub Pages on push to main
```

There are no subdirectories (beyond `.github`), no build output directories, no `node_modules`, and no configuration files.

---

## Architecture

`index.html` is organized top-to-bottom into three logical sections:

1. **`<head>`** — viewport meta, Google Fonts imports (Bebas Neue, Barlow Condensed, Barlow)
2. **`<style>`** — all CSS, approximately 500 lines with clearly commented sections
3. **`<body>` + `<script>`** — HTML markup and a single IIFE-wrapped JavaScript block

### JavaScript structure

All JavaScript lives inside one IIFE for scope isolation:

```js
(function () {
  'use strict';
  // constants, DOM refs, functions, event listeners
})();
```

The script is organized into comment-delimited sections:

| Section | Responsibility |
|---|---|
| Constants / variables | `PRODUCTOS` catalog array, DOM element cache |
| State | `pedido` — initialized via `loadCart()` |
| Persistence | `saveCart()`, `loadCart()` (localStorage) |
| Toast | `showToast()` |
| Core render | `renderMenu()` — dynamically builds product cards |
| Card init | `initCard()` — attaches per-card size/qty/add listeners |
| Cart operations | `addToCart()`, `updateQty()` |
| Cart render | `renderCart()`, `renderItemList()` — updates inline cart, drawer, and FAB |
| Cart clearing | `clearCart()` |
| Drawer modal | `openDrawer()`, `closeDrawer()` |
| Cart error | `showCartError()` — inline validation message |
| Order submission | `sendWhatsApp()` |
| Initialization | `renderMenu()`, `renderCart()` calls at bottom |

### CSS structure

CSS uses custom properties defined in `:root` and is organized into comment-delimited blocks: reset/variables, body/texture, hero, ticker, section header, product cards, cart, drawer modal, toast, FAB.

The body uses a dark near-black background (`#0a0a0a`) with an SVG fractal-noise grain texture applied as a fixed pseudo-element overlay.

---

## Product Data Model

Products are defined in the `PRODUCTOS` array:

```js
{
  id: 'tradicional',              // unique string id
  nombre: 'Tradicional',          // display name
  emoji: '🌯',
  desc: 'Diezmillo al carbón…',   // short description
  badge: { text: '⭐ Más pedido', clase: 'badge-top' }, // null for no badge
  precios: {
    Chico: 150,                   // price in MXN
    Grande: 200
  }
}
```

The `badge` field is either `null` or an object with `text` (display string) and `clase` (CSS class name). Do not use a bare string — the renderer expects the object shape.

**Current menu items** (ID → name):
- `tradicional` — Tradicional ($150/$200) · badge: `badge-top`
- `philly` — Clásico Philly ($160/$210) · badge: `badge-new`
- `sonorense` — Sonorense ($160/$210) · no badge

To add or modify a product, edit only the `PRODUCTOS` array near the top of the `<script>` block. The rest of the UI renders dynamically from this data.

---

## Cart / State Management

- **In-memory state**: `pedido` — array of cart-item objects `{ producto, tamano, precio, cantidad }`
  - `producto` is the display string (`nombre + ' ' + emoji`, e.g. `'Tradicional 🌯'`)
  - `tamano` is the size string (`'Chico'` or `'Grande'`)
- **Persistence**: localStorage under the key `godzi_pedido` (JSON-serialized array)
- **Hydration**: `pedido` is initialized as `loadCart()` at declaration; all mutations call `saveCart()` immediately after
- **Max quantity**: per-card qty selector caps at 10; cart line caps at 20 total

---

## WhatsApp Integration

Orders are submitted via a direct WhatsApp deep-link:

```
https://wa.me/526623866834?text=<encodeURIComponent(message)>
```

The message is assembled in `sendWhatsApp()` from the customer name field, itemized cart lines, total, and optional notes. The cart is cleared immediately before navigation.

**Instagram iOS fix**: When the user agent contains `Instagram`, the function navigates to the native `whatsapp://send?phone=...&text=...` scheme instead, bypassing Instagram's WKWebView redirect filter.

---

## CSS Design Tokens

```css
:root {
  --orange:      #E05A1A;
  --orange-dark: #B8470E;
  --orange-warm: #F07030;
  --dark:        #110300;
  --dark-card:   #1B0600;
  --dark-deep:   #240900;
  --cream:       #F7EDD8;
  --radius:      20px;
  --shadow-card: 0 20px 60px rgba(0,0,0,0.45), 0 4px 16px rgba(0,0,0,0.3);
  --shadow-btn:  0 8px 24px rgba(224,90,26,0.5);
}
```

Always use these tokens instead of hardcoded color values. Define new tokens in `:root` if a value is reused more than twice.

---

## Coding Conventions

### Language / naming
- **Variable and function names are in Spanish**: `pedido`, `tamano`, `renderMenu`, `addToCart`, `openDrawer`, etc.
- **Comments and section headers are in Spanish**: `/* CARRITO STYLING */`, `// agrega al pedido`
- New code should follow the same Spanish-naming convention

### Formatting
- 2-space indentation throughout
- Single quotes for JS strings
- No trailing semicolons on CSS rules; semicolons used in JS
- Avoid adding linting or formatting tooling — the project intentionally has none

### JS style
- Prefer functional array methods: `.forEach()`, `.map()`, `.filter()`, `.reduce()`, `.join()`
- Update the DOM via direct `innerHTML` assignment (not virtual DOM or templates)
- Keep all JS inside the existing IIFE — do not add `<script>` tags or modules
- `try/catch` is used only around localStorage calls; do not add broad error-swallowing elsewhere
- Use `var` (not `let`/`const`) to match the existing codebase style

### CSS style
- Mobile-first; use `clamp()` for fluid typography and `flexbox` for layouts
- New visual components should follow the existing card/button/toast patterns

---

## Development Workflow

### Running the app
No install or build step is needed. Either:
- Open `index.html` directly in a browser (`file://` protocol), **or**
- Serve with any static file server: `python3 -m http.server 8080` then visit `http://localhost:8080`

### Deployment
Pushing to `main` triggers the GitHub Actions workflow (`.github/workflows/static.yml`) which deploys the entire repository root to GitHub Pages automatically.

### Making changes
1. Edit `index.html` directly — there is no source/dist split
2. Reload the browser to see changes
3. Test cart functionality manually: add items, adjust quantities, verify localStorage persistence across page reloads, and confirm the WhatsApp link generates a correctly formatted message

### Testing
There is no automated test suite. Manual browser testing is the only verification method. Key flows to check after any change:
- [ ] Page loads without JS errors
- [ ] All three product cards render with correct prices and badges
- [ ] Size option toggles correctly; "Agregar" button is disabled until both size and qty are selected
- [ ] "Agregar" button adds item to cart; FAB badge and inline cart update
- [ ] Quantity +/− buttons in cart work; removing last unit removes the item
- [ ] Cart persists after page reload
- [ ] WhatsApp button opens a correctly formatted message
- [ ] Toast notifications appear and auto-dismiss (~2.8s)
- [ ] Cart drawer opens/closes via FAB and overlay click
- [ ] Validation errors show when submitting with empty cart or missing name

---

## Git Conventions

- Commit messages are written in English
- Feature branches follow the pattern `claude/<short-description>` (observed in history)
- There is no PR template; CI is limited to the Pages deployment workflow

---

## Key Constraints

- **Do not introduce a build step or bundler** unless explicitly requested — the zero-dependency constraint is intentional
- **Do not add a framework** (React, Vue, etc.) without explicit approval
- **Do not create additional files** (CSS files, JS modules) — keep everything in `index.html`
- **Preserve Spanish naming** for all new variables, functions, and comments in the codebase
- The phone number `526623866834` in `sendWhatsApp()` is production data — do not alter it
- Use `var` declarations to match the existing ES5-style JS throughout the IIFE
