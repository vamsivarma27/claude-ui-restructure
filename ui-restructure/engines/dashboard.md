# Dashboard Style Engine

Apply a data-dense, analytics-first design system. Maximum information per viewport. Sidebar-heavy layout. Card grids for metrics.

---

## Design Philosophy

- **Data density:** Show as much relevant information as possible without scrolling
- **Scanability:** Visual hierarchy guides eye from KPIs → charts → tables → actions
- **Status-driven:** Colors communicate health (green/amber/red)
- **Sidebar-centric:** Navigation and context always visible
- **Grid-first:** Content organized in responsive card grids

---

## Spacing Scale

```
Base unit: 4px

2xs: 2px
xs:  4px
sm:  8px
md:  12px
lg:  16px
xl:  20px
2xl: 24px
3xl: 32px
```

Dense layout — don't waste space.

---

## Radius Scale

```
none: 0px
xs:   4px
sm:   6px
md:   8px
lg:   12px
```

---

## Typography Scale

```
Display:  32px / font-weight: 700 / tracking: -0.01em (KPI numbers)
H1:       22px / font-weight: 700
H2:       18px / font-weight: 600
H3:       15px / font-weight: 600
H4:       13px / font-weight: 600
Body:     13px / font-weight: 400 / line-height: 1.5
Small:    12px / font-weight: 400
Label:    11px / font-weight: 500 / uppercase / tracking: 0.08em
Mono:     13px / font-family: monospace (for numeric data)
```

Font stack:
```
"Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif
```

---

## Color System

Light mode (default for dashboards):
```
Background:
  page:       #F8FAFC    /* Slate-50 */
  sidebar:    #FFFFFF    /* Pure white sidebar */
  surface:    #FFFFFF    /* Card surface */
  muted:      #F1F5F9    /* Slate-100 */

Border:
  light:      #E2E8F0    /* Slate-200 */
  default:    #CBD5E1    /* Slate-300 */

Text:
  primary:    #0F172A    /* Slate-900 */
  secondary:  #475569    /* Slate-600 */
  tertiary:   #94A3B8    /* Slate-400 */

Accent:
  primary:    #3B82F6    /* Blue-500 */
  hover:      #2563EB    /* Blue-600 */

Status:
  success:    #10B981    /* Emerald-500 */
  warning:    #F59E0B    /* Amber-500 */
  error:      #EF4444    /* Red-500 */
  neutral:    #6B7280    /* Gray-500 */

Chart colors (sequential):
  1:  #3B82F6  (blue)
  2:  #10B981  (emerald)
  3:  #8B5CF6  (violet)
  4:  #F59E0B  (amber)
  5:  #EF4444  (red)
  6:  #06B6D4  (cyan)
```

---

## Shadow Scale

```
sm:   0 1px 2px rgba(0,0,0,0.05)
md:   0 1px 3px rgba(0,0,0,0.07), 0 1px 2px rgba(0,0,0,0.04)
lg:   0 4px 6px rgba(0,0,0,0.05), 0 2px 4px rgba(0,0,0,0.04)
```

---

## Layout Philosophy

- **Sidebar:** Fixed 256px wide, collapsible to 64px (icon only)
- **Top bar:** 56px, search + notifications + user avatar
- **Content area:** Card grid — 12-column grid, responsive breakpoints
- **KPI row:** Always first — 4 metric cards in a row
- **Charts:** 50/50 split or 70/30 below KPIs
- **Tables:** Full-width, paginated, sortable columns
- **Side panels:** 320–400px slide-in for detail views

---

## Component Patterns

### App Shell
```jsx
<div className="flex h-screen bg-slate-50 overflow-hidden">
  {/* Sidebar */}
  <aside className="w-64 flex-shrink-0 bg-white border-r border-slate-200 flex flex-col">
    <div className="h-14 flex items-center px-4 border-b border-slate-200">
      <span className="font-semibold text-slate-900">{brand}</span>
    </div>
    <nav className="flex-1 py-3 px-2 overflow-y-auto">
      {navItems}
    </nav>
  </aside>

  {/* Main */}
  <div className="flex-1 flex flex-col overflow-hidden">
    <header className="h-14 bg-white border-b border-slate-200 flex items-center px-6 gap-4">
      <div className="flex-1">{/* search */}</div>
      <div className="flex items-center gap-3">{/* actions */}</div>
    </header>
    <main className="flex-1 overflow-y-auto p-6">
      {/* content */}
    </main>
  </div>
</div>
```

### KPI Card
```jsx
<div className="bg-white border border-slate-200 rounded-lg p-4 shadow-sm">
  <p className="text-xs font-medium text-slate-500 uppercase tracking-wider">{label}</p>
  <p className="text-3xl font-bold text-slate-900 mt-1">{value}</p>
  <div className="flex items-center gap-1 mt-2">
    <TrendIcon />
    <span className="text-xs text-emerald-600 font-medium">{change}</span>
    <span className="text-xs text-slate-400">vs last period</span>
  </div>
</div>
```

### KPI Grid
```jsx
<div className="grid grid-cols-4 gap-4 mb-6">
  {metrics.map(m => <KPICard key={m.id} {...m} />)}
</div>
```

### Chart Row
```jsx
<div className="grid grid-cols-12 gap-4 mb-6">
  <div className="col-span-8 bg-white border border-slate-200 rounded-lg p-4 shadow-sm">
    {/* main chart */}
  </div>
  <div className="col-span-4 bg-white border border-slate-200 rounded-lg p-4 shadow-sm">
    {/* supporting chart */}
  </div>
</div>
```

### Data Table
```jsx
<div className="bg-white border border-slate-200 rounded-lg shadow-sm overflow-hidden">
  <div className="px-4 py-3 border-b border-slate-200 flex items-center justify-between">
    <h3 className="text-sm font-semibold text-slate-900">{title}</h3>
    <div className="flex items-center gap-2">{/* filters */}</div>
  </div>
  <table className="w-full text-sm">
    <thead className="bg-slate-50 text-slate-500 text-xs font-medium uppercase tracking-wider">
      <tr>
        {columns.map(col => <th className="px-4 py-3 text-left">{col}</th>)}
      </tr>
    </thead>
    <tbody className="divide-y divide-slate-100">
      {rows.map(row => (
        <tr className="hover:bg-slate-50 transition-colors">
          {/* cells */}
        </tr>
      ))}
    </tbody>
  </table>
</div>
```

### Sidebar Nav Item
```jsx
<a className="flex items-center gap-3 px-3 py-2 rounded-md text-sm font-medium text-slate-600 hover:bg-slate-50 hover:text-slate-900 transition-colors">
  <Icon className="w-4 h-4" />
  {label}
</a>
```

Active state:
```jsx
<a className="flex items-center gap-3 px-3 py-2 rounded-md text-sm font-medium bg-blue-50 text-blue-700">
  <Icon className="w-4 h-4" />
  {label}
</a>
```

---

## Motion & Transition

Dashboard motion is functional and unobtrusive — transitions confirm actions without distracting from data.

```
Duration scale:
  micro:   100ms   → Button press, badge update, status dot change
  fast:    150ms   → Table row hover, sidebar item hover
  normal:  200ms   → Dropdown, tooltip, popover
  medium:  280ms   → Side panel slide-in, modal appear
  slow:    400ms   → Chart animation (data load), skeleton shimmer

Easing:
  entrance:  cubic-bezier(0.0, 0.0, 0.2, 1)    /* ease-out — panels/modals sliding in */
  exit:      cubic-bezier(0.4, 0.0, 1, 1)       /* ease-in — panels sliding out */
  spring:    cubic-bezier(0.34, 1.56, 0.64, 1)  /* spring — button press, toggles */
  data:      ease-in-out                         /* for chart/data transitions */
```

Tailwind patterns:
```
Primary buttons:        transition-colors duration-[100ms] active:scale-[0.97]
Table rows:             transition-colors duration-[150ms]
Sidebar nav items:      transition-colors duration-[150ms]
KPI card hover:         transition-shadow duration-200 ease-out hover:shadow-md
Side panel slide-in:    duration-[280ms] ease-out (translateX)
Skeleton shimmer:       animate-pulse (Tailwind) or custom shimmer animation
```

---

## Tailwind Config Additions

```js
theme: {
  extend: {
    gridTemplateColumns: {
      '12': 'repeat(12, minmax(0, 1fr))',
    },
    colors: {
      dashboard: {
        sidebar: '#FFFFFF',
        page: '#F8FAFC',
        card: '#FFFFFF',
      }
    },
    width: {
      'sidebar': '256px',
      'sidebar-collapsed': '64px',
    },
    height: {
      'topbar': '56px',
    },
  }
}
```
