# Minimal Style Engine

Apply a clean, neutral, content-first design system. No visual noise. Typography-driven hierarchy. Functional without flair.

---

## Design Philosophy

- **Reduction:** Remove everything that doesn't serve the content
- **Neutrality:** No strong personality — adapts to any brand
- **Typography-first:** Hierarchy through size and weight, not decoration
- **Whitespace:** Space is a design element — use it generously
- **Flat:** No shadows unless absolutely necessary for depth cue

---

## Spacing Scale

```
Base unit: 8px

xs:  4px
sm:  8px
md:  16px
lg:  24px
xl:  32px
2xl: 48px
3xl: 64px
4xl: 96px
```

---

## Radius Scale

```
none: 0px
sm:   4px
md:   6px
lg:   8px
```

Keep radius small and consistent.

---

## Typography Scale

```
H1:    36px / font-weight: 700 / tracking: -0.02em / line-height: 1.2
H2:    28px / font-weight: 600 / tracking: -0.01em / line-height: 1.3
H3:    22px / font-weight: 600 / tracking: -0.005em / line-height: 1.4
H4:    18px / font-weight: 600 / line-height: 1.4
Body:  16px / font-weight: 400 / line-height: 1.7
Small: 14px / font-weight: 400 / line-height: 1.5
Label: 12px / font-weight: 500 / tracking: 0.05em / uppercase
```

Font stack:
```
"Inter", system-ui, -apple-system, sans-serif
```

---

## Color System

```
Background:
  page:       #FFFFFF
  surface:    #FAFAFA
  muted:      #F5F5F5

Border:
  light:      #F0F0F0
  default:    #E5E5E5
  strong:     #D4D4D4

Text:
  primary:    #171717
  secondary:  #525252
  tertiary:   #A3A3A3
  disabled:   #D4D4D4

Accent:
  default:    #171717   /* Black — minimal uses near-black as accent */
  subtle:     #F5F5F5

Status:
  success:    #16A34A
  warning:    #D97706
  error:      #DC2626
  info:       #2563EB
```

---

## Shadow Scale

```
none: none
xs:   0 1px 2px rgba(0,0,0,0.05)
sm:   0 1px 3px rgba(0,0,0,0.07), 0 1px 2px rgba(0,0,0,0.04)
```

Use shadows sparingly — only to indicate interactivity (hover state lift) or modal depth.

---

## Layout Philosophy

- **Navigation:** Clean top bar, left-aligned logo, right-aligned actions
- **Max width:** 1200px container, 48px horizontal padding
- **Content sections:** Separated by whitespace (80px+), not dividers
- **Cards:** Flat with border, no shadows at rest
- **Forms:** Single column, generous label spacing
- **Tables:** Borderless rows, header weight distinguishes from data

---

## Component Patterns

### Navigation
```jsx
<nav className="border-b border-neutral-200">
  <div className="max-w-6xl mx-auto px-12 h-14 flex items-center justify-between">
    <a className="text-base font-semibold text-neutral-900">{brand}</a>
    <div className="flex items-center gap-8">
      {links}
    </div>
    <div className="flex items-center gap-3">
      {actions}
    </div>
  </div>
</nav>
```

### Card
```jsx
<div className="bg-white border border-neutral-200 rounded-md p-6">
  {/* content */}
</div>
```

### Button (Primary)
```jsx
<button className="bg-neutral-900 hover:bg-neutral-800 text-white text-sm font-medium px-4 py-2 rounded transition-colors">
  {label}
</button>
```

### Button (Secondary)
```jsx
<button className="bg-white hover:bg-neutral-50 text-neutral-900 text-sm font-medium px-4 py-2 rounded border border-neutral-300 transition-colors">
  {label}
</button>
```

### Section
```jsx
<section className="py-20">
  <div className="max-w-5xl mx-auto px-12">
    <h2 className="text-3xl font-semibold text-neutral-900 mb-4">{heading}</h2>
    <p className="text-neutral-600 text-lg mb-12">{subheading}</p>
    {/* content */}
  </div>
</section>
```

### Input
```jsx
<input className="w-full border border-neutral-300 rounded px-3 py-2 text-sm text-neutral-900 placeholder-neutral-400 focus:outline-none focus:ring-1 focus:ring-neutral-900 focus:border-neutral-900 transition" />
```

---

## Motion & Transition

Minimal motion is understated — transitions are present but never decorative. Content moves, chrome does not.

```
Duration scale:
  micro:   100ms   → Button press, checkbox, toggle
  fast:    150ms   → Input focus transition, hover state
  normal:  200ms   → Dropdown, tooltip, card hover
  medium:  250ms   → Modal appear, drawer
  slow:    350ms   → Page transition, onboarding

Easing:
  entrance:  cubic-bezier(0.0, 0.0, 0.2, 1)    /* ease-out — default for all entrances */
  exit:      cubic-bezier(0.4, 0.0, 1, 1)       /* ease-in — exits */
  spring:    cubic-bezier(0.34, 1.56, 0.64, 1)  /* spring — button press, toggles only */
```

Tailwind patterns:
```
Interactive buttons:    transition-colors duration-[100ms] ease-out active:scale-[0.97]
Card hover:             transition-shadow duration-200 ease-out hover:shadow-sm
Input focus:            transition duration-[150ms] ease-out
Navigation items:       transition-colors duration-[100ms]
Modal/drawer:           duration-[250ms] ease-out (enter), duration-200 ease-in (exit)
```

---

## Tailwind Config Additions

```js
theme: {
  extend: {
    fontFamily: {
      sans: ['Inter', 'system-ui', '-apple-system', 'sans-serif'],
    },
    colors: {
      neutral: {
        // Keep Tailwind defaults — minimal uses standard neutral palette
      }
    },
    maxWidth: {
      'content': '1200px',
    },
  }
}
```
