# Linear Style Engine

Apply Linear's signature design language — sharp, dense, highly functional. Every pixel serves a purpose. No decorative flourishes.

---

## Design Philosophy

- **Density:** Pack maximum information into minimum space
- **Speed:** Interactions feel instant — tight transitions (80–150ms)
- **Precision:** 1px borders, exact alignment, no rounding excess
- **Restraint:** Monochrome-first, color only for status/action
- **Focus:** Content over chrome — UI disappears, work remains

---

## Spacing Scale

```
Base unit: 4px

2xs: 2px   (0.5x base)
xs:  4px   (1x base)
sm:  8px   (2x base)
md:  12px  (3x base)
lg:  16px  (4x base)
xl:  24px  (6x base)
2xl: 32px  (8x base)
3xl: 48px  (12x base)
```

Tailwind mapping:
```
gap-1  → 4px
gap-2  → 8px
gap-3  → 12px
gap-4  → 16px
gap-6  → 24px
p-2    → 8px
p-3    → 12px
p-4    → 16px
```

---

## Radius Scale

```
none: 0px
xs:   2px   (subtle — inputs, tags)
sm:   4px   (standard — buttons, cards)
md:   6px   (dropdowns, popovers)
```

Linear does NOT use large radius values.

---

## Typography Scale

```
H1:    24px / font-weight: 600 / tracking: -0.01em
H2:    18px / font-weight: 600 / tracking: -0.005em
H3:    15px / font-weight: 600 / tracking: 0
H4:    13px / font-weight: 600 / tracking: 0
Body:  14px / font-weight: 400 / line-height: 1.5
Small: 12px / font-weight: 400 / line-height: 1.4
Mono:  13px / font-family: monospace / line-height: 1.6
```

Font stack:
```
"Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif
```

---

## Color System

```
Background:
  app:        #0F0F0F   /* Deep app background */
  surface:    #141414   /* Card/panel surface */
  elevated:   #1A1A1A   /* Elevated elements */
  overlay:    #1F1F1F   /* Modals, dropdowns */

Border:
  subtle:     rgba(255,255,255,0.06)
  default:    rgba(255,255,255,0.10)
  strong:     rgba(255,255,255,0.16)

Text:
  primary:    rgba(255,255,255,0.92)
  secondary:  rgba(255,255,255,0.50)
  tertiary:   rgba(255,255,255,0.30)
  disabled:   rgba(255,255,255,0.18)

Accent (purple — Linear's signature):
  default:    #5E6AD2
  hover:      #6B7ADE
  active:     #5360C8

Status:
  success:    #4ADE80
  warning:    #FB923C
  error:      #F87171
  info:       #60A5FA
```

Light mode (when applicable):
```
Background:
  app:        #FFFFFF
  surface:    #F7F7F7
  elevated:   #FFFFFF
  overlay:    #FFFFFF

Border:
  subtle:     rgba(0,0,0,0.05)
  default:    rgba(0,0,0,0.09)
  strong:     rgba(0,0,0,0.14)

Text:
  primary:    #0D0D0D
  secondary:  rgba(0,0,0,0.50)
  tertiary:   rgba(0,0,0,0.35)
```

---

## Shadow Scale

Linear uses almost no shadows — separation comes from borders.

```
none: none
xs:   0 1px 2px rgba(0,0,0,0.40)
sm:   0 2px 4px rgba(0,0,0,0.50)
md:   0 4px 8px rgba(0,0,0,0.60)
```

---

## Layout Philosophy

- **Navigation:** Fixed left sidebar (240px wide), icon-only collapsed (48px)
- **Sidebar sections:** Teams, Projects, Views — separated by thin dividers
- **Content area:** Takes remaining width, no extra padding waste
- **Header bar:** 48px tall, border-bottom only — no shadow
- **Command palette:** K-bar style modal (⌘K) — central interaction point
- **Tables/Lists:** Full-width, 40px row height, hover states only
- **No hero sections** — Linear is an app, not a marketing site

---

## Component Patterns

### App Shell
```jsx
<div className="flex h-screen bg-[#0F0F0F] text-white overflow-hidden">
  {/* Sidebar */}
  <aside className="w-60 flex-shrink-0 border-r border-white/10 flex flex-col">
    {/* nav items */}
  </aside>
  {/* Main */}
  <main className="flex-1 overflow-auto">
    <header className="h-12 border-b border-white/10 flex items-center px-4">
      {/* breadcrumb + actions */}
    </header>
    <div className="p-6">
      {/* content */}
    </div>
  </main>
</div>
```

### List Item (Issue row)
```jsx
<div className="flex items-center h-10 px-4 hover:bg-white/5 border-b border-white/5 cursor-pointer gap-3">
  <StatusIcon />
  <span className="text-sm text-white/90 flex-1 truncate">{title}</span>
  <span className="text-xs text-white/40">{id}</span>
  <Avatar size="xs" />
</div>
```

### Card (minimal)
```jsx
<div className="bg-[#141414] border border-white/10 rounded-sm p-3">
  {/* content */}
</div>
```

### Button (Primary)
```jsx
<button className="bg-[#5E6AD2] hover:bg-[#6B7ADE] text-white text-xs font-medium px-3 py-1.5 rounded transition-colors duration-75">
  {label}
</button>
```

### Button (Ghost)
```jsx
<button className="text-white/60 hover:text-white/90 hover:bg-white/8 text-xs font-medium px-2 py-1 rounded transition-colors duration-75">
  {label}
</button>
```

---

## Motion & Transition

Linear motion is instant and precise — every transition is tighter than you expect. Speed signals power.

```
Duration scale:
  instant:  75ms    → Button press, checkbox, toggle (Linear's signature snap)
  micro:    100ms   → Hover state on list items, sidebar items
  fast:     150ms   → Dropdown open, popover appear
  normal:   200ms   → Modal appear, command palette
  medium:   250ms   → Sidebar expand/collapse

Easing:
  default:  linear (most UI elements — precision over decoration)
  entrance: cubic-bezier(0.0, 0.0, 0.2, 1)    /* ease-out for modal/overlay entrances */
  exit:     cubic-bezier(0.4, 0.0, 1, 1)       /* ease-in for exits */
  spring:   cubic-bezier(0.34, 1.56, 0.64, 1)  /* spring — only for toggles/active press */
```

Tailwind patterns:
```
Interactive buttons:    transition-colors duration-75 active:scale-[0.97]
List item hover:        transition-colors duration-[100ms]
Input focus:            transition-colors duration-[100ms]
Sidebar item:           transition-colors duration-75
Command palette:        duration-200 ease-out (enter), duration-150 ease-in (exit)
```

---

## Tailwind Config Additions

```js
theme: {
  extend: {
    colors: {
      linear: {
        app: '#0F0F0F',
        surface: '#141414',
        elevated: '#1A1A1A',
        border: 'rgba(255,255,255,0.10)',
        accent: '#5E6AD2',
      }
    },
    borderRadius: {
      'linear': '4px',
      'linear-sm': '2px',
    },
    transitionDuration: {
      '75': '75ms',
    },
    fontSize: {
      'linear-xs': ['12px', '1.4'],
      'linear-sm': ['13px', '1.5'],
      'linear-base': ['14px', '1.5'],
    },
  }
}
```
