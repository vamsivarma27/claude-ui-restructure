<div align="center">

<img src="https://raw.githubusercontent.com/vamsivarma27/ui-restructure/main/ui-restructure/assets/banner.svg" alt="ui-restructure" width="100%" />

# ui-restructure

**The AI skill that solves UI lock-in.**
Redesign your entire UI from scratch — without deleting a single line of business logic.

[![SkillsMP](https://img.shields.io/badge/SkillsMP-approved-6366f1?style=flat-square&logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTYiIGhlaWdodD0iMTYiIHZpZXdCb3g9IjAgMCAxNiAxNiIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cmVjdCB3aWR0aD0iMTYiIGhlaWdodD0iMTYiIHJ4PSI0IiBmaWxsPSIjNjM2NmYxIi8+PC9zdmc+)](https://skillsmp.com)
[![Version](https://img.shields.io/badge/version-1.1.0-0ea5e9?style=flat-square)](https://github.com/vamsivarma27/ui-restructure/releases)
[![License](https://img.shields.io/badge/license-MIT-22c55e?style=flat-square)](LICENSE)
[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-f59e0b?style=flat-square)](https://agentskills.io)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-ec4899?style=flat-square)](https://github.com/vamsivarma27/ui-restructure/blob/main/.github/CONTRIBUTING.md)

Works with **Claude Code** · **Cursor** · **Windsurf** · **Cline** · **GitHub Copilot** · and any [Agent Skills](https://agentskills.io)-compatible tool.

</div>

---

## The Problem

Every developer who has used AI to build UI has hit this wall.

You ask AI to generate a dashboard. It builds one. You ship it. Six months later you want a completely different look — cleaner, more modern, more Apple-like. So you ask AI to redesign it.

**And it doesn't.**

It changes the colors. Maybe the fonts. But the layout is the same. The spacing is the same. The component hierarchy is the same. The div structure is the same.

Why? Because AI-generated UI creates **invisible lock-in**:

```
❌ Design tokens repeated in 40+ files
❌ Layout structure hardcoded inside JSX className props
❌ Spacing values scattered as Tailwind classes across every component
❌ Typography repeated individually — not from a system
❌ Container widths, grid columns, and flex directions fixed in the DOM
```

When you ask AI to "redesign" — it reads all this structure and uses it as the starting point. It can only drift slightly from what already exists. **It cannot escape it.**

The result: you end up with a UI that looks like a reskin, not a redesign. The bones are always the same. You're locked in.

---

## The Solution

`/ui-restructure` breaks the lock.

It reverse engineers your existing UI, extracts only the logical intent (what data is shown, what actions exist, what flows happen), strips away all the structural decisions, and rebuilds the UI from scratch using a fresh design system — **while keeping every hook, handler, API call, and data mapping completely untouched.**

```
Before                          After
──────────────────────────────────────────────────────
Old tokens everywhere      →    Fresh token system
Hardcoded Tailwind layout  →    New layout from scratch
Same div hierarchy         →    New component structure
Spacing hacked per file    →    Consistent spacing scale
Colors inline everywhere   →    Semantic color system
Developer-ordered UI       →    User-ordered UI (god mode)
```

Your business logic, your API integrations, your state management — **zero changes.**

---

## Install

```bash
# Via openskills (recommended)
npx openskills install vamsivarma27/ui-restructure

# Via agent-skills-cli
npx agent-skills-cli install vamsivarma27/ui-restructure

# Manual — copy into your project
cp -r ui-restructure/ .claude/skills/

# Or into your global Claude skills
cp -r ui-restructure/ ~/.claude/skills/
```

---

## Quick Start

```bash
# Full redesign — strips everything, rebuilds from scratch
/ui-restructure

# Apply Apple glassmorphism style
/ui-restructure --style apple

# Think like a user — redesign for user experience, not developer convenience
/ui-restructure --god-mode
```

---

## Commands

### `/ui-restructure`

Full redesign. Strips all layout structure, resets design tokens, rebuilds everything from scratch using the minimal style (default).

```bash
/ui-restructure
```

**What changes:** layout, spacing, typography, colors, tokens, component hierarchy
**What never changes:** hooks, API calls, handlers, data mapping, props, types

---

### `--style` — Apply a Named Design System

Choose from four production-grade style engines, each encoding a complete design system.

```bash
/ui-restructure --style apple
/ui-restructure --style linear
/ui-restructure --style minimal
/ui-restructure --style dashboard
```

| Style | Description | Best for |
|---|---|---|
| `apple` | Glassmorphism · soft shadows · large radius · frosted panels · airy spacing | Consumer apps, portfolio, SaaS marketing |
| `linear` | Compact · sharp borders · dark · dense · monochrome-first | Developer tools, productivity apps, internal tools |
| `minimal` | Clean · neutral · typography-first · no shadows · generous whitespace | Blogs, documentation, simple SaaS |
| `dashboard` | Sidebar-heavy · KPI card grids · data-dense · analytics-first layout | Admin panels, analytics, data platforms |

---

### `--mode` — Control What Gets Rebuilt

Use modes when you want a partial restructure — not everything at once.

```bash
/ui-restructure --mode layout    # Change structure only, keep your tokens
/ui-restructure --mode theme     # Change tokens/colors only, keep your layout
/ui-restructure --mode grid      # Convert list views to card grids
/ui-restructure --mode full      # Default — rebuild everything
```

| Mode | What changes | What stays |
|---|---|---|
| `full` | Everything (layout + tokens + hierarchy) | Logic only |
| `layout` | Flex/grid structure, containers, navigation position, column config | Tokens, colors, typography |
| `theme` | Design tokens, color system, typography scale, shadows, radius | All layout and structure |
| `grid` | Converts vertical lists and `ul/li` patterns into responsive card grids | Tokens, data, handlers |

---

### `--god-mode` — Redesign for the User, Not the Developer

This is the most powerful command in the skill.

```bash
/ui-restructure --god-mode
```

God Mode doesn't ask *"how should I arrange these components?"* — it asks *"what does the user actually need on this page?"*

It simulates **100 different user types** visiting every page:

- First-time visitors trying to understand what the product does
- Goal-oriented users looking for a specific action
- Returning users who want speed and no re-learning
- Mobile users with one thumb and interrupted attention
- Stressed users who will leave in 3 seconds if confused

For every page, it runs a **friction audit** — scoring each UI element on how important it is to the user versus how visually prominent it currently is. Elements that are critical but hidden get promoted. Elements that are rare but loud get demoted.

It then applies the **6 core UX principles**: Usability, Findability, Accessibility, Desirability, Learnability, and Credibility — and documents every violation and fix.

**By default, God Mode preserves your tokens.** It only changes hierarchy, placement, order, and visual weight.

```bash
/ui-restructure --god-mode                           # Redesign for users, keep tokens
/ui-restructure --god-mode --remove-tokens           # Redesign for users + reset tokens
/ui-restructure --god-mode --style apple --remove-tokens  # + apply Apple design system
```

---

### `--keep-tokens` — Rebuild Layout, Keep Your Colors

Already happy with your color scheme and spacing scale? Rebuild only the structural layout.

```bash
/ui-restructure --keep-tokens
/ui-restructure --style linear --keep-tokens
```

Tokens, colors, radius, shadows — untouched. Layout, flex/grid, component hierarchy, container structure — rebuilt from scratch.

---

### `--remove-tokens` — Explicitly Reset the Design System

Primarily useful alongside `--god-mode` (which keeps tokens by default).

```bash
/ui-restructure --god-mode --remove-tokens
```

Clears all values in `tokens.ts`, `theme.ts`, `tailwind.config.ts` theme section, and CSS variables in `globals.css` — then regenerates them from the active style engine.

---

### `--grid` — Force Card Grid Layout

Convert any page that uses a list, vertical stack, or `ul/li` pattern into a responsive card grid.

```bash
/ui-restructure --grid cards
```

All `.map()` loops, handlers, and data bindings are fully preserved. Only the container and item wrapper structure changes.

| Screen | Columns |
|---|---|
| Mobile | 1 |
| Tablet | 2 |
| Desktop | 3 |
| Wide | 4 |

---

### `--density` — Control Spacing Density

```bash
/ui-restructure --density compact      # 0.75x base — more content visible
/ui-restructure --density comfortable  # 1.0x base — default
/ui-restructure --density spacious     # 1.5x base — breathable, airy
```

Scales the entire spacing system proportionally across the rebuilt layout.

---

### `--prompt` — Custom Redesign in Natural Language

Describe exactly what you want. The skill combines your description with the style engine and mode constraints.

```bash
/ui-restructure --prompt "enterprise SaaS dashboard with a dark sidebar, floating KPI cards, and a data table below"
/ui-restructure --prompt "mobile-first onboarding flow with a progress bar and one action per step"
/ui-restructure --prompt "clean pricing page with a billing toggle and a feature comparison table"
```

---

## Combined Examples

```bash
# Apple style, layout only (keep your current tokens)
/ui-restructure --style apple --mode layout

# Linear style, compact, grid layout
/ui-restructure --style linear --density compact --mode grid

# Full rebuild with a custom prompt
/ui-restructure --style dashboard --prompt "analytics platform with collapsible sidebar"

# God Mode with Apple tokens reset
/ui-restructure --god-mode --style apple --remove-tokens

# Theme-only refresh — same layout, new colors
/ui-restructure --mode theme --style minimal
```

---

## What Gets Changed vs. Preserved

| Category | Status |
|---|---|
| `useState`, `useReducer`, `useEffect` | ✅ Preserved |
| Custom hooks | ✅ Preserved |
| API calls, `fetch`, `axios`, SWR, React Query | ✅ Preserved |
| Server actions (Next.js) | ✅ Preserved |
| Event handlers and callbacks | ✅ Preserved |
| Data mapping — `.map()`, `.filter()` | ✅ Preserved |
| Props and TypeScript interfaces | ✅ Preserved |
| Route handlers and API routes | ✅ Preserved |
| Layout wrappers and containers | 🔄 Rebuilt |
| `flex` / `grid` / `gap-*` classes | 🔄 Rebuilt |
| `p-*` / `m-*` / `space-*` spacing | 🔄 Rebuilt |
| Typography classes | 🔄 Rebuilt |
| Color and token classes | 🔄 Rebuilt |
| Design token files (`tokens.ts`, `theme.ts`) | 🔄 Rebuilt |
| `tailwind.config.ts` theme section | 🔄 Rebuilt |
| CSS variables in `globals.css` | 🔄 Rebuilt |

---

## Framework & Styling Support

| Framework | Supported |
|---|---|
| Next.js App Router | ✅ |
| Next.js Pages Router | ✅ |
| React (Vite) | ✅ |
| React (CRA) | ✅ |
| Vue 3 | ✅ |

| Styling System | Supported |
|---|---|
| Tailwind CSS | ✅ |
| CSS Modules | ✅ |
| styled-components | ✅ |
| shadcn/ui | ✅ |
| Plain CSS | ✅ |
| Inline styles | ✅ |

---

## AI Tool Support

This skill follows the [Agent Skills open standard](https://agentskills.io/specification). Install it once, use it in any compatible tool.

| Tool | Install path |
|---|---|
| Claude Code | `~/.claude/skills/ui-restructure/` or `.claude/skills/ui-restructure/` |
| Cursor | `~/.cursor/skills/ui-restructure/` or `.cursor/skills/ui-restructure/` |
| Windsurf | `~/.windsurf/skills/ui-restructure/` |
| Cline | `.cline/skills/ui-restructure/` |
| GitHub Copilot | `.github/skills/ui-restructure/` |

Use `npx openskills install vamsivarma27/ui-restructure` to auto-install to all detected tools at once.

---

## Skill Structure

```
ui-restructure/
├── SKILL.md                          ← Main skill (AI reads this to execute commands)
│
├── engines/                          ← Style engines
│   ├── apple.md                      ← Apple HIG — glassmorphism, SF Pro, airy spacing
│   ├── linear.md                     ← Linear.app — compact dark, Inter, dense layout
│   ├── minimal.md                    ← Clean neutral — typography-first, no shadows
│   └── dashboard.md                  ← Analytics — sidebar, KPI cards, 12-col grid
│
├── modes/                            ← Execution modes
│   ├── full.md                       ← Rebuild everything
│   ├── layout.md                     ← Structure only
│   ├── theme.md                      ← Tokens/colors only
│   ├── grid.md                       ← List → card grid
│   └── godmode.md                    ← User-first redesign (100-user mindset)
│
├── parser/
│   └── commands.md                   ← Argument parsing rules
│
├── prompts/
│   └── restructure-flow.md           ← Master execution flow
│
└── references/
    ├── ui-craft.md                   ← World-class frontend engineering (motion, typography,
    │                                    micro-interactions, iconography, accessibility)
    └── user-mindset.md               ← 10 Laws of User Behavior + 10 developer UX mistakes
```

---

## The Polish Standard

Every restructure runs a mandatory polish pass enforcing:

- **Motion** — spring physics, correct easing curves (ease-out in, ease-in out), GPU-only animations
- **Micro-interactions** — 5 states on every element (default, hover, focus, active, disabled)
- **Typography** — proportional line-height, 60–75 char line length, optical sizing
- **Iconography** — consistent library, optically aligned to adjacent text, correct stroke weight
- **Shadows** — two-layer ambient + key light system, 4 elevation levels
- **Spacing** — strict 8pt grid, inner radius = outer radius − padding
- **Accessibility** — `focus-visible` custom rings, WCAG AA contrast, 44px touch targets
- **States** — loading skeleton, empty state, error state designed on every list/async element

The bar is: **Would this pass on Stripe, Linear, Vercel, or apple.com?**

---

## Contributing

See [CONTRIBUTING.md](.github/CONTRIBUTING.md) for branch naming, commit format, and PR rules.

All contributions require:
- A valid `skills validate ./ui-restructure` result
- An updated `CHANGELOG.md` entry
- Review from `@vamsivarma27`

---

## License

MIT — see [LICENSE](LICENSE)

---

<div align="center">

**[View on SkillsMP](https://skillsmp.com) · [Report a bug](https://github.com/vamsivarma27/ui-restructure/issues) · [Request a feature](https://github.com/vamsivarma27/ui-restructure/issues)**

Made by [Vamsi Varma](https://github.com/vamsivarma27)

</div>
