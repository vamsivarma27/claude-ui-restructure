---
name: ui-restructure
description: |
  Solves UI lock-in. Reverse engineers your existing UI, strips all layout/token decisions, preserves every hook, handler, and API call untouched, then rebuilds the UI from scratch with a fresh design system. Use /ui-restructure to fully redesign, --style apple|linear|minimal|dashboard for named styles, --mode layout|theme|grid for partial rebuilds, --god-mode to redesign from the user's perspective using 100-user mindset simulation and 6 UX principles, --keep-tokens to preserve your token system, --prompt for custom redesigns. Applies mandatory world-class polish: spring motion, 5-state micro-interactions, 8pt spacing grid, WCAG accessibility. Supports Next.js, React, Vue 3, Tailwind, CSS Modules, styled-components, shadcn.
license: MIT
metadata:
  author: claude-ui-restructure
  version: "1.2.0"
  category: ui-design
  tags: ui,restructure,redesign,design-system,tailwind,nextjs,react,vue,tokens
argument-hint: "[--god-mode] [--style apple|linear|minimal|dashboard] [--mode full|layout|theme|grid] [--prompt 'custom UI'] [--keep-tokens] [--remove-tokens] [--grid cards] [--density compact]"
---

# Claude UI Restructure Skill

You are executing the `/restructure` skill. Your goal is to fully redesign the UI of this codebase without touching any business logic, API integrations, hooks, or data flow.

Follow the 10-step execution pipeline below. Read each step fully before acting on it.

---

## God Mode — Special Execution Path

If the command contains `--god-mode`, skip Steps 1–11 below and instead:

1. Read `references/user-mindset.md` fully — internalize the 10 Laws of User Behavior and all 10 developer mistakes
2. Read `modes/godmode.md` fully — this is your complete execution guide
3. Execute the 7-phase God Mode pipeline defined there
4. **Default: tokens are preserved.** Only add `--remove-tokens` if the user explicitly passed that flag.
5. If `--remove-tokens` is also passed, after completing God Mode, additionally run Step 7 (token reset) and Step 10 (token rebuild) from the standard pipeline using the style from `--style` (default: minimal).

God Mode does NOT combine with `--mode`. It replaces the mode entirely.
God Mode CAN combine with `--style` — the style engine defines visual language when `--remove-tokens` is also used.

---

## Behavior Rules (Non-Negotiable)

**NEVER modify:**
- Hooks (`useState`, `useEffect`, `useReducer`, custom hooks)
- Event handlers and callbacks
- API calls, fetch logic, server actions
- Data mapping and transformation
- Props interfaces and types
- Service layer files
- Database models
- Route handlers / API routes

**ONLY modify UI:**
- Layout wrappers and containers
- Spacing classes (`gap-*`, `p-*`, `m-*`, `space-*`)
- Typography classes
- Color/token usage
- Grid and flex structures
- Design token files

---

## Step 1 — Parse Command Arguments

Read the user's `/restructure` command and extract:

| Argument | Value |
|---|---|
| `--style` | `apple` / `linear` / `minimal` / `dashboard` / none |
| `--mode` | `full` / `layout` / `theme` / `grid` (default: `full`) |
| `--prompt` | Custom UI description string |
| `--keep-tokens` | Boolean flag — reuse existing tokens |
| `--grid` | `cards` / `list` |
| `--density` | `compact` / `comfortable` / `spacious` |

If no arguments, run full redesign.

Load the parser reference: `parser/commands.md`

---

## Step 2 — Detect Framework

Scan the project root for:

| Signal | Framework |
|---|---|
| `app/` directory + `layout.tsx` | Next.js App Router |
| `pages/` directory + `_app.tsx` | Next.js Pages Router |
| `vite.config.ts` + no `app/` dir | React (Vite) |
| `src/App.jsx` + `public/index.html` | React (CRA) |
| `*.vue` files + `vite.config.ts` | Vue 3 |

State detected framework before proceeding.

---

## Step 3 — Detect Styling System

Scan for:

| File/Signal | Styling System |
|---|---|
| `tailwind.config.*` | Tailwind CSS |
| `*.module.css` files | CSS Modules |
| `styled-components` in package.json | styled-components |
| `components.json` (shadcn config) | shadcn/ui |
| Inline `style={{}}` props | Inline styles |
| Plain `*.css` imports | Plain CSS |

Multiple systems may coexist. Record all detected systems.

---

## Step 4 — Scan UI Files

Scan these directories:
- `components/`
- `app/`
- `pages/`
- `src/`
- `layouts/`

For each UI file, build a UI model:

```
UI Model:
- Navigation type: sidebar | top-nav | bottom-nav
- Layout type: single-column | two-column | sidebar+content | full-width
- Content pattern: card-grid | list | table | dashboard | hero+sections
- Spacing scale: tight (4px base) | normal (8px base) | airy (16px base)
- Typography scale: extracted from heading classes
- Container width: max-w-* detected
- Density: compact | comfortable | spacious
- Token files: list of files
```

Output the UI model before proceeding.

---

## Step 5 — Identify Logic to Preserve

Before touching any file, scan every UI component and mark:

**PRESERVE (do not touch):**
```
// Data & state
const [items, setItems] = useState(...)
const { data } = useSWR(...)
useEffect(() => { fetchData() }, [])

// Handlers
const handleSubmit = async () => { ... }
const onDelete = (id) => { ... }

// Data mapping
{items.map(item => ( ... ))}

// API calls
const res = await fetch('/api/...')
const data = await res.json()

// Props
interface Props { userId: string; onSuccess: () => void }
```

**STRIP (remove layout bias):**
```
className="flex gap-4 p-4 max-w-7xl mx-auto"
className="grid grid-cols-3 gap-6"
className="text-sm font-medium text-gray-600"
className="rounded-lg shadow-md border border-gray-200"
```

---

## Step 6 — Strip UI Structure

For each UI component file:

1. Read the file
2. Identify all layout/spacing/typography classes
3. Remove them, leaving only semantic structure + logic

**Transformation example:**

Before:
```jsx
<div className="flex flex-col gap-6 p-8 max-w-7xl mx-auto">
  <div className="grid grid-cols-3 gap-4">
    {items.map(item => (
      <div className="bg-white rounded-xl shadow-md p-4 border border-gray-100">
        <h3 className="text-sm font-semibold text-gray-800">{item.name}</h3>
        <p className="text-xs text-gray-500 mt-1">{item.description}</p>
        <button onClick={() => handleDelete(item.id)}
                className="mt-3 px-3 py-1.5 text-xs bg-red-500 text-white rounded-md">
          Delete
        </button>
      </div>
    ))}
  </div>
</div>
```

After (logic preserved, layout stripped):
```jsx
<div>
  <div>
    {items.map(item => (
      <div key={item.id}>
        <h3>{item.name}</h3>
        <p>{item.description}</p>
        <button onClick={() => handleDelete(item.id)}>
          Delete
        </button>
      </div>
    ))}
  </div>
</div>
```

Repeat for all UI files. Do NOT strip logic. Do NOT strip semantic HTML tags. Do NOT strip key props.

---

## Step 7 — Reset Design Tokens

Find token files:
- `tokens.ts` / `tokens.js`
- `theme.ts` / `theme.js`
- `design-system.ts`
- `tailwind.config.ts` (theme.extend section)
- CSS variables in `globals.css` or `variables.css`

**If `--keep-tokens` flag is set:** Skip this step.

**Otherwise:**
- Delete or clear the existing token values
- Do NOT delete the file structure — just reset values
- Prepare for new token injection in Step 10

---

## Step 8 — Load Style Engine

Based on `--style` argument, load the corresponding engine file:

| Style | Engine File |
|---|---|
| `apple` | `engines/apple.md` |
| `linear` | `engines/linear.md` |
| `minimal` | `engines/minimal.md` |
| `dashboard` | `engines/dashboard.md` |
| none specified | Use `minimal` as default |

Read the engine file fully. It defines the complete design system to apply.

---

## Step 9 — Apply Mode

Based on `--mode` argument, load the corresponding mode file:

| Mode | File | What It Does |
|---|---|---|
| `full` | `modes/full.md` | Rebuild everything |
| `layout` | `modes/layout.md` | Change structure only, keep tokens |
| `theme` | `modes/theme.md` | Change tokens only, keep layout |
| `grid` | `modes/grid.md` | Convert lists to card grids |

Default mode is `full`.

If `--prompt` is provided, combine:
- Style engine definition
- User's prompt description
- Mode constraints

Then generate the UI accordingly.

---

## Step 10 — Rebuild UI

Using the stripped components from Step 6 + the style engine from Step 8 + the mode from Step 9:

1. **Regenerate design tokens** with new scale values
2. **Rebuild layout containers** with new structure
3. **Apply new typography scale** to all heading/text elements
4. **Apply new spacing scale** throughout
5. **Apply new color system** from style engine
6. **Rebuild component structures** using new grid/layout
7. **Write all modified files**

For each file modified, show a brief diff summary.

---

## Step 11 — Polish Pass (mandatory, never skip)

After rebuilding all UI files, read `references/ui-craft.md` and apply the full polish pass to every component:

**Motion:**
- Add `transition-all` or specific transition properties to every interactive element
- Apply correct easing curves (ease-out for entrance, ease-in for exit, spring for press)
- Add `active:scale-[0.97]` to all buttons
- Add `hover:translateY(-2px)` + shadow transition to all clickable cards
- Use Framer Motion spring presets if the project already uses Framer Motion

**States:**
- Every button must have: default, hover, focus-visible, active, disabled styles
- Every input must have: default, focus (ring + border), error, disabled styles
- Every list/table must have an empty state component
- Every async action must have a loading state (skeleton or spinner)
- Every error path must have an error state component

**Typography:**
- Verify line-height is proportional to font size (large text = tight, body = relaxed)
- Add `text-balance` to all headings to prevent orphaned words
- Verify max-w-prose on body copy blocks
- Apply negative letter-spacing to headings ≥ 24px

**Icons:**
- Verify one icon library is used consistently
- Verify icon size matches adjacent text size optically
- Add `aria-hidden="true"` to all decorative icons
- Add `aria-label` to all icon-only buttons

**Spacing:**
- Verify all spacing values are on 4/8px grid
- Verify inner border-radius = outer border-radius − padding on nested cards

**Accessibility:**
- Replace `outline: none` with `focus-visible` custom ring styles
- Verify all interactive elements are keyboard reachable
- Verify touch targets ≥ 44×44px (add padding if needed without changing visual size)

Run the complete checklist from `references/ui-craft.md` section 11 before finalizing.

After completing all files and polish pass, output:

```
✓ UI Restructure Complete
Framework: [detected framework]
Style: [applied style]
Mode: [applied mode]
Files modified: [count]
Logic preserved: ✓ (hooks, handlers, API calls untouched)
Tokens regenerated: ✓ / Tokens kept: ✓
Polish pass: ✓ (motion, states, typography, icons, accessibility)
```

---

## Quick Command Reference

| Command | Action |
|---|---|
| `/ui-restructure` | Full redesign, default minimal style, resets tokens |
| `/ui-restructure --god-mode` | **User-first redesign** — thinks like 100 users, keeps tokens, fixes UX hierarchy |
| `/ui-restructure --god-mode --remove-tokens` | God Mode + reset tokens with new design system |
| `/ui-restructure --god-mode --style apple` | God Mode + rebuild tokens in Apple style |
| `/ui-restructure --style apple` | Apple glassmorphism style |
| `/ui-restructure --style linear` | Linear compact style |
| `/ui-restructure --style minimal` | Clean minimal style |
| `/ui-restructure --style dashboard` | Data-dense dashboard style |
| `/ui-restructure --mode layout` | Change layout structure only, keep tokens |
| `/ui-restructure --mode theme` | Change tokens/colors only |
| `/ui-restructure --mode grid` | Convert lists to card grids |
| `/ui-restructure --keep-tokens` | Rebuild layout but keep existing tokens |
| `/ui-restructure --remove-tokens` | Explicitly reset all tokens (same as default without --keep-tokens) |
| `/ui-restructure --grid cards` | Force card grid layout |
| `/ui-restructure --density compact` | Apply compact spacing density |
| `/ui-restructure --prompt "..."` | Custom redesign prompt |

---

For detailed engine and mode specs, see the `engines/` and `modes/` directories.
For command parsing details, see `parser/commands.md`.
For the full restructure flow, see `prompts/restructure-flow.md`.
