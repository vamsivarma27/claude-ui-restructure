# Apple Style Engine

Apply Apple's signature Human Interface Design language — characterized by depth, translucency, refined typography, and generous whitespace.

---

## Design Philosophy

- **Depth through layers:** Elements float above a blurred background rather than sitting flat
- **Clarity:** Typography does the heavy lifting; decorations are minimal
- **Deference:** UI recedes to let content shine
- **Translucency:** Frosted glass effect connects layers without hard boundaries

---

## Spacing Scale

```
Base unit: 8px

xs:  4px   (0.5 * base)
sm:  8px   (1x base)
md:  16px  (2x base)
lg:  24px  (3x base)
xl:  40px  (5x base)
2xl: 64px  (8x base)
3xl: 96px  (12x base)
```

Tailwind mapping:
```
gap-2  → 8px
gap-4  → 16px
gap-6  → 24px
gap-10 → 40px
p-4    → 16px
p-6    → 24px
p-10   → 40px
```

---

## Radius Scale

```
sm:   8px   (subtle rounding)
md:   12px  (standard cards)
lg:   16px  (large panels)
xl:   20px  (modals, sheets)
full: 9999px (pills, tags)
```

---

## Typography Scale

```
Display:   72px / font-weight: 700 / tracking: -0.03em
H1:        48px / font-weight: 700 / tracking: -0.02em
H2:        34px / font-weight: 600 / tracking: -0.01em
H3:        24px / font-weight: 600 / tracking: -0.005em
Body:      17px / font-weight: 400 / line-height: 1.6
Small:     13px / font-weight: 400 / line-height: 1.5
Caption:   11px / font-weight: 500 / tracking: 0.04em / uppercase
```

Font stack:
```
-apple-system, BlinkMacSystemFont, "SF Pro Display", "SF Pro Text", "Helvetica Neue", Arial, sans-serif
```

---

## Color System

```
Background:
  primary:    rgba(255, 255, 255, 0.72)   /* Frosted white */
  secondary:  rgba(242, 242, 247, 0.80)   /* System gray 6 */
  elevated:   rgba(255, 255, 255, 0.90)   /* Elevated surface */

Text:
  primary:    rgb(0, 0, 0)                /* Pure black */
  secondary:  rgba(60, 60, 67, 0.60)     /* System gray */
  tertiary:   rgba(60, 60, 67, 0.30)     /* Placeholder */

Accent:
  blue:       rgb(0, 122, 255)
  purple:     rgb(175, 82, 222)
  pink:       rgb(255, 45, 85)
  orange:     rgb(255, 149, 0)
  green:      rgb(52, 199, 89)

Separators:
  opaque:     rgb(198, 198, 200)
  non-opaque: rgba(60, 60, 67, 0.12)
```

Dark mode:
```
Background:
  primary:    rgba(28, 28, 30, 0.80)
  secondary:  rgba(44, 44, 46, 0.80)
  elevated:   rgba(58, 58, 60, 0.80)

Text:
  primary:    rgb(255, 255, 255)
  secondary:  rgba(235, 235, 245, 0.60)
  tertiary:   rgba(235, 235, 245, 0.30)
```

---

## Shadow Scale

```
xs:   0 1px 2px rgba(0,0,0,0.04), 0 1px 3px rgba(0,0,0,0.06)
sm:   0 2px 8px rgba(0,0,0,0.08), 0 1px 3px rgba(0,0,0,0.06)
md:   0 4px 16px rgba(0,0,0,0.10), 0 2px 6px rgba(0,0,0,0.06)
lg:   0 8px 32px rgba(0,0,0,0.12), 0 4px 12px rgba(0,0,0,0.08)
xl:   0 20px 60px rgba(0,0,0,0.15), 0 8px 24px rgba(0,0,0,0.10)
```

---

## Glass Effect (CSS)

```css
.glass-panel {
  background: rgba(255, 255, 255, 0.72);
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border: 1px solid rgba(255, 255, 255, 0.18);
  border-radius: 16px;
}

.glass-panel-dark {
  background: rgba(28, 28, 30, 0.80);
  backdrop-filter: blur(20px) saturate(180%);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 16px;
}
```

---

## Layout Philosophy

- **Navigation:** Fixed top nav with glassmorphism background, centered brand mark
- **Content area:** Wide max-width (1200px), generous side padding (48px+)
- **Cards:** Floating above page with soft shadows + glass backgrounds
- **Sections:** Tall, breathing sections (padding-y: 80px+)
- **Sidebar (if present):** Translucent, blurred, overlays content on mobile
- **Modals/Sheets:** Slide up from bottom (mobile), fade center (desktop), heavy blur backdrop

---

## Component Patterns

### Navigation
```jsx
<nav className="fixed top-0 w-full bg-white/70 backdrop-blur-xl border-b border-black/5 z-50">
  <div className="max-w-6xl mx-auto px-12 h-16 flex items-center justify-between">
    {/* logo | links | actions */}
  </div>
</nav>
```

### Card
```jsx
<div className="bg-white/90 backdrop-blur-sm rounded-2xl shadow-lg border border-white/20 p-6">
  {/* content */}
</div>
```

### Button (Primary)
```jsx
<button className="bg-blue-500 hover:bg-blue-600 text-white font-medium text-sm px-5 py-2.5 rounded-xl transition-all duration-200 shadow-sm hover:shadow-md">
  {label}
</button>
```

### Section
```jsx
<section className="py-20 px-12">
  <div className="max-w-5xl mx-auto">
    {/* content */}
  </div>
</section>
```

---

## Motion & Transition

Apple motion is smooth, organic, and slightly dramatic — reflecting physical materials.

```
Duration scale:
  micro:   120ms   → Button press, checkbox, toggle
  fast:    200ms   → Tooltip, badge, icon swap
  normal:  280ms   → Dropdown, hover card, popover
  medium:  360ms   → Modal appear, drawer slide
  slow:    500ms   → Page transition, onboarding step

Easing:
  entrance:   cubic-bezier(0.0, 0.0, 0.2, 1)    /* Ease-out — decelerates into position */
  exit:       cubic-bezier(0.4, 0.0, 1, 1)       /* Ease-in — accelerates out */
  spring:     cubic-bezier(0.34, 1.56, 0.64, 1)  /* Overshoot spring — for button press, toggles */
  smooth:     cubic-bezier(0.4, 0.0, 0.2, 1)     /* Ease-in-out — for two-way transitions */
```

Tailwind patterns:
```
Interactive buttons:    transition-all duration-200 ease-out active:scale-[0.97]
Card hover:             transition-all duration-300 ease-out hover:-translate-y-1 hover:shadow-lg
Input focus:            transition-colors duration-150 ease-out
Modal/sheet entrance:   duration-[360ms] ease-out (slide up from bottom on mobile)
Backdrop blur toggle:   duration-200 ease-out
```

---

## Tailwind Config Additions

```js
// tailwind.config.ts
theme: {
  extend: {
    backdropBlur: {
      xs: '2px',
      apple: '20px',
    },
    borderRadius: {
      'apple-sm': '8px',
      'apple-md': '12px',
      'apple-lg': '16px',
      'apple-xl': '20px',
    },
    boxShadow: {
      'apple-xs': '0 1px 2px rgba(0,0,0,0.04), 0 1px 3px rgba(0,0,0,0.06)',
      'apple-sm': '0 2px 8px rgba(0,0,0,0.08), 0 1px 3px rgba(0,0,0,0.06)',
      'apple-md': '0 4px 16px rgba(0,0,0,0.10), 0 2px 6px rgba(0,0,0,0.06)',
      'apple-lg': '0 8px 32px rgba(0,0,0,0.12), 0 4px 12px rgba(0,0,0,0.08)',
      'apple-xl': '0 20px 60px rgba(0,0,0,0.15), 0 8px 24px rgba(0,0,0,0.10)',
    },
  }
}
```
