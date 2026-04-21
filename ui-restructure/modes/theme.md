# Theme Mode

Change design tokens and color system only. Layout structure is preserved.

---

## What Theme Mode Does

Theme mode runs a **partial pipeline**:

1. ✓ Detect framework
2. ✓ Detect styling system
3. ✓ Scan token files
4. ✗ Skip layout scan (layout preserved)
5. ✗ Skip structural strip
6. ✓ Reset design token files
7. ✓ Load style engine (color/typography/shadow rules only)
8. ✓ Rebuild token files
9. ✓ Update color classes in components

---

## Scope

Theme mode touches:

| Category | Action |
|---|---|
| `tokens.ts` / `theme.ts` | Reset + rebuilt |
| `tailwind.config.ts` theme section | Updated |
| CSS variables in `globals.css` | Reset + rebuilt |
| Color classes (`bg-*`, `text-*`, `border-*`) | Updated |
| Typography scale (size/weight) | Updated |
| Shadow scale | Updated |
| Radius scale | Updated |

Theme mode does NOT touch:

| Category | Action |
|---|---|
| `flex` / `grid` layout | Untouched |
| Container widths | Untouched |
| Navigation structure | Untouched |
| Column configurations | Untouched |
| Spacing scale (gap/p/m) | Untouched |
| Business logic | Untouched |
| Hooks and handlers | Untouched |

---

## When to Use Theme Mode

Use `--mode theme` when:

- The layout is already good, but colors/fonts feel wrong
- You want to switch from light to dark theme
- You want to apply a brand color palette
- You want to change the typography scale
- You want a fresh token set without restructuring the layout

---

## Token Replacement Strategy

### For Tailwind projects:

1. Open `tailwind.config.ts`
2. Clear `theme.extend.colors`, `theme.extend.fontFamily`, `theme.extend.spacing`, `theme.extend.borderRadius`, `theme.extend.boxShadow`
3. Replace with style engine token values
4. **If a separate `tokens.ts` / `theme.ts` / `design-system.ts` also exists alongside `tailwind.config.ts`:** update that file too — replace its color, spacing, radius, and shadow values with the style engine values. Both files must stay in sync. Do NOT skip token files just because `tailwind.config.ts` was already updated.
5. Scan all component files for hardcoded color classes (e.g., `text-gray-600`, `bg-white`, `border-gray-200`)
6. Replace with new semantic token classes

### For CSS variable projects:

1. Open `globals.css` or `variables.css`
2. Reset all `--color-*`, `--font-*`, `--radius-*`, `--shadow-*` variables
3. Replace with style engine values
4. Verify component files reference variables (not hardcoded values)

### For styled-components / CSS modules:

1. Find the theme provider or central theme object
2. Replace the token object values
3. Component styles that reference theme tokens will update automatically

---

## Output Format

```
✓ Theme Restructure Complete (theme mode)

Framework:        Next.js App Router
Style applied:    Apple
Layout:           Preserved

Token files updated:
  ✓ tailwind.config.ts   (colors, typography, radius, shadows)
  ✓ app/globals.css      (CSS variables)

Color classes updated in:
  ✓ components/Button.tsx
  ✓ components/Card.tsx
  ✓ components/Navbar.tsx
  ✓ [N more components]

Untouched:
  → All layout/flex/grid classes (preserved)
  → All spacing/gap classes (preserved)
  → All business logic (preserved)
```
