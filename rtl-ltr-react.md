
Most i18n tutorials stop at translating text. But if you're building for Arabic audiences, switching the language is only half the job — the *layout* has to flip too.

This guide covers everything I learned building bilingual (AR/EN) apps with proper RTL/LTR support.

---

## Why `dir="rtl"` Alone Is Not Enough

Setting `<html dir="rtl">` switches text alignment and some browser defaults, but it doesn't handle:

- Flexbox / Grid direction
- Margin and padding (e.g. `ml-4` becomes wrong in RTL)
- Positioned elements (icons, arrows, decorations)
- Third-party components that hardcode `left`/`right`

---

## The Right Approach: CSS Logical Properties

Instead of `margin-left`, use `margin-inline-start`. It automatically maps to the correct side based on direction.

```css
/* ❌ breaks in RTL */
.icon { margin-left: 8px; }

/* ✅ works in both directions */
.icon { margin-inline-start: 8px; }
```

In Tailwind CSS v3+, use the `ps-*` / `pe-*` / `ms-*` / `me-*` utilities:

```html
<!-- ✅ start = left in LTR, right in RTL -->
<div class="ps-4 ms-2">...</div>
```

---

## Switching Direction with i18next

```js
import i18n from 'i18next';

i18n.on('languageChanged', (lng) => {
  const dir = lng === 'ar' ? 'rtl' : 'ltr';
  document.documentElement.dir = dir;
  document.documentElement.lang = lng;
  document.body.classList.toggle('arabic', lng === 'ar');
});
```

---

## Handling Third-Party Components

Some libraries hardcode `left`/`right`. The cleanest fix is a wrapper that flips transforms when in RTL:

```jsx
const RTLAware = ({ children }) => {
  const { i18n } = useTranslation();
  const isRTL = i18n.language === 'ar';

  return (
    <div style={{ transform: isRTL ? 'scaleX(-1)' : 'none' }}>
      {children}
    </div>
  );
};
```

Use this carefully , only for icons/arrows that are directional by nature (e.g. back/forward arrows, sliders).

---

## Tailwind RTL Support

### Tailwind v3+ (no plugin needed)

From v3, logical property utilities are built-in — `ps-*`, `pe-*`, `ms-*`, `me-*`, `start-*`, `end-*`. Just use them directly.

Enable the `rtl:` variant for edge cases by setting `dir="rtl"` on a parent element:

```html
<div class="text-left rtl:text-right">...</div>
```

### Tailwind v2 — `tailwindcss-rtl` plugin

Before v3, the go-to solution was the [`tailwindcss-rtl`](https://github.com/20lives/tailwindcss-rtl) plugin by Mohamad Hammoud. It added the same logical property utilities (`ps-*`, `pe-*`, etc.) as a plugin.

```js
// tailwind.config.js
module.exports = {
  plugins: [require('tailwindcss-rtl')],
};
```

If you're on Tailwind v3+, you don't need it — the utilities are already there.

---

## Common Pitfalls

| Issue | Wrong | Right |
|---|---|---|
| Spacing | `pl-4` / `pr-4` | `ps-4` / `pe-4` |
| Flex row | `flex-row` (hardcoded) | `flex-row rtl:flex-row-reverse` |
| Absolute position | `left-0` | `start-0` |
| Border side | `border-l-2` | `border-s-2` |

---

## Summary

- Use CSS logical properties (`inline-start`, `inline-end`) everywhere
- Switch `dir` and `lang` on `<html>` when language changes
- Use Tailwind's `ps-*`/`pe-*`/`start-*`/`end-*` utilities
- Test every component in both directions before shipping
