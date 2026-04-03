# Core Web Vitals Optimization: LCP & CLS in Real Projects

Performance scores matter — but most articles explain *what* LCP and CLS are, not *how to actually fix them* in a real codebase.

Here's what I did to improve scores in a Magento PWA and a Next.js app.

---

## LCP (Largest Contentful Paint)

LCP measures how fast the largest visible element loads. Target: **under 2.5s**.

### 1. Preload the Hero Image

The single biggest LCP win. If your hero image is loaded via CSS or a lazy component, the browser discovers it too late.

```html
<link rel="preload" as="image" href="/hero.webp" fetchpriority="high" />
```

In Next.js:
```jsx
<Image src="/hero.webp" priority alt="hero" />
```

The `priority` prop adds `fetchpriority="high"` and a preload link automatically.

### 2. Use WebP / AVIF

Convert all images. WebP is typically 30–50% smaller than JPEG at the same quality.

```bash
# Quick conversion with sharp (Node.js)
npx sharp-cli input.jpg -o output.webp
```

### 3. Eliminate Render-Blocking Resources

Move non-critical CSS to load asynchronously:

```html
<link rel="stylesheet" href="non-critical.css" media="print" onload="this.media='all'" />
```

---

## CLS (Cumulative Layout Shift)

CLS measures unexpected layout movement. Target: **under 0.1**.

### 1. Always Set Width & Height on Images

Without dimensions, the browser doesn't reserve space — the image loads and pushes content down.

```html
<!-- ❌ causes CLS -->
<img src="product.jpg" alt="product" />

<!-- ✅ browser reserves space -->
<img src="product.jpg" width="400" height="300" alt="product" />
```

In Tailwind:
```html
<div class="aspect-[4/3]">
  <img src="product.jpg" class="w-full h-full object-cover" />
</div>
```

### 2. Reserve Space for Dynamic Content

If you load ads, banners, or async content — give them a fixed container:

```jsx
// ❌ banner injects and shifts content
{banner && <Banner />}

// ✅ space is always reserved
<div className="min-h-[90px]">
  {banner && <Banner />}
</div>
```

### 3. Avoid FOUT (Flash of Unstyled Text)

Web fonts cause layout shift when they load. Use `font-display: optional` for body text or preload critical fonts:

```css
@font-face {
  font-family: 'MyFont';
  src: url('/fonts/myfont.woff2') format('woff2');
  font-display: swap; /* or 'optional' to eliminate shift entirely */
}
```

---

## Tools I Use

| Tool | Purpose |
|---|---|
| Lighthouse (DevTools) | Quick local audit |
| PageSpeed Insights | Real-world CrUX data |
| WebPageTest | Waterfall analysis |
| Chrome DevTools Performance tab | Frame-by-frame CLS debugging |

---

## Quick Wins Checklist

- [ ] Preload hero image with `fetchpriority="high"`
- [ ] Convert all images to WebP
- [ ] Set explicit `width` + `height` on every `<img>`
- [ ] Reserve space for async-loaded content
- [ ] Remove unused CSS (PurgeCSS / Tailwind's built-in purge)
- [ ] Defer non-critical JS with `<script defer>`
