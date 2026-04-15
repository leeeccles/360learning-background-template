# 360Learning Homepage Background Image Template

A plug-and-play CSS template for adding a subtle background photo to your 360Learning homepage, with seamless edge blending that works at any screen width.

Built from a live implementation session — April 2026.

---

## What you get

- Gradient background tinted to your platform's brand colour
- Background photo that stays fixed while content scrolls (parallax effect)
- Soft fade at the top (blends into the nav bar), bottom, and right edge
- Works whether the newsfeed panel is open or hidden
- "My Work" section drop shadow for depth

---

## Quick start

### Step 1 — Host your image on GitHub Pages

If you don't already have an image host:

1. Create a new GitHub repo (e.g. `your-username/lms-assets`)
2. Enable GitHub Pages: **Settings → Pages → Source → main branch**
3. Wait ~1 minute for Pages to activate
4. Upload your image to the repo

Your image URL will be:
```
https://your-username.github.io/lms-assets/your-image.jpg
```

**Tip:** You can upload an image without cloning the repo locally:
```bash
gh api repos/your-username/lms-assets/contents/your-image.jpg \
  --method PUT \
  --field message="Add background image" \
  --field content="$(base64 -i your-image.jpg)"
```

### Step 2 — Edit the CSS template

Open `homepage-custom.css` and replace the placeholder:

```css
background-image: url('IMAGE_URL') !important;
```

With your image URL:

```css
background-image: url('https://your-username.github.io/lms-assets/your-image.jpg') !important;
```

### Step 3 — Paste into 360Learning

1. Copy the entire contents of `homepage-custom.css`
2. Go to **Settings → Branding → Group's Custom CSS**
3. Paste and click outside the field to save
4. Hard-refresh: `Cmd+Shift+R` (Mac) or `Ctrl+Shift+R` (Windows)

> CSS must be applied at the **top-level (platform) group** — subgroup CSS won't reach the homepage.

---

## Tuning the image

All tuning values are in `homepage-custom.css`. Here's what each one does:

### Position
```css
background-position: left 65% !important;
```
- `left` = anchors the image to the left edge of the screen
- `65%` = vertical position (higher % = lower in the image)
- For photos with people, start around `65–75%` so heads aren't cropped

### Size
```css
background-size: 75% auto !important;
```
- Image fills 75% of viewport width; height scales proportionally
- Increase if you want it to fill more of the screen
- The right-side mask handles the edge where it ends

### Opacity
```css
opacity: 0.10 !important;
```
- Range: `0.05` (barely there) → `0.15` (clearly visible)
- `0.08–0.10` is the sweet spot for most photos

### Top fade (nav bar blend)
```css
linear-gradient(to bottom, transparent 0%, black 28%, ...)
```
- `28%` = how far down before the image is fully visible
- Increase (e.g. `35%`) for a softer, longer blend into the nav bar
- Decrease (e.g. `15%`) if the top of the photo is being hidden too much

### Right fade (wide screen / no newsfeed)
```css
linear-gradient(to right, black 30%, transparent 75%)
```
- `30%` = where the fade begins
- `75%` = where the image is fully transparent
- If you see a hard edge on wide screens, lower the first value (e.g. `20%`)
- If the fade is cutting into the subject too early, raise both values

---

## How it works (technical notes)

### Why `::before` and not `body::before`
`body` has `overflow: hidden` set by 360Learning — any `::before` pseudo-element on it gets clipped. `.__view::before` sits inside the scroll container and renders correctly.

### Why `background-attachment: fixed`
This anchors the image to the viewport rather than the element, so it stays still while the page content scrolls — creating a depth effect. The side effect is that the image is sized relative to the viewport width, which is why a right-side mask is needed for wide screens.

### Why two mask layers with `mask-composite: intersect`
CSS supports multiple `mask-image` layers, but by default they're additive. `intersect` (or `source-in` in WebKit) means the final result is only visible where **both** masks are opaque — giving clean simultaneous top/bottom and right-side fading.

### Why `!important` everywhere
360Learning's platform CSS uses `!important` extensively. Without it, custom rules are silently overridden.

---

## Responsive behaviour

| Scenario | Behaviour |
|---|---|
| Standard view (with newsfeed) | Image fills left ~75% of viewport, fades right |
| Wide view (no newsfeed) | Same — right fade hides the gap cleanly |
| Narrow screens | Image may compress; adjust `background-size` up |

---

## Things that don't work (save yourself the debugging)

| Attempt | Why it fails |
|---|---|
| `body { background: ... }` | `body` has `overflow: hidden` — clips everything |
| `body::before` for photo layer | Clipped by body overflow |
| `position: fixed` inside `.__panel` | Panel has `overflow: hidden scroll` — fixed elements clip to panel |
| `background-size: cover` with a fixed image | Works, but no control over which part of the image shows — use `X% auto` + a right mask instead |
| Targeting `[data-v-xxxxxxxx]` attributes | These change between 360Learning releases |
| `.main-page-layout`, `.group-home` | Not present in the rendered Vue.js DOM |

---

## Debugging checklist

1. **CSS not loading?** DevTools → Elements → search for `id="customCSS"` — your rules should be inside a `<style>` tag
2. **Changes not showing?** Always hard-refresh: `Cmd+Shift+R` / `Ctrl+Shift+R`
3. **Nuclear test:** Temporarily add `* { background: red !important; }` — if nothing turns red, CSS isn't saving
4. **Image not loading?** Check the GitHub Pages URL directly in your browser first — it takes ~1 min to activate after enabling Pages
5. **Hard edge on the right?** Lower the first value in the right mask gradient (e.g. `black 20%`)
6. **Head cut off?** Increase the vertical position value (e.g. `70%` → `80%`)
7. **Image too faint/strong?** Adjust `opacity` in 0.01 increments

---

*Built from a live 360Learning implementation session — April 2026.*
