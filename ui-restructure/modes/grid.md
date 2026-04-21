# Grid Mode

Convert list-based layouts to card grid layouts. Preserves data mapping and logic.

---

## What Grid Mode Does

Grid mode runs a **targeted pipeline**:

1. ✓ Detect framework
2. ✓ Scan for list patterns
3. ✓ Identify items rendered in vertical lists
4. ✓ Preserve all map/loop logic
5. ✓ Convert list container to grid container
6. ✓ Convert list item to card structure
7. ✓ Apply card design from style engine

---

## Scope

Grid mode touches:

| Category | Action |
|---|---|
| List containers (`flex flex-col gap-*`) | Converted to grid |
| `<ul>` / `<ol>` wrappers | Converted to grid div |
| `<li>` items | Converted to card divs |
| Horizontal list rows | Converted to cards |
| Table-like custom rows | Converted to cards |

Grid mode does NOT touch:

| Category | Action |
|---|---|
| `.map()` logic | Untouched |
| Data passed to items | Untouched |
| Item event handlers | Untouched |
| Actual table elements | Untouched (tables stay tables) |
| Business logic | Untouched |
| Token files | Untouched |

---

## List Detection Patterns

Scan for these patterns to identify convertible lists:

```jsx
// Pattern 1: flex-col container with mapped items
<div className="flex flex-col gap-*">
  {items.map(item => <Row key={item.id} {...item} />)}
</div>

// Pattern 2: explicit ul/li
<ul className="space-y-*">
  {items.map(item => <li key={item.id}>...</li>)}
</ul>

// Pattern 3: vertical stack
<div className="space-y-*">
  {items.map(item => (...))}
</div>
```

---

## Grid Conversion

### Default: Cards grid

Convert to responsive card grid:

```jsx
// Before
<div className="flex flex-col gap-4">
  {items.map(item => (
    <div className="flex items-center p-3 border-b">
      <span>{item.name}</span>
      <button onClick={() => handleClick(item.id)}>View</button>
    </div>
  ))}
</div>

// After (logic fully preserved)
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
  {items.map(item => (
    <div className="bg-white border border-neutral-200 rounded-lg p-4 hover:shadow-md transition-shadow">
      <span>{item.name}</span>
      <button onClick={() => handleClick(item.id)}>View</button>
    </div>
  ))}
</div>
```

### With `--grid cards`:

2–4 column responsive grid with styled cards.

### With `--grid list`:

**Reverse conversion: grid → list.** When `--grid list` is passed, grid mode reverses direction — it detects existing card grids and converts them to vertical list layouts.

**Detection patterns for existing grids:**
```jsx
// Pattern 1: CSS grid container
<div className="grid grid-cols-* gap-*">
  {items.map(item => ( ... ))}
</div>

// Pattern 2: flex wrap
<div className="flex flex-wrap gap-*">
  {items.map(item => ( ... ))}
</div>
```

**Conversion — grid to list:**
```jsx
// Before (grid)
<div className="grid grid-cols-3 gap-6">
  {items.map(item => (
    <div key={item.id} className="bg-white rounded-xl p-4 shadow">
      <h3 className="font-semibold">{item.name}</h3>
      <button onClick={() => handleAction(item.id)}>Action</button>
    </div>
  ))}
</div>

// After (list — logic fully preserved)
<div className="flex flex-col gap-3">
  {items.map(item => (
    <div key={item.id} className="flex items-center justify-between p-4 bg-white border border-neutral-200 rounded-lg hover:bg-neutral-50 transition-colors">
      <span className="text-sm font-medium text-neutral-900">{item.name}</span>
      <button onClick={() => handleAction(item.id)}>Action</button>
    </div>
  ))}
</div>
```

**Rules for --grid list conversion:**
- Remove `grid grid-cols-*` classes — replace with `flex flex-col`
- Remove `gap-*` on the grid — replace with `gap-2` or `gap-3` (list row spacing)
- Convert card-style items to horizontal list rows (flex items-center justify-between)
- Preserve all `.map()` loops, event handlers, and data props untouched
- Preserve `key` props
- Do NOT convert actual `<table>` / `<tbody>` / `<tr>` elements — only grid/flex-wrap containers

### With `--density compact`:

Tighter grid (4–5 columns), smaller card padding.

---

## Grid Column Rules

| Screen | Columns (default) | Columns (compact) |
|---|---|---|
| Mobile (< 640px) | 1 | 2 |
| Tablet (640–1024px) | 2 | 3 |
| Desktop (1024–1280px) | 3 | 4 |
| Wide (1280px+) | 4 | 5 |

---

## Card Structure

When converting a list item to a card:

1. Identify the data displayed (name, description, status, image, etc.)
2. Map data to card regions:
   - **Header:** Primary identifier (name, title, ID)
   - **Body:** Secondary data (description, metadata)
   - **Footer:** Actions (buttons, links)
3. Apply card styling from the active style engine

---

## Output Format

```
✓ Grid Conversion Complete (grid mode)

Files converted:
  ✓ app/products/page.tsx     (list → 3-col card grid)
  ✓ components/UserList.tsx   (ul/li → card grid)
  ✓ app/orders/page.tsx       (vertical stack → 2-col card grid)

Logic preserved in all conversions:
  ✓ .map() loops untouched
  ✓ onClick handlers untouched
  ✓ Item data props untouched

Grid applied:
  Columns: 1 / 2 / 3 (responsive)
  Card style: minimal border
  Density: comfortable
```
