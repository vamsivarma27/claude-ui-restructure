# claude-ui-restructure

> A Claude Code skill that solves UI lock-in — fully redesign your UI without deleting your project.

## The Problem

When AI generates UI:
- Design tokens get created and repeated everywhere
- Layout structure gets embedded in JSX
- Spacing gets hardcoded in Tailwind classes
- Typography gets scattered across components

Later when you try to redesign: AI reuses old layout, old spacing, old component hierarchy — and only changes colors. **This is UI lock-in.**

## The Solution

`/restructure` reverse engineers your existing UI, extracts layout intent, preserves all business logic, strips the UI structure, and rebuilds from scratch with a new design system.

Your hooks, API calls, handlers, data mappings — untouched.

---

## Install

```bash
# Using openskills
npx openskills install vamsivarma27/ui-restructure

# Manual (copy to project)
cp -r claude-ui-restructure .claude/skills/
```

---

## Usage

```bash
# Full redesign (default minimal style)
/restructure

# Apply Apple glassmorphism style
/restructure --style apple

# Apply Linear compact dark style
/restructure --style linear

# Apply dashboard analytics style
/restructure --style dashboard

# Change layout only, keep tokens
/restructure --mode layout

# Change tokens/colors only
/restructure --mode theme

# Convert lists to card grids
/restructure --mode grid

# Custom redesign prompt
/restructure --prompt "enterprise SaaS dashboard with dark sidebar"

# Keep existing tokens, rebuild layout
/restructure --keep-tokens

# Compact density
/restructure --density compact

# Combined
/restructure --style linear --mode layout --density compact
```

---

## What Gets Changed vs. Preserved

| What | Action |
|---|---|
| Layout wrappers, flex/grid | **Rebuilt** |
| Spacing classes (`gap-*`, `p-*`, `m-*`) | **Rebuilt** |
| Typography classes | **Rebuilt** |
| Color/token classes | **Rebuilt** |
| Design token files | **Reset + Regenerated** |
| `useState` / `useEffect` / hooks | **Preserved** |
| API calls, fetch logic | **Preserved** |
| Event handlers | **Preserved** |
| Data mapping (`.map()`) | **Preserved** |
| Props and types | **Preserved** |
| Route handlers / API routes | **Preserved** |

---

## Style Engines

| Style | Description |
|---|---|
| `apple` | Glassmorphism, large radius, airy spacing, frosted panels |
| `linear` | Compact, sharp borders, dark, data-dense |
| `minimal` | Clean, neutral, typography-first, no shadows |
| `dashboard` | Sidebar-heavy, card grids, KPI-first, analytics layout |

---

## Framework Support

- Next.js App Router
- Next.js Pages Router
- React (Vite)
- React (CRA)
- Vue 3

## Styling Support

- Tailwind CSS
- CSS Modules
- styled-components
- shadcn/ui
- Plain CSS

---

## Skill Structure

```
claude-ui-restructure/
├── SKILL.md                    # Main skill definition (Claude reads this)
├── engines/
│   ├── apple.md                # Apple HIG style engine
│   ├── linear.md               # Linear.app style engine
│   ├── minimal.md              # Minimal style engine
│   └── dashboard.md            # Dashboard/analytics style engine
├── modes/
│   ├── full.md                 # Full rebuild mode
│   ├── layout.md               # Layout-only mode
│   ├── theme.md                # Theme/tokens-only mode
│   └── grid.md                 # List-to-grid conversion mode
├── parser/
│   └── commands.md             # Command argument parser
└── prompts/
    └── restructure-flow.md     # Master execution flow
```

---

## License

MIT — see [LICENSE](LICENSE)

---

## Published on

- [SkillsMP](https://skillsmp.com) — search `claude-ui-restructure`
