# CLAUDE.md — Burros El Godzi

This file provides context for AI assistants working in this repository.

---

## Project Overview

**Burros El Godzi** is a lightweight, single-page web application (SPA) that serves as an online menu and ordering interface for a Mexican street-food restaurant. Customers browse the menu, build a cart, and submit their order directly via WhatsApp.

- **Language**: HTML5 + CSS3 + Vanilla JavaScript — no frameworks, no build tools, no external packages
- **Entry point**: `index.html` is the entire application (~1,100 lines, ~243 KB)
- **Audience/locale**: Spanish-speaking customers; UI, comments, and variable names are all in Spanish

---

## Repository Structure

```
Burros-El-Godzi/
├── index.html   # Complete application (HTML markup + embedded CSS + embedded JS)
└── README.md    # Minimal title placeholder
```

There are no subdirectories, no build output directories, no `node_modules`, and no configuration files.

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
| Constants / variables | `PRODUCTOS` catalog array, `pedido` cart array, DOM element cache |
| Core render functions | `renderMenu()`, `renderCart()` |
| Cart operations | `addToCart()`, `updateQty()`, `removeItem()`, `clearCart()` |
| Persistence | `saveCart()`, `loadCart()` (localStorage) |
| UI helpers | `showToast()`, `openDrawer()`, `closeDrawer()` |
| Order submission | `sendWhatsApp()` |
| Initialization | `loadCart()` call + event listeners wired at bottom |

### CSS structure

CSS uses custom properties defined in `:root` and is organized into comment-delimited blocks matching the HTML sections: reset, body/texture, hero, ticker, section header, product cards, cart, drawer modal, toast, FAB.

---

## Product Data Model

Products are defined in the `PRODUCTOS` array:

```js
{
  id: 'tradicional',           // unique string id
  nombre: 'Tradicional',       // display name
  emoji: '🫓',
  desc: 'Diezmillo al carbón…', // short description
  badge: '⭐ Más pedido',       // optional badge label (omit key or set null for none)
  precios: {
    Chico: 150,                 // price in MXN
    Grande: 200
  }
}
```

**Current menu items** (ID → name):
- `tradicional` — Tradicional ($150/$200)
- `philly` — Clásico Philly ($160/$210)
- `sonorense` — Sonorense ($160/$210)

To add or modify a product, edit only the `PRODUCTOS` array near the top of the `<script>` block. The rest of the UI renders dynamically from this data.

---

## Cart / State Management

- **In-memory state**: `pedido` — array of cart-item objects `{ id, nombre, tamano, precio, cantidad }`
- **Persistence**: localStorage under the key `godzi_pedido` (JSON-serialized array)
- **Hydration**: `loadCart()` is called once on page load; all mutations call `saveCart()` immediately after

---

## WhatsApp Integration

Orders are submitted via a direct WhatsApp deep-link:

```
https://wa.me/526623866834?text=<encodeURIComponent(message)>
```

The message is assembled in `sendWhatsApp()` from the customer name field, itemized cart lines, total, and optional notes.

---

## CSS Design Tokens

```css
:root {
  --orange:      #e8873a;
  --orange-dark: #c96820;
  --orange-warm: #f4a460;
  --dark:        #1a1a1a;
  --cream:       #fdf6ec;
  --radius:      20px;
  --shadow-card: 0 4px 24px rgba(0,0,0,.12);
  --shadow-btn:  0 2px 12px rgba(232,135,58,.4);
}
```

Always use these tokens instead of hardcoded color values.

---

## Coding Conventions

### Language / naming
- **Variable and function names are in Spanish**: `pedido`, `carrito`, `tamano`, `renderCarrito`, `abrirCajón`, etc.
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

### CSS style
- Mobile-first; use `clamp()` for fluid typography and `flexbox` for layouts
- New visual components should follow the existing card/button/toast patterns
- Define new tokens in `:root` if a value is reused more than twice

---

## Development Workflow

### Running the app
No install or build step is needed. Either:
- Open `index.html` directly in a browser (`file://` protocol), **or**
- Serve with any static file server: `python3 -m http.server 8080` then visit `http://localhost:8080`

### Making changes
1. Edit `index.html` directly — there is no source/dist split
2. Reload the browser to see changes
3. Test cart functionality manually: add items, adjust quantities, verify localStorage persistence across page reloads, and confirm the WhatsApp link generates a correctly formatted message

### Testing
There is no automated test suite. Manual browser testing is the only verification method. Key flows to check after any change:
- [ ] Page loads without JS errors
- [ ] All three product cards render with correct prices
- [ ] "Agregar" button adds item to cart; FAB badge updates
- [ ] Quantity +/- buttons work; removing last unit removes the item
- [ ] Cart persists after page reload
- [ ] WhatsApp button opens a correctly formatted message
- [ ] Toast notifications appear and auto-dismiss

---

## Git Conventions

- Commit messages are written in English
- Feature branches follow the pattern `claude/<short-description>` (observed in history)
- There is no PR template or CI pipeline

---

## Key Constraints

- **Do not introduce a build step or bundler** unless explicitly requested — the zero-dependency constraint is intentional
- **Do not add a framework** (React, Vue, etc.) without explicit approval
- **Do not create additional files** (CSS files, JS modules) — keep everything in `index.html`
- **Preserve Spanish naming** for all new variables, functions, and comments in the codebase
- The phone number `526623866834` in `sendWhatsApp()` is production data — do not alter it
