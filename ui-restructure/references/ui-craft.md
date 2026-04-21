# UI Craft — World-Class Frontend Engineering Reference

This file encodes 25 years of production frontend engineering expertise. Apply every principle in this file during any restructure. A good-looking UI that feels dead is not world-class. A world-class UI breathes, responds, gives feedback, and respects the user's attention.

---

## The Standard

Before shipping any component ask: **Would this pass on Stripe, Linear, Vercel, or Apple.com?**

If the answer is no — keep refining.

---

## 1. Motion & Animation

### The Golden Rules

- **Nothing should snap.** Every state change needs a transition.
- **Duration is proportional to distance.** A tooltip appearing = 120ms. A full-page modal = 280ms. A sidebar expanding = 220ms.
- **Ease out for entrances. Ease in for exits. Spring for interactions.**
- **Never animate more than opacity + transform at the same time on the same element.** GPU-composited only. Never animate `height`, `width`, `top`, `left`, `margin`, `padding` — these cause layout thrash and feel sluggish.

### Duration Scale

```
Instant:    0ms     → No visual change needed (state updates)
Micro:      80ms    → Button press, checkbox tick, toggle
Fast:       120ms   → Tooltip appear, badge update, icon swap
Normal:     200ms   → Dropdown open, popover, hover card
Medium:     280ms   → Modal appear, drawer slide, page transition
Slow:       400ms   → Onboarding step, empty state illustration
```

### Easing Curves

```css
/* Entrance — decelerates into position */
--ease-out:    cubic-bezier(0.0, 0.0, 0.2, 1);

/* Exit — accelerates out of view */
--ease-in:     cubic-bezier(0.4, 0.0, 1, 1);

/* Both directions — for elements that appear and disappear */
--ease-in-out: cubic-bezier(0.4, 0.0, 0.2, 1);

/* Spring — for interactive elements (buttons, toggles, checkboxes) */
--spring:      cubic-bezier(0.34, 1.56, 0.64, 1);

/* Linear — only for continuous animations (spinners, progress bars) */
--linear:      linear;
```

### Framer Motion Presets (React)

```tsx
// Fade + slide up (modal, card entrance)
const fadeUp = {
  initial: { opacity: 0, y: 8 },
  animate: { opacity: 1, y: 0 },
  exit:    { opacity: 0, y: 4 },
  transition: { duration: 0.22, ease: [0.0, 0.0, 0.2, 1] }
}

// Scale spring (button press feedback)
const buttonPress = {
  whileTap: { scale: 0.97 },
  transition: { type: "spring", stiffness: 400, damping: 20 }
}

// Stagger children (list/grid entrance)
const staggerContainer = {
  animate: { transition: { staggerChildren: 0.05 } }
}

const staggerItem = {
  initial: { opacity: 0, y: 12 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.2, ease: [0.0, 0.0, 0.2, 1] }
}

// Sidebar slide
const sidebarVariants = {
  open:   { x: 0, transition: { type: "spring", stiffness: 300, damping: 30 } },
  closed: { x: "-100%", transition: { duration: 0.22, ease: [0.4, 0.0, 1, 1] } }
}
```

### CSS Transition Patterns

```css
/* Button — all interactive properties */
.button {
  transition: background-color 120ms ease-out,
              box-shadow 120ms ease-out,
              transform 80ms ease-out,
              color 120ms ease-out;
}

/* Card hover lift */
.card {
  transition: transform 200ms cubic-bezier(0.0, 0.0, 0.2, 1),
              box-shadow 200ms cubic-bezier(0.0, 0.0, 0.2, 1);
}
.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0,0,0,0.12);
}

/* Input focus */
.input {
  transition: border-color 150ms ease-out,
              box-shadow 150ms ease-out;
}
.input:focus {
  border-color: var(--color-accent);
  box-shadow: 0 0 0 3px rgba(var(--color-accent-rgb), 0.15);
}
```

---

## 2. Micro-Interactions

Every interactive element has at minimum 5 states. Never skip states.

### The 5 States (mandatory)

```
Default   → Resting state
Hover     → Cursor over element
Focus     → Keyboard focused (accessibility critical)
Active    → Mouse/finger pressed
Disabled  → Not interactable
```

Extended states (add when applicable):
```
Loading   → Async action in progress
Success   → Action completed
Error     → Action failed
Empty     → No data
```

### Button Micro-Interaction

```tsx
// World-class button — every state defined
<button
  className="
    relative inline-flex items-center justify-center gap-2
    px-4 py-2 rounded-lg text-sm font-medium
    bg-neutral-900 text-white
    /* Hover: subtle lighten */
    hover:bg-neutral-800
    /* Active: press down */
    active:scale-[0.97] active:bg-neutral-950
    /* Focus: visible ring */
    focus-visible:outline-none focus-visible:ring-2
    focus-visible:ring-offset-2 focus-visible:ring-neutral-900
    /* Disabled: muted, no pointer */
    disabled:opacity-40 disabled:cursor-not-allowed
    disabled:pointer-events-none
    /* Transition */
    transition-all duration-[120ms] ease-out
  "
>
  {isLoading ? <Spinner size={14} /> : children}
</button>
```

### Hover Card Lift

```css
.interactive-card {
  cursor: pointer;
  transition: transform 200ms cubic-bezier(0,0,0.2,1),
              box-shadow 200ms cubic-bezier(0,0,0.2,1);
}
.interactive-card:hover {
  transform: translateY(-2px) scale(1.005);
  box-shadow: 0 12px 32px rgba(0,0,0,0.10), 0 4px 8px rgba(0,0,0,0.06);
}
.interactive-card:active {
  transform: translateY(0px) scale(0.998);
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
}
```

### Toggle Switch

```tsx
// Smooth toggle with spring
<motion.div
  className={`w-11 h-6 rounded-full p-0.5 cursor-pointer ${
    on ? 'bg-blue-500' : 'bg-neutral-300'
  }`}
  onClick={() => setOn(!on)}
  layout
  transition={{ type: "spring", stiffness: 500, damping: 30 }}
>
  <motion.div
    className="w-5 h-5 bg-white rounded-full shadow-sm"
    layout
    animate={{ x: on ? 20 : 0 }}
    transition={{ type: "spring", stiffness: 500, damping: 30 }}
  />
</motion.div>
```

---

## 3. Typography Mastery

### The 5 Rules

1. **Never use more than 2 font families.** Display + body, or one variable font.
2. **Line length is 60–75 characters.** Use `max-w-prose` (65ch). Beyond this, readability drops.
3. **Line height scales inversely with font size.** Large headings: 1.1. Body: 1.6–1.7. Small labels: 1.4.
4. **Optical size matters.** At 12px, increase letter-spacing. At 48px+, use negative letter-spacing.
5. **Color contrast minimum WCAG AA.** Body text on background ≥ 4.5:1.

### Type Scale (8pt rhythm-based)

```css
:root {
  --text-xs:   0.75rem;   /* 12px — captions, labels, meta */
  --text-sm:   0.875rem;  /* 14px — secondary body, UI chrome */
  --text-base: 1rem;      /* 16px — primary body copy */
  --text-lg:   1.125rem;  /* 18px — intro text, callouts */
  --text-xl:   1.25rem;   /* 20px — small headings */
  --text-2xl:  1.5rem;    /* 24px — section headings */
  --text-3xl:  1.875rem;  /* 30px — page headings */
  --text-4xl:  2.25rem;   /* 36px — hero headings */
  --text-5xl:  3rem;      /* 48px — display headings */

  /* Line heights — tighter at top, looser at bottom */
  --leading-tight:  1.1;
  --leading-snug:   1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.625;
  --leading-loose:  1.75;

  /* Letter spacing */
  --tracking-tight:  -0.025em;  /* Large headings */
  --tracking-snug:   -0.01em;   /* Medium headings */
  --tracking-normal:  0em;      /* Body */
  --tracking-wide:    0.025em;  /* Small caps, labels */
  --tracking-widest:  0.08em;   /* Uppercase labels, tags */
}
```

### Font Pairings (proven, production-grade)

```
1. Inter (body) + Cal Sans / Geist (display)
   → SaaS, dashboards, developer tools

2. Inter (body) + Inter (display, heavier weight)
   → Clean minimal, single-font approach

3. -apple-system (body) + -apple-system (display)
   → Native-feeling iOS/macOS apps

4. Geist (body) + Geist Mono (code)
   → Vercel aesthetic, developer-first

5. DM Sans (body) + DM Serif Display (display)
   → Editorial, premium, brand-forward

6. Söhne / Circular (body) — premium paid fonts
   → Agency-quality, luxury brands
```

---

## 4. Iconography

### Sizing Rules

```
12px → Inline with 12px text only (very small labels)
14px → Sidebar icon labels, table actions
16px → Standard UI icons (matches 16px body text perfectly)
18px → Slightly prominent icons, nav items
20px → Card actions, feature icons
24px → Section icons, empty state icons
32px → Large feature illustrations
48px+ → Marketing/hero icons
```

**Critical:** Icon size must optically align with adjacent text size.
```
12px text → 12–14px icon
14px text → 14–16px icon  ← most common
16px text → 16–18px icon
```

### Stroke Weight

```
Thin (1px):   → Light interfaces, Apple-style
Regular (1.5px): → Most SaaS apps (Lucide, Heroicons default)
Medium (2px):  → Bold interfaces, dashboards (Phosphor bold)
```

**Never mix stroke weights.** Pick one library and one weight for the entire app.

### Recommended Icon Libraries

```
Lucide React     → Default. Clean 1.5px stroke. 1000+ icons.
                   import { ChevronRight } from 'lucide-react'

Phosphor React   → 6 weights (thin to bold). Most flexible.
                   import { ArrowRight } from '@phosphor-icons/react'

Heroicons        → Tailwind team. Outline + solid variants.
                   import { ArrowRightIcon } from '@heroicons/react/24/outline'

Radix Icons      → Pixel-perfect, component-scale. Best for small sizes.
                   import { ChevronRightIcon } from '@radix-ui/react-icons'
```

### Optical Alignment

Icons visually appear smaller than their bounding box. Always:
```tsx
// Don't center by bounding box — optically center
<div className="flex items-center gap-2">
  <ArrowRight size={16} className="mt-[0.5px]" /> {/* micro-adjust */}
  <span className="text-sm">Continue</span>
</div>
```

---

## 5. Color Theory & Hierarchy

### 60-30-10 Rule

```
60% → Base/neutral (backgrounds, surfaces)
30% → Secondary (text, borders, subtle elements)
10% → Accent (primary actions, links, highlights)
```

### Semantic Color Layers (always define these)

```css
:root {
  /* Backgrounds — 3 levels of depth */
  --bg-base:     ;  /* Page background */
  --bg-surface:  ;  /* Card, panel surface */
  --bg-elevated: ;  /* Dropdown, modal, tooltip */
  --bg-overlay:  ;  /* Modal backdrop */

  /* Text — 4 levels of hierarchy */
  --text-primary:   ;  /* Headlines, important info */
  --text-secondary: ;  /* Body, descriptions */
  --text-tertiary:  ;  /* Placeholders, hints */
  --text-disabled:  ;  /* Disabled state */
  --text-inverse:   ;  /* Text on dark backgrounds */

  /* Borders — 3 levels */
  --border-subtle:  ;  /* Dividers, section separators */
  --border-default: ;  /* Input borders, card borders */
  --border-strong:  ;  /* Focused inputs, strong dividers */

  /* Interaction */
  --accent:         ;  /* Primary CTA, links */
  --accent-hover:   ;  /* Hovered primary */
  --accent-subtle:  ;  /* Accent tint for backgrounds */

  /* Status */
  --success:  ;
  --warning:  ;
  --error:    ;
  --info:     ;
}
```

### Contrast Minimums

```
Normal text (< 18px):     4.5:1 minimum (WCAG AA)
Large text (≥ 18px bold): 3:1 minimum
UI components (buttons, inputs): 3:1 against background
Focus indicators: 3:1 against adjacent colors
```

---

## 6. Spacing Rhythm (8pt Grid)

**Every spacing value must be a multiple of 4 or 8.**

```
Never use: 5px, 7px, 9px, 11px, 13px, 15px, 17px
Always use: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96px
```

### Spacing Decisions

```
2px  → Between icon and adjacent text (optical tightening)
4px  → Within a component (badge padding, tight labels)
8px  → Between related elements (icon + label, tag + tag)
12px → Compact component padding (dense table cells)
16px → Standard component padding (cards, inputs)
24px → Between components in a group
32px → Between sections within a card
48px → Between major sections on a page
64px → Between page-level sections
80px → Hero/marketing section padding
```

---

## 7. Shadow & Depth

Shadows communicate elevation. Use a strict elevation system — never make up shadow values.

```
Level 0: No shadow → Flat, same surface as background
Level 1: Subtle    → Cards, inputs resting (0 1px 3px rgba(0,0,0,0.07))
Level 2: Raised    → Dropdowns, hovered cards (0 4px 12px rgba(0,0,0,0.10))
Level 3: Floating  → Modals, popovers (0 8px 24px rgba(0,0,0,0.14))
Level 4: Overlay   → Sheets, command palette (0 20px 48px rgba(0,0,0,0.18))
```

### Shadow Crafting Principles

```css
/* Real shadows have TWO layers: ambient + key light */
.card {
  box-shadow:
    0 1px 3px rgba(0, 0, 0, 0.06),   /* Ambient: spread, low opacity */
    0 4px 12px rgba(0, 0, 0, 0.10);  /* Key: taller, sharper */
}

/* Dark mode shadows are darker (no pure black) */
.card-dark {
  box-shadow:
    0 1px 3px rgba(0, 0, 0, 0.40),
    0 4px 12px rgba(0, 0, 0, 0.50);
}

/* Colored shadows for accent elements */
.button-accent {
  box-shadow: 0 4px 14px rgba(var(--accent-rgb), 0.35);
}
```

---

## 8. Border Radius

Pick **one radius scale** and never deviate.

### Sizing Principle

```
Radius is proportional to the element size.
Small elements (badges, tags) → small radius (4–6px)
Medium elements (inputs, buttons) → medium radius (6–10px)
Large elements (cards, modals) → large radius (12–16px)
Pills (full tags, avatar chips) → 9999px
```

### The Key Rule

**Inner radius = outer radius − padding.**

```
Card has: border-radius: 16px, padding: 16px
Inner element must have: border-radius: 8px (not 16px!)
```

Applying the same radius to nested elements is a common mistake. It kills the layered depth.

---

## 9. Loading & Empty States

Every state must be designed. Unstyled loading = unfinished product.

### Loading Skeleton

```tsx
// Shimmer skeleton — GPU-only animation
const Skeleton = ({ className }: { className?: string }) => (
  <div
    className={`animate-pulse rounded bg-neutral-200 dark:bg-neutral-800 ${className}`}
  />
)

// Shimmer variant (more polished)
// globals.css:
// @keyframes shimmer {
//   from { background-position: -200px 0; }
//   to   { background-position: calc(200px + 100%) 0; }
// }
// .shimmer {
//   background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
//   background-size: 400px 100%;
//   animation: shimmer 1.4s ease-in-out infinite;
// }
```

### Empty State

```tsx
// Every list/table needs an empty state
const EmptyState = ({ icon: Icon, title, description, action }) => (
  <div className="flex flex-col items-center justify-center py-20 px-6 text-center">
    <div className="w-12 h-12 rounded-xl bg-neutral-100 flex items-center justify-center mb-4">
      <Icon size={22} className="text-neutral-400" />
    </div>
    <h3 className="text-sm font-semibold text-neutral-900 mb-1">{title}</h3>
    <p className="text-sm text-neutral-500 max-w-xs mb-6">{description}</p>
    {action && (
      <button className="...">{action.label}</button>
    )}
  </div>
)
```

### Error State

```tsx
const ErrorState = ({ message, onRetry }) => (
  <div className="flex flex-col items-center justify-center py-16 gap-3">
    <div className="w-10 h-10 rounded-full bg-red-50 flex items-center justify-center">
      <AlertCircle size={18} className="text-red-500" />
    </div>
    <p className="text-sm text-neutral-600">{message}</p>
    {onRetry && (
      <button onClick={onRetry} className="text-sm text-blue-600 hover:underline">
        Try again
      </button>
    )}
  </div>
)
```

---

## 10. Focus & Accessibility

**Keyboard navigation must work perfectly.** This is non-negotiable.

```css
/* Remove default outline — replace with better one */
*:focus {
  outline: none;
}
*:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
  border-radius: 4px;
}

/* For dark backgrounds */
.dark *:focus-visible {
  outline-color: white;
}
```

```tsx
// Every interactive element needs accessible props
<button
  aria-label="Close dialog"
  aria-pressed={isActive}
  aria-disabled={isDisabled}
  tabIndex={isDisabled ? -1 : 0}
>
```

### Touch Targets (mobile)

```
Minimum touch target: 44×44px (Apple HIG) / 48×48dp (Material)
Never make tap targets smaller than 40px in any dimension.
Add padding to small elements to hit the target without changing visual size.
```

```css
.small-button {
  /* Visually 24×24px, touch target 44×44px */
  width: 24px;
  height: 24px;
  padding: 10px;
  margin: -10px;
}
```

---

## 11. Polished Details Checklist

Run this checklist on every component before marking it done:

```
Typography
  □ Line height appropriate for font size
  □ No orphaned words in paragraphs (use text-balance)
  □ Max line length respected (max-w-prose on body copy)
  □ Heading tracking tightened at large sizes
  □ Label text uppercase + tracked

Spacing
  □ All values on 4/8px grid
  □ Inner radius = outer radius − padding
  □ Consistent padding within same component category

Icons
  □ Consistent icon library used
  □ Consistent stroke weight
  □ Optically aligned with adjacent text
  □ aria-hidden="true" on all decorative icons (no semantic meaning)
  □ aria-label on all icon-only interactive buttons

Motion
  □ Every state transition has a duration + easing
  □ No layout properties animated (height, width, margin, padding)
  □ Entrance uses ease-out, exit uses ease-in
  □ Interactive elements have whileTap/active state

Color
  □ Semantic tokens used (not hardcoded hex)
  □ All text passes WCAG AA contrast
  □ Focus states visible on all backgrounds

States
  □ Default, hover, focus, active, disabled all defined
  □ Loading state designed (skeleton or spinner)
  □ Empty state designed
  □ Error state designed

Accessibility
  □ All interactive elements keyboard navigable
  □ focus-visible styles applied
  □ Touch targets ≥ 44×44px on mobile
  □ ARIA labels on icon-only buttons

Performance
  □ Only opacity + transform animated (no layout properties)
  □ will-change applied only when needed and removed after animation
  □ Images have explicit width/height (prevent CLS)
```

---

## 12. What Separates World-Class from Average

| Average UI | World-Class UI |
|---|---|
| Snaps between states | Everything transitions |
| One font, no weight variation | Intentional type hierarchy |
| Hardcoded spacing | Strict 8pt grid, never deviates |
| Same radius everywhere | Proportional radius system |
| Flat shadows or no shadows | Two-layer ambient + key shadows |
| No empty/loading/error states | Every state is designed |
| Default browser focus ring | Custom focus-visible style |
| Icon library inconsistency | One library, one weight, every page |
| Touch targets too small | 44px minimum everywhere |
| Layout animations | Only opacity + transform |
| No micro-interactions | Hover lift, press scale, spring toggles |
| Monochrome throughout | 60-30-10 color rule applied |

Apply every row in this table. Every single one.
