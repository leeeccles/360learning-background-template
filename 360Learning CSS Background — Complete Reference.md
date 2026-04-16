# 360Learning CSS Background — Complete Reference

A complete reference for building and maintaining homepage backgrounds in 360Learning custom CSS. Covers everything from DOM structure and architecture constraints to advanced image composition and edge-blending techniques.

*Built from live implementation sessions — April 2026.*

---

## Contents

1. [How 360Learning CSS Works](#1-how-360learning-css-works)
2. [The DOM Structure](#2-the-dom-structure)
3. [Critical Architecture Constraints](#3-critical-architecture-constraints)
4. [Key Selectors](#4-key-selectors)
5. [Platform CSS Variables](#5-platform-css-variables)
6. [Scoping to the Homepage Only](#6-scoping-to-the-homepage-only)
7. [Pattern: Gradient Background](#7-pattern-gradient-background)
8. [Pattern: Blueprint Grid Background](#8-pattern-blueprint-grid-background)
9. [Pattern: Background Image](#9-pattern-background-image)
10. [Pattern: My Work Drop Shadow](#10-pattern-my-work-drop-shadow)
11. [The Nav Bar Shadow Problem](#11-the-nav-bar-shadow-problem)
12. [Things That Don't Work](#12-things-that-dont-work)
13. [Debugging Checklist](#13-debugging-checklist)
14. [Minimal Working CSS (Clean Starting Point)](#14-minimal-working-css-clean-starting-point)

---

## 1. How 360Learning CSS Works

- CSS is added in **Settings → Branding → Group's Custom CSS**
- Click **outside the field** to save — there is no save button
- Always **hard-refresh** after saving: `Cmd+Shift+R` (Mac) / `Ctrl+Shift+R` (Windows)
- In Chrome DevTools → Network tab → tick **Disable cache** for reliable testing
- CSS applies to the platform interface and the login page
- Changes must be tested after every 360Learning release (~every 3 weeks)
- 360Learning does not support custom CSS — you own all maintenance
- CSS must be applied at the **top-level (platform) group** — subgroup CSS won't reach the homepage

**To confirm CSS is loading:** DevTools → Elements → search for `customCSS` — your rules appear inside a `<style id="customCSS">` tag.

---

## 2. The DOM Structure

This is the actual rendered structure of the 360Learning homepage. Understanding this is essential before writing any CSS.

```
<body class="new-font">
  <div id="vuejs" data-v-app>

    <div class="navigation-bar-container">    ← Pill tab nav (Overview, Users, etc.)
      <div class="workspace-navigation-bar">
        <div class="chips-carousel">          ← Swiper carousel
          <ul class="swiper-wrapper">...</ul>
          ::before                            ← Left fade mask (white by default)
          ::after                             ← Right fade mask (white by default)
          <div class="carousel-arrow left-arrow">
          <div class="carousel-arrow right-arrow">
        </div>
      </div>
    </div>

    <div class="__vuescroll">
      <div class="__panel">                   ← SCROLL CONTAINER (overflow: hidden scroll)
        <div class="__view">                  ← CONTENT AREA — target this for backgrounds
          <div class="scrollable-container">
            <div class="content-container">

              <!-- Hero widget (HTML widget iframe) -->
              <!-- Homepage widgets -->

              <div id="my-work-section">      ← MY WORK — unique to homepage, use for scoping
              </div>

            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</body>
```

**Key insight:** `.navigation-bar-container` and `.__panel` are **siblings** — they sit at the same level. CSS applied to `.__view` (inside `.__panel`) cannot affect the nav bar, and vice versa.

---

## 3. Critical Architecture Constraints

These caused the most debugging time. Know them before you start.

| Element | Constraint | Impact |
|---|---|---|
| `body` | Has `overflow: hidden` set by 360Learning | `body::before` photo layers get clipped |
| `.__panel` | Has `overflow: hidden scroll` | `position: fixed` inside it is clipped to panel bounds |
| `.__panel` mask | Fades both content AND background | Creates white line under nav bar — avoid |
| Nav bar shadow | Added dynamically by JS on scroll | Not present on page load — must use `!important` to suppress |
| Swiper fades | `::before`/`::after` default to white | Turn white box if nav bg is changed without updating them |
| `#my-work-section` | Only exists on the homepage | Safe scoping selector — never matches course or group pages |
| `data-v-xxxxxxxx` attributes | Can change between releases | Avoid using these as selectors |
| `background-attachment: fixed` | Image is sized/positioned relative to **viewport**, not element | On wider screens the image may not reach the right edge — needs a right-side mask |

---

## 4. Key Selectors

### Selectors that work (confirmed)

| Element | Selector |
|---|---|
| Homepage content area | `.__view:has(#my-work-section)` |
| Homepage scroll container | `.__panel:has(#my-work-section)` |
| Nav bar container | `.navigation-bar-container` |
| Nav bar inner | `.workspace-navigation-bar` |
| My work section | `#my-work-section` |
| Swiper left fade | `.chips-carousel::before` |
| Swiper right fade | `.chips-carousel::after` |
| Scroll arrow buttons | `.carousel-arrow` |
| Show more button hover | `.clickable-text.neutral:hover` |
| Homepage body scoping | `body.new-font:has(#my-work-section)` |
| Group avatar type icon | `.group-pic-container .type-icon` |
| Top-left logo | `.section.logo-section.workspace-navigation-logo` |

### Selectors that do NOT work (not present in rendered DOM)

- `.main-page-layout`
- `.group-home`
- `[class*="HomePage"]`
- `[class*="overview"]`

---

## 5. Platform CSS Variables

Set by 360Learning — respond to branding colour automatically:

```css
--primary-100    /* lightest tint */
--primary-600    /* your main brand colour */
--primary-800    /* darkest */

--platform-background    /* usually white */
--menu-background        /* left sidebar colour */
--my-work-background     /* beige/cream default */
```

`color-mix()` is the key to dynamic colours:

```css
color-mix(in srgb, var(--primary-600) 3%, white)
/* 2% = barely visible tint, 50% = halfway to brand colour */
```

---

## 6. Scoping to the Homepage Only

Always scope homepage changes using `:has(#my-work-section)`:

```css
/* ✅ Homepage only */
body.new-font:has(#my-work-section) .some-element { ... }
.__view:has(#my-work-section) { ... }

/* ❌ Applies everywhere — avoid */
.__view { ... }
```

---

## 7. Pattern: Gradient Background

A soft brand-colour tint — lighter at the top, gently deeper toward the bottom. Works well as a standalone background or as the base layer behind a photo.

```css
.__view:has(#my-work-section) {
  position: relative !important;
  background: linear-gradient(
    to bottom,
    color-mix(in srgb, var(--primary-600) 2%, white) 0%,
    color-mix(in srgb, var(--primary-600) 6%, white) 100%
  ) !important;
  min-height: 100vh !important;
}
```

Adjust the percentages inside `color-mix()` to control intensity. `2%/6%` is very subtle; `5%/15%` is clearly visible.

---

## 8. Pattern: Blueprint Grid Background

Four layered gradients create a large 100px grid with a finer 20px subdivision, tinted in the platform brand colour.

```css
:root {
  --bp-base:      color-mix(in srgb, var(--primary-600) 2%, white);
  --bp-line-soft: color-mix(in srgb, var(--primary-600) 3%, transparent);
  --bp-line-hard: color-mix(in srgb, var(--primary-600) 6%, transparent);
}

.__view:has(#my-work-section) {
  background:
    linear-gradient(var(--bp-line-hard) 1px, transparent 1px),
    linear-gradient(90deg, var(--bp-line-hard) 1px, transparent 1px),
    linear-gradient(var(--bp-line-soft) 1px, transparent 1px),
    linear-gradient(90deg, var(--bp-line-soft) 1px, transparent 1px),
    var(--bp-base) !important;
  background-size: 100px 100px, 100px 100px, 20px 20px, 20px 20px !important;
  min-height: 100vh !important;
}
```

**If matching the nav bar** — apply the same background to `.navigation-bar-container` and suppress its scroll shadow. You must also update the swiper fades or they appear as white boxes:

```css
body.new-font:has(#my-work-section) .navigation-bar-container {
  background: /* same as above */ !important;
  background-size: 100px 100px, 100px 100px, 20px 20px, 20px 20px !important;
  box-shadow: none !important;
  border: none !important;
}

body.new-font:has(#my-work-section) .navigation-bar-container .chips-carousel::before {
  background: linear-gradient(to right, var(--bp-base), transparent) !important;
}
body.new-font:has(#my-work-section) .navigation-bar-container .chips-carousel::after {
  background: linear-gradient(to left, var(--bp-base), transparent) !important;
}
```

---

## 9. Pattern: Background Image

This is the most nuanced pattern. Read all notes carefully before tuning.

### Why `.__view::before` and not `body::before`

`body` has `overflow: hidden` — any `::before` on it gets clipped. `.__view::before` sits inside the scroll container and renders correctly.

### Why `background-attachment: fixed`

Anchors the image to the viewport rather than the element. The image stays still while content scrolls, creating a depth/parallax effect. **Important side effect:** image size and position are calculated relative to the **viewport**, not the element — see the viewport width issue below.

### The viewport width issue

With `background-attachment: fixed`, if you use `background-size: 75% auto`, the image fills 75% of the viewport width. On a standard view (with the newsfeed panel), this fills the content area nicely. But on a wider view — large monitor, newsfeed hidden, or full-screen — the right 25% of the screen has no image, leaving a visible edge.

**Solution:** Don't try to use `cover` to fix this (see below). Instead, keep the percentage size and add a right-side mask gradient that fades the image out before it ends. This makes the edge invisible regardless of screen width.

### Why `background-size: 75% auto` is better than `cover`

`cover` scales the image to fill the entire element — it always looks seamless at the edges, but you lose control over *which part* of the image is visible. With a group photo, important subjects (faces, composition) may be cropped unpredictably as the viewport resizes. Using `75% auto` + a right-side mask gives you full compositional control: you choose exactly what's shown, and the mask handles the edge gracefully.

### Hosting images on GitHub Pages

1. Create a GitHub repo (e.g. `your-username/lms-assets`)
2. Enable Pages: **Settings → Pages → Source → main branch**
3. Wait ~1 minute for Pages to activate
4. Upload your image

Image URL format:
```
https://your-username.github.io/lms-assets/your-image.jpg
```

**Uploading without a local clone** (uses the GitHub CLI):
```bash
gh api repos/your-username/lms-assets/contents/your-image.jpg \
  --method PUT \
  --field message="Add background image" \
  --field content="$(base64 -i your-image.jpg)"
```

### The multi-layer mask

The mask has two independent gradients combined using `mask-composite: intersect`:

1. **Top-to-bottom** — fades the image in below the nav bar and out toward the bottom of the page
2. **Right-side** — fades the image out before it ends, preventing any visible edge on wide screens

`mask-composite: intersect` means the image is only visible where *both* masks are opaque simultaneously. `-webkit-mask-composite: source-in` is the Safari/Chrome equivalent — always include both.

### The full pattern

```css
.__view:has(#my-work-section)::before {
  content: '' !important;
  position: absolute !important;
  top: 0 !important;
  left: 0 !important;
  right: 0 !important;
  height: 100% !important;
  background-image: url('https://your-username.github.io/lms-assets/your-image.jpg') !important;
  background-size: 75% auto !important;
  background-position: left 65% !important;
  background-repeat: no-repeat !important;
  background-attachment: fixed !important;
  opacity: 0.10 !important;
  filter: grayscale(15%) !important;
  mask-image:
    linear-gradient(to bottom, transparent 0%, black 28%, black 60%, transparent 100%),
    linear-gradient(to right, black 30%, transparent 75%) !important;
  -webkit-mask-image:
    linear-gradient(to bottom, transparent 0%, black 28%, black 60%, transparent 100%),
    linear-gradient(to right, black 30%, transparent 75%) !important;
  mask-composite: intersect !important;
  -webkit-mask-composite: source-in !important;
  z-index: 0 !important;
  pointer-events: none !important;
}

/* Keep all content above the photo layer */
.__view:has(#my-work-section) > * {
  position: relative !important;
  z-index: 1 !important;
}
```

### Tuning reference

**`background-position: X% Y%`**

| Value | Effect |
|---|---|
| `left` or `0%` | Image anchored to left edge of viewport |
| `50%` (center) | Image centred — may drift off-screen on wide viewports |
| Y% low (e.g. `30%`) | Shows upper portion of image — heads may be cut off |
| Y% high (e.g. `70%`) | Shows lower portion — good for tall group photos |

Start with `left 65%` for a portrait or group photo. Tune the vertical value until the subject's head is fully visible.

**`background-size`**

| Value | Effect |
|---|---|
| `75% auto` | Image fills 75% of viewport width; height scales proportionally |
| `85% auto` | Fills more of the screen before the right mask kicks in |
| `cover` | Always fills the whole element — no control over which part shows |

**`opacity`**

| Value | Character |
|---|---|
| `0.05` | Barely there — texture only |
| `0.08–0.10` | Subtle but visible — recommended range |
| `0.15` | Clearly visible, strong presence |

**Top mask** `linear-gradient(to bottom, transparent 0%, black XX%, ...)`

- `XX%` = how far down before the image is fully visible
- `28%` = standard — soft blend into the nav bar
- Increase to `35–40%` for a longer, softer transition
- Decrease to `15%` if too much of the top of the photo is being hidden

**Right mask** `linear-gradient(to right, black XX%, transparent YY%)`

- `XX%` = where the fade begins (lower = starts fading sooner)
- `YY%` = where the image is fully gone
- Keep ~20–25 percentage points of gap between the two for a soft fade
- If you still see a hard edge: lower `XX%` (e.g. `20%`)
- If the fade cuts into the subject: raise both values (e.g. `45%, 80%`)

---

## 10. Pattern: My Work Drop Shadow

Adds a subtle lifted-card shadow to the My Work section using the brand colour.

```css
#my-work-section {
  border-radius: 12px !important;
  box-shadow:
    0 2px 8px color-mix(in srgb, var(--primary-600) 8%, transparent),
    0 8px 24px color-mix(in srgb, var(--primary-600) 5%, transparent) !important;
}
```

---

## 11. The Nav Bar Shadow Problem

360Learning adds a `box-shadow` to `.navigation-bar-container` dynamically via JavaScript when the page scrolls. It is not present on page load, so you must suppress it with `!important`:

```css
body.new-font:has(#my-work-section) .navigation-bar-container {
  box-shadow: none !important;
}
```

---

## 12. Things That Don't Work

| Attempt | Why it fails |
|---|---|
| `body { background: ... }` | `body` has `overflow: hidden` — visible area clips |
| `body::before` for photo layer | Clipped by body overflow |
| `position: fixed` inside `.__panel` | `.__panel` has `overflow: hidden scroll` — fixed elements clip to panel bounds |
| `body::after` fixed overlay for scroll fade | Outside scroll context — doesn't track scroll position |
| `mask-image` on `.__panel` | Fades both content and background — creates white line under nav |
| Sticky `::before` inside `.scrollable-container` | Clipped by parent overflow, unreliable |
| `.main-page-layout`, `.group-home` | Not present in rendered Vue.js DOM |
| `background-size: cover` for positioned photos | Always fills the container, but gives no control over which part of the image is visible — use `X% auto` + right mask instead |
| `background-position` animation on `border-box` gradient | Doesn't animate reliably |
| Targeting `[data-v-xxxxxxxx]` attributes | Change between 360Learning releases |
| `--my-work-background: var(--bp-base)` | Causes white line artefact at section boundary |
| `@property` + `conic-gradient` on Create button | Causes flashing in 360Learning's CSS environment |
| Overriding nav background without updating swiper fades | White box appears at pill overflow edge |
| Overriding swiper fades without matching nav background | Tinted box appears at pill overflow edge |

---

## 13. Debugging Checklist

1. **Confirm CSS is saving:** DevTools → Elements → search for `id="customCSS"` — your rules should be visible inside the `<style>` tag
2. **Disable cache:** DevTools → Network tab → tick Disable cache → refresh
3. **Nuclear test:** Add `* { background: red !important; }` temporarily — if nothing turns red, CSS isn't saving
4. **Find a class name:** DevTools element picker (`Cmd+Shift+C`) → click element → read `class` attribute
5. **Check overrides:** DevTools Styles panel → filter by `background` to see all rules on a selected element
6. **Confirm correct page:** Homepage URL ends in `/home/content/all`
7. **Check group level:** CSS must be applied at the platform (top-level) group — subgroup CSS won't reach the homepage
8. **Saving behaviour:** Paste CSS → click outside the field → navigate away and back to confirm it persisted
9. **Image not loading:** Open the GitHub Pages URL directly in the browser to verify it's live — Pages takes ~1 minute to activate after enabling
10. **Hard edge on right:** Lower the first value in the right mask gradient (e.g. `black 20%`)
11. **Head cut off at top:** Increase the vertical position value (e.g. `65%` → `75%`)
12. **Image too faint:** Raise `opacity` in 0.01 increments; if still not visible, check the image URL is correct

---

## 14. Minimal Working CSS (Clean Starting Point)

Copy this and build from here. Includes UI tidying, gradient background, and the shadow lift. Add the background image block from Section 9 when ready.

```css
/* =================== UI TIDYING =================== */

.section.logo-section.workspace-navigation-logo {
  display: none !important;
}
.nested-section.home-section {
  margin-top: 8px !important;
}
.group-pic-container .type-icon {
  display: none !important;
}


/* =================== HOMEPAGE: GRADIENT BACKGROUND =================== */

.__view:has(#my-work-section) {
  position: relative !important;
  background: linear-gradient(
    to bottom,
    color-mix(in srgb, var(--primary-600) 2%, white) 0%,
    color-mix(in srgb, var(--primary-600) 6%, white) 100%
  ) !important;
  min-height: 100vh !important;
}


/* =================== MY WORK: SHADOW LIFT =================== */

#my-work-section {
  border-radius: 12px !important;
  box-shadow:
    0 2px 8px color-mix(in srgb, var(--primary-600) 8%, transparent),
    0 8px 24px color-mix(in srgb, var(--primary-600) 5%, transparent) !important;
}
```

---

*For the background image template (CSS file + setup guide), see: [github.com/leeeccles/360learning-background-template](https://github.com/leeeccles/360learning-background-template)*
