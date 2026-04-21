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

### Next.js App Router — Server vs Client Component Rules (Non-Negotiable)

When the framework is **Next.js App Router**, every component file is either a Server Component or a Client Component. The skill MUST respect this distinction at all times.

**How to identify each type:**
- **Client Component:** File begins with `"use client"` directive (first line, before any imports)
- **Server Component:** File has NO `"use client"` directive — it may use `async`/`await`, call `db.query(...)`, call `getServerSession()`, or other server-side APIs

**Rules — never violate these:**

1. **NEVER add `"use client"` to a Server Component.** Adding `"use client"` to a Server Component would break the application — it cannot use server-side APIs (DB, session, server-only imports) in a Client Component.

2. **NEVER remove `"use client"` from a Client Component.** Removing the directive would cause a build error — hooks (`useState`, `useCallback`, etc.) and event handlers (`onClick`, `onMouseEnter`, etc.) are not allowed in Server Components.

3. **NEVER modify the `"use client"` directive itself** — do not move it, rename it, or alter its position at the top of the file.

4. **Server-side data access is preserved like hooks:** Calls to `db.query(...)`, `getServerSession()`, `prisma.findMany(...)`, and other server-side APIs in Server Components fall under the NEVER modify rule (same protection as `fetch` and API calls). Treat them as protected logic regardless of whether they look like "API calls" or "database calls."

5. **Async Server Components:** A component defined as `export default async function MyComponent()` with no `"use client"` is a Server Component. Do NOT add `"use client"` to make it non-async or to enable hooks.

**In Step 4 (Scan UI Files):** When scanning App Router files, record for each file:
- Is it a Client Component? (starts with `"use client"`)
- Is it a Server Component? (no `"use client"` directive)

**In Step 6 (Strip UI Structure):** Strip layout/styling classes as normal. But for both Server and Client Components:
- Preserve the `"use client"` directive exactly as-is at the top of Client Component files
- Do NOT add `"use client"` to Server Component files during stripping

**In Step 10 (Rebuild UI):** Rebuild layout and classes as normal. The `"use client"` status of each file does not change during rebuild — only layout/styling classes change.

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

Scan the project root and `src/` directory for these signals:

| Signal | Framework |
|---|---|
| `app/` directory (at root) + `layout.tsx` inside it | Next.js App Router |
| `src/app/` directory + `layout.tsx` inside `src/app/` | Next.js App Router (src/ layout convention) |
| `pages/` directory + `_app.tsx` | Next.js Pages Router |
| `vite.config.ts` + no `app/` dir | React (Vite) |
| `src/App.jsx` + `public/index.html` | React (CRA) |
| `*.vue` files + `vite.config.ts` | Vue 3 |

**`src/app/` pattern (Next.js App Router inside src/):** Many Next.js projects place the App Router directory inside `src/` — i.e., `src/app/layout.tsx` instead of `app/layout.tsx` at the project root. If `app/` is not present at the root but `src/app/` exists with a `layout.tsx`, detect this as **Next.js App Router**. In Step 4, scan `src/app/` as the App Router directory (instead of `app/`). All file exclusion rules and Server/Client Component rules apply identically. The scan root for App Router files is `src/app/` in this case.

**Conflict resolution — when multiple signals match:**
- If both `app/` (with `layout.tsx`) and `pages/` exist simultaneously: **App Router wins.** This is the Next.js 13+ hybrid convention. Treat the project as Next.js App Router and skip Pages Router scanning.
- If `src/app/` (with `layout.tsx`) and `pages/` exist simultaneously: **App Router wins** — same rule applies.
- If `*.vue` files exist alongside `app/` or `pages/`: Vue 3 wins (the `.vue` extension is a definitive signal).

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

**Framer Motion detection (animation library — separate from styling system):**

Also check for Framer Motion in the project:
- `framer-motion` in `package.json` dependencies or devDependencies
- `import { motion } from 'framer-motion'` in any component file

If Framer Motion is detected:
- Record `framer-motion: true` alongside the styling system
- Step 11 (Polish Pass) MUST use Framer Motion variants instead of CSS transition classes for all motion effects
- Note in your reasoning: "Framer Motion detected — Step 11 will use motion variants"

---

## Step 4 — Scan UI Files

Scan directories based on the detected framework (Step 2):

**Always scan (recursively — all nested subdirectories included):**
- `components/`
- `src/`
- `layouts/`

**Scan conditionally (recursively — all nested subdirectories included):**
- `app/` — only if framework is Next.js App Router (or if both app/ and pages/ exist: App Router wins, scan only `app/`)
- `pages/` — only if framework is Next.js Pages Router (not when App Router is detected)

**Scanning is always recursive.** When a directory is listed for scanning, scan ALL files in ALL subdirectories at all depths — not just the top level. For example, `app/dashboard/analytics/components/ReportCard.tsx` (depth 4) and `app/dashboard/analytics/components/charts/LineChart.tsx` (depth 5) are both in scope when `app/` is scanned.

**Route group directories** (Next.js App Router convention): directories whose names are wrapped in parentheses — e.g., `(auth)/`, `(marketing)/`, `(dashboard)/` — are route groups. They do NOT affect the URL path but they DO contain real component and page files. Treat them as regular directories during recursive scanning. Example: `app/(auth)/login/components/LoginForm.tsx` is in scope and must be processed.

If both `app/` and `pages/` exist and App Router was detected: scan `app/` only (recursively). Do NOT scan or modify files in `pages/`.

**File exclusion rules — NEVER scan or modify these files:**

Before processing any file in a scanned directory, check for these exclusion patterns. Skip any file that matches:

1. **Barrel files** — files that contain ONLY re-export statements and no JSX/UI rendering:
   - Files whose entire content consists of `export { ... } from '...'`, `export * from '...'`, `export type { ... } from '...'`, or `export default ... from '...'` lines
   - Typically named `index.ts`, `index.tsx`, `index.js`, but can be any name
   - Detection: if a file has no JSX (`<` tags), no `className`, no `style=`, and only export statements — it is a barrel file. Skip it.

2. **Test files** — files that contain test assertions and mock implementations:
   - `*.test.tsx` / `*.test.ts` / `*.test.jsx` / `*.test.js`
   - `*.spec.tsx` / `*.spec.ts` / `*.spec.jsx` / `*.spec.js`
   - `__tests__/` directory contents
   - Files with `describe(`, `it(`, `test(`, `expect(` as primary content patterns
   - NEVER modify test files — they are not UI files and contain logic under test.

2b. **Storybook story files** — files that define Storybook stories for component documentation/testing:
   - `*.stories.tsx` / `*.stories.ts` / `*.stories.jsx` / `*.stories.js`
   - `*.stories.mdx` — MDX story format
   - `*.story.tsx` / `*.story.ts` / `*.story.jsx` / `*.story.js` — alternate story extension
   - Detection: if the filename contains `.stories.` or `.story.` before the file extension — it is a Storybook file. Skip it.
   - NEVER modify Storybook files — they are documentation/test fixtures, not UI component files. Modifying them would corrupt story args, meta definitions, and story exports, breaking the Storybook build.

3. **Type definition files** — not UI files:
   - `*.d.ts` files

4. **Server Action files** — files with `"use server"` directive are pure server logic, no UI:
   - Files whose first line is `"use server"` (the Next.js Server Action directive)
   - These files contain only server-side business logic (database calls, `revalidatePath`, form processing)
   - Detection: if the first non-empty line of a file is `"use server"` — it is a Server Action file. Skip it.
   - NEVER scan or modify Server Action files — they contain no UI classes and fall under the "server actions" NEVER modify rule.
   - Note: Server Components (no directive at all) are NOT excluded from scanning — they may have JSX and layout classes that need rebuilding. Only files with `"use server"` as their first directive are excluded.

5. **API Route files** — Next.js App Router API routes are pure server handlers, no UI:
   - Files named `route.ts` / `route.js` / `route.tsx` / `route.jsx` in any directory
   - These files export `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `HEAD`, `OPTIONS` handler functions
   - Detection: if the filename is `route.ts` (or `route.js` / `route.tsx` / `route.jsx`) — it is an API route file. Skip it.
   - NEVER scan or modify API route files — they contain no JSX or UI classes and fall under the "route handlers / API routes" NEVER modify rule.

6. **Service layer directories** — non-UI subdirectories inside scanned parent directories:
   - When scanning `src/` or `app/`, skip files inside these subdirectories: `lib/`, `utils/`, `services/`, `helpers/`, `hooks/` (custom hook files — not UI components), `store/`, `context/`, `config/`, `db/`, `models/`, `types/`
   - These directories contain business logic, utilities, and data access code — not UI components
   - Detection: check the file's directory path. If it contains any of these non-UI segment names between the scan root and the file, skip it.
   - Exception: `hooks/` files that contain JSX (render hooks that return components) — scan these. Pure hook files (no JSX) — skip.
   - Note: `src/components/` and `app/components/` (explicit components subdirectories) are always scanned regardless of this rule.

7. **Middleware files** — Next.js middleware intercepts requests at the edge; it contains routing/auth logic, no JSX:
   - Files named `middleware.ts` / `middleware.js` / `middleware.tsx` / `middleware.jsx` in any directory
   - These files export a `middleware(request: NextRequest)` function and a `config` object — pure routing logic, no UI classes
   - Detection: if the filename is `middleware.ts` (or `middleware.js` / `middleware.tsx` / `middleware.jsx`) — it is a middleware file. Skip it.
   - NEVER scan or modify middleware files — they contain no JSX or UI classes. This applies to `middleware.ts` at the project root AND `src/middleware.ts` inside the `src/` scan directory.

8. **Instrumentation files** — Next.js instrumentation hooks for observability (OpenTelemetry, Sentry, Datadog); contains no JSX:
   - Files named `instrumentation.ts` / `instrumentation.js` / `instrumentation.node.ts` / `instrumentation.node.js` in any directory
   - These files export a `register()` function that initializes monitoring/tracing — pure server-side setup, no UI classes
   - Detection: if the filename is `instrumentation.ts`, `instrumentation.js`, `instrumentation.node.ts`, or `instrumentation.node.js` — it is an instrumentation file. Skip it.
   - NEVER scan or modify instrumentation files — they contain no JSX or UI classes. This applies to root-level `instrumentation.ts` AND `src/instrumentation.ts` inside the `src/` scan directory.

Applying these exclusions prevents: (1) barrel files being accidentally modified and breaking re-export paths, (2) test files being treated as UI components, (2b) Storybook story files being treated as UI components, (3) TypeScript declarations being touched, (4) Server Action files being scanned despite containing no UI, (5) API route files being scanned despite containing no UI, (6) service layer utility files being scanned and potentially misidentified as UI components, (7) middleware files being scanned despite containing no UI, (8) instrumentation files being scanned despite containing no UI.

For each UI file (after exclusions), build a UI model:

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

**Non-className HTML attributes — NEVER strip (Non-Negotiable):**

The strip pass targets ONLY the `className` attribute (and `class=` in Vue templates). Every other HTML and JSX attribute on every element MUST be left completely untouched. The following attribute categories are explicitly protected — do not remove, rename, or alter them under any circumstances:

- **Element identity:** `id`, `name`
- **Element type/role:** `type`, `role`
- **Form values:** `value`, `defaultValue`, `checked`, `defaultChecked`
- **Form constraints:** `required`, `disabled`, `readOnly`, `maxLength`, `minLength`, `min`, `max`, `step`, `pattern`, `multiple`
- **Input hints:** `placeholder`, `autoComplete`, `spellCheck`, `autoFocus`, `autoCapitalize`
- **Accessibility:** `aria-*` (all ARIA attributes), `htmlFor`, `tabIndex`
- **Data attributes:** `data-*` (all data attributes — `data-testid`, `data-analytics`, etc.)
- **React-specific:** `key`, `ref`, `suppressHydrationWarning`, `suppressContentEditableWarning`
- **Event handlers (all):** `onClick`, `onKeyDown`, `onKeyUp`, `onKeyPress`, `onFocus`, `onBlur`, `onChange`, `onSubmit`, `onMouseEnter`, `onMouseLeave`, `onMouseDown`, `onMouseUp`, `onTouchStart`, `onTouchEnd`, and ALL other `on*` event handlers — these are logic, not styling
- **Media/link:** `src`, `href`, `alt`, `target`, `rel`, `download`, `action`, `method`
- **Content:** `dangerouslySetInnerHTML`, `style` (inline style prop — handled separately in Inline styles section)
- **HTML attributes:** Any attribute not listed in the "strip" category below is preserved as-is

**What you DO strip:** ONLY the string value(s) of `className="..."` attributes that contain Tailwind utility classes for layout, spacing, typography, and color. Nothing else.

If you are uncertain whether an attribute is a layout class or a logic attribute — PRESERVE it. The only attributes that change are `className` (its value is rebuilt) and `class`/`:class` in Vue templates.

**Vue SFC handling (when framework is Vue 3):**

Vue Single File Components have three blocks — handle each differently:

- **`<template>` block:** Strip `class="..."` and `:class="..."` attributes that contain only layout/spacing/typography classes. Preserve `:class` expressions that contain conditional logic (e.g., `:class="isActive ? 'active' : ''"` — strip the class values but preserve the ternary structure). Preserve `v-for`, `:key`, `@click`, `v-if`, and all other Vue directives and bindings.
- **`<script setup>` block (and `<script>` block):** Do NOT modify. This contains all component logic — `defineProps`, `defineEmits`, `computed`, `ref`, `reactive`, event handlers. It is fully protected under the PRESERVE rules.
- **`<style scoped>` block (and `<style>` block):** **Preserve as-is.** Do NOT strip, reset, or modify scoped styles. They are component-specific styles, not token files. Token files (Step 7) are separate global token sources.

Template literal classNames with embedded logic (e.g., `` className={`base-classes ${condition ? 'a' : 'b'}`} ``): strip the static CSS class strings but preserve the ternary/conditional logic and the template literal structure itself.

**`cn()` and `clsx()` utility wrapper handling:**

Many real-world Tailwind projects pass `className` through a `cn()` function (from shadcn/ui: `import { cn } from '@/lib/utils'`) or `clsx()` function (`import { clsx } from 'clsx'`). These are utility wrappers that merge and deduplicate Tailwind class strings. The `className` value is a **function call expression**, not a plain string literal.

Rules for handling `cn()` and `clsx()` wrappers:

1. **NEVER strip the wrapper function call.** `className={cn(...)}` must remain `className={cn(...)}` after strip. Do NOT replace it with `className=""` or a plain string.
2. **NEVER strip the `clsx()` wrapper.** Same rule: `className={clsx(...)}` remains `className={clsx(...)}`.
3. **Preserve the import statements** for `cn` and `clsx` — `import { cn } from '@/lib/utils'` and `import { clsx } from 'clsx'` are NOT layout classes and must remain.
4. **Strip the layout/spacing/typography class string values INSIDE the wrapper** — treat the arguments to `cn()` or `clsx()` the same as you would a plain `className` string: strip the Tailwind utility strings, but preserve conditional logic, ternaries, and object syntax (e.g., `{ "opacity-50": dismissed }` — preserve the object structure, update the class value if it is a layout class).
5. **Preserve all conditional logic inside the wrapper** — `variant === 'error' && "bg-red-50 border-red-200"` strips the class string values but preserves `variant === 'error' &&`.

Example:
```jsx
// Before (cn() wrapper)
<div className={cn(
  "flex items-center gap-3 px-4 py-3 rounded-lg border",
  variant === 'error' && "bg-red-50 border-red-200 text-red-800"
)}>
  <button className={clsx("ml-auto p-1 rounded hover:bg-black/10", { "opacity-50": dismissed })}>

// After strip (wrapper preserved, class strings cleared, conditional logic preserved):
<div className={cn(
  "",
  variant === 'error' && ""
)}>
  <button className={clsx("", { "opacity-50": dismissed })}>

// After rebuild (wrapper preserved, new style engine classes applied):
<div className={cn(
  "flex items-center gap-2 px-5 py-4 rounded-xl border",
  variant === 'error' && "bg-red-50/80 border-red-200 text-red-900"
)}>
  <button className={clsx("ml-auto p-1.5 rounded-lg hover:bg-black/10", { "opacity-50": dismissed })}>
```

**Inline styles handling (when styling system is "Inline styles"):**

When Step 3 detected `style={{}}` props as the styling system, strip inline style property values during this step — but preserve the `style={{}}` prop structure and any dynamic/conditional values that reference state or props.

- **Static visual values** (colors, padding, borderRadius, fontSize, fontWeight, background): clear the value or set to empty string — these will be replaced in Step 10.
- **Dynamic values** that reference component state or props (e.g., `background: hovered ? '#f5f5f5' : '#fff'`): preserve the conditional logic but the actual color strings will be updated in Step 10.
- **Event handlers** (`onMouseEnter`, `onMouseLeave`, etc.): NEVER touch — these are logic, not style.

Example:
```jsx
// Before (inline styles)
<div
  style={{ padding: '16px', background: hovered ? '#f5f5f5' : '#fff', borderRadius: '8px' }}
  onMouseEnter={() => setHovered(true)}
  onMouseLeave={() => setHovered(false)}
>
  <h3 style={{ fontSize: '14px', fontWeight: 600 }}>{item.name}</h3>
</div>

// After strip (logic/structure preserved, visual values cleared for rebuild):
<div
  style={{ padding: '', background: hovered ? '' : '', borderRadius: '' }}
  onMouseEnter={() => setHovered(true)}
  onMouseLeave={() => setHovered(false)}
>
  <h3 style={{ fontSize: '', fontWeight: 600 }}>{item.name}</h3>
</div>
```

**Plain CSS handling (when styling system is "Plain CSS"):**

When Step 3 detected plain `*.css` imports (e.g., `import './styles/card.css'`), the JSX and CSS files require different treatment during the strip step.

**In JSX/TSX component files:**
- **DO NOT strip `className` values** — for Plain CSS projects, `className="card"` is not a utility class; it is a selector reference that matches a rule in the `.css` file. Stripping the className would break the CSS link. Preserve all `className` attributes exactly as-is.
- **DO NOT remove CSS import statements** — `import './styles/card.css'` (or similar path) must remain. It is not a layout class; removing it would break all styles for the component.
- **DO strip** any Tailwind utility classes mixed alongside plain CSS class names (if both systems coexist). If the project is purely plain CSS with no Tailwind, do not strip any classNames.

**In CSS files (`*.css`):**
- Do NOT strip anything during the strip step. CSS files are rebuilt in Step 10.
- CSS files are NOT token files — do not process them in Step 7.

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

**tailwind.config.ts / tailwind.config.js — Protected Fields (Non-Negotiable):**

When resetting tokens in `tailwind.config.ts` or `tailwind.config.js`, ONLY update `theme.extend` values (colors, spacing, borderRadius, fontFamily, boxShadow, etc.). NEVER touch:

1. **`content` array** — tells Tailwind which files to scan for class names. Removing or changing this will cause Tailwind to stop generating CSS for the project's files. PRESERVE exactly.
2. **`plugins` array** — contains installed Tailwind plugins (e.g., `forms`, `typography`, `animate`). PRESERVE exactly.
3. **`safelist` array** — contains class names that must always be generated even if not detected in content files. PRESERVE exactly.
4. **`darkMode` setting** — controls dark mode strategy (`'class'`, `'media'`, `['class', '[data-theme="dark"]']`). PRESERVE exactly.
5. **Import statements** — `import forms from '@tailwindcss/forms'` and similar plugin imports at the top of the file MUST remain.
6. **`presets` array** — if present, PRESERVE exactly.

**What changes in tailwind.config during Step 7:**
- Only values inside `theme.extend` are cleared (colors, borderRadius, fontFamily, boxShadow, spacing)

**What changes in tailwind.config during Step 10 (rebuild):**
- Only values inside `theme.extend` are replaced with style engine token values

**Example — correct tailwind.config handling:**
```ts
// PRESERVE: imports
import type { Config } from 'tailwindcss'
import forms from '@tailwindcss/forms'     // ← PRESERVE this import

const config: Config = {
  darkMode: 'class',                        // ← PRESERVE — never touch
  content: [                                // ← PRESERVE entire array — never touch
    './app/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  safelist: ['bg-red-500', 'text-yellow-600'], // ← PRESERVE — never touch
  theme: {
    extend: {
      colors: { primary: '#6366f1' },       // ← UPDATE with engine values
      borderRadius: { DEFAULT: '0.5rem' },  // ← UPDATE with engine values
    },
  },
  plugins: [forms],                         // ← PRESERVE — never touch
}
export default config
```

**globals.css — Protected Content (Non-Negotiable):**

When resetting CSS variables in `globals.css`, ONLY update the CSS custom property values inside `:root { }` blocks. NEVER touch:

1. **`@tailwind` directives** — `@tailwind base;`, `@tailwind components;`, `@tailwind utilities;` MUST remain exactly as-is. These are Tailwind build directives, not token values. Removing or modifying them will break the entire Tailwind compilation.
2. **`@media` query blocks** — preserve the `@media (prefers-color-scheme: dark) { ... }` structure. Only update the CSS variable VALUES inside the `:root { }` within those blocks.
3. **`body { }` and other standard CSS rules** — preserve all non-variable CSS rules (body font-family, base styles, selector rules, etc.).
4. **`@layer` wrapper structure** — Preserve the `@layer base { }`, `@layer components { }`, and `@layer utilities { }` keyword and block wrapper. **Important: the `@layer` WRAPPER is protected, but `@apply` rules inside `@layer components { }` blocks are NOT fully protected.** If a `@layer components { }` block contains custom class definitions using `@apply` with hardcoded Tailwind layout/color/spacing/radius class names (e.g., `.btn { @apply flex gap-2 px-4 py-2 bg-blue-500 rounded-md; }`), those `@apply` class values must be rebuilt in Step 10 — the same way as `className` values in JSX. Preserve: the selector name (`.btn`, `.card`), the `@apply` keyword itself, and any CSS variable references inside `@apply` (e.g., `@apply bg-[--color-primary]` — the variable value is already updated in `:root { }`). Fully preserve `@layer base {}` HTML element reset rules (e.g., `*, ::before, ::after { box-sizing: border-box; }`) as-is.
5. **Import statements** — `@import` lines must not be removed.
6. **`@font-face` blocks** — `@font-face { font-family: ...; src: ...; font-weight: ...; font-display: ...; }` blocks define custom web fonts. NEVER remove or modify them — they are font loading declarations, not CSS variable tokens. Preserve the entire `@font-face` block exactly as-is.
7. **`@keyframes` blocks** — `@keyframes animName { from { ... } to { ... } }` blocks define animations. NEVER remove or modify them — they are animation definitions, not design tokens. Preserve all `@keyframes` blocks exactly as-is, including all keyframe stops (0%, 25%, `from`, `to`, etc.).

**What changes in globals.css during Step 7:**
- CSS custom property VALUES inside `:root { }` blocks: clear them (e.g., `--background: ;`)

**What changes in globals.css during Step 10 (rebuild):**
- CSS custom property VALUES inside `:root { }` blocks: set to new style engine values
- CSS custom property VALUES inside `@media (prefers-color-scheme: dark) { :root { } }`: set to engine dark mode values
- `@apply` class values inside `@layer components { }` custom class definitions: update with style engine values (preserve selector names and `@apply` keyword)

**Example — correct globals.css handling:**
```css
/* BEFORE */
@tailwind base;           ← PRESERVE exactly
@tailwind components;     ← PRESERVE exactly
@tailwind utilities;      ← PRESERVE exactly

@font-face {              ← PRESERVE entire block — never touch
  font-family: 'CustomFont';
  src: url('/fonts/CustomFont.woff2') format('woff2');
  font-weight: 400;
  font-display: swap;
}

@keyframes fadeIn {       ← PRESERVE entire block — never touch
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

:root {
  --background: #ffffff;  ← UPDATE value only
  --primary: #6366f1;     ← UPDATE value only
}

@media (prefers-color-scheme: dark) {  ← PRESERVE block structure
  :root {
    --background: #0a0a0a;             ← UPDATE value only
  }
}

body {                    ← PRESERVE entire rule
  font-family: system-ui;
}

.animate-fade-in {        ← PRESERVE entire rule — references a keyframe
  animation: fadeIn 200ms ease-out;
}
```

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

**Inline styles rebuild (when styling system is "Inline styles"):**

If Step 3 detected inline styles as the primary styling system, apply the new style engine values directly into the `style={{}}` props — do NOT convert to className unless the project has adopted Tailwind during the restructure.

- Replace stripped padding/margin values with style engine spacing scale values (e.g., `padding: '16px'` → `padding: '16px'` from engine md spacing)
- Replace stripped color values with style engine color system values (hex/rgba from engine)
- Replace stripped borderRadius values with engine radius scale values
- Replace stripped fontSize/fontWeight with engine typography scale values
- For dynamic conditional values: update BOTH branches — e.g., `background: hovered ? '#f5f5f5' : '#fff'` → `background: hovered ? '[engine surface color]' : '[engine page color]'`
- Preserve all event handlers, state references, and prop bindings unchanged

**Plain CSS rebuild (when styling system is "Plain CSS"):**

If Step 3 detected plain CSS imports (`import './styles/card.css'` or similar), rebuild the CSS files with style engine values while preserving all CSS structure.

**JSX/TSX component files:** Do NOT change classNames (they are CSS selector references, not utility classes). The component files need NO className changes — only the CSS files change.

**CSS files (`*.css`):** Update property VALUES inside each rule block — keep all class selectors (`.card`, `.card-title`, etc.) exactly as-is. Only the values change:

```css
/* Before — original values */
.card { padding: 16px; background: #fff; border-radius: 8px; border: 1px solid #e5e7eb; }
.card-title { font-size: 14px; font-weight: 600; color: #111; }
.card-btn { padding: 8px 16px; background: #6366f1; color: #fff; border-radius: 6px; }

/* After — style engine values applied (example: minimal engine) */
.card { padding: 16px; background: #FAFAFA; border-radius: 6px; border: 1px solid #E5E5E5; }
.card-title { font-size: 14px; font-weight: 600; color: #171717; }
.card-btn { padding: 8px 16px; background: #171717; color: #FFFFFF; border-radius: 4px; }
```

**CSS custom properties (`variables.css`):** Update variable values with style engine color, spacing, and radius values:

```css
/* Before */
:root { --color-primary: #6366f1; --color-bg: #fff; --radius-md: 8px; }

/* After — style engine values applied */
:root { --color-primary: #171717; --color-bg: #FFFFFF; --radius-md: 6px; }
```

**Rules:**
- NEVER rename CSS class selectors
- NEVER remove CSS rules or properties
- NEVER change the CSS file structure (selector order, rule grouping)
- ONLY update property values (colors, padding, border-radius, font-size, font-weight, etc.) to match the style engine
- `import './styles/*.css'` statements in JSX/TSX are preserved unchanged (Step 6 rule)

**`@apply` in `@layer components {}` rebuild (when globals.css has `@layer components` blocks):**

If `globals.css` contains `@layer components { }` blocks with custom class definitions that use `@apply` with hardcoded Tailwind class names, update those `@apply` values in this step with style engine values.

- **Preserve:** the `@layer components { }` wrapper, selector names (`.btn`, `.card`), the `@apply` keyword, and any CSS custom property references (e.g., `@apply bg-[--color-primary]` — variable value already set via `:root { }`)
- **Update:** the Tailwind utility class values after `@apply` — apply engine layout, spacing, color, and radius values
- **Do NOT update** `@layer base {}` HTML element reset rules (e.g., `*, ::before, ::after { box-sizing: border-box; }`)

```css
/* Before (original @apply classes) */
@layer components {
  .btn-primary { @apply flex items-center gap-2 px-4 py-2 bg-blue-500 text-white rounded-md font-medium; }
  .card        { @apply rounded-xl border border-gray-200 bg-white shadow-sm p-6; }
}

/* After (apple engine @apply values applied) */
@layer components {
  .btn-primary { @apply flex items-center gap-3 px-5 py-2.5 bg-blue-500/90 text-white rounded-xl font-medium backdrop-blur-sm; }
  .card        { @apply rounded-2xl border border-white/20 bg-white/80 shadow-xl backdrop-blur-md p-6; }
}
```

For each file modified, show a brief diff summary.

---

## Step 11 — Polish Pass (mandatory, never skip)

After rebuilding all UI files, read `references/ui-craft.md` and apply the full polish pass to every component:

**Motion:**
- Add `transition-all` or specific transition properties to every interactive element
- Apply correct easing curves (ease-out for entrance, ease-in for exit, spring for press)
- Add `active:scale-[0.97]` to all buttons
- Add `hover:translateY(-2px)` + shadow transition to all clickable cards
- **Framer Motion branch (when Step 3 detected `framer-motion: true`):** Use Framer Motion variants INSTEAD of CSS transition classes. Specifically:
  - Wrap interactive buttons with `motion.button` (or add `whileTap={{ scale: 0.97 }}` to existing `motion.*` elements) using the `buttonPress` preset from `references/ui-craft.md` Section 1
  - Add `whileHover={{ y: -2 }}` + `transition={{ type: "spring", stiffness: 300, damping: 20 }}` to clickable card elements (use `motion.div` wrapper)
  - For list/grid entrances, wrap the container with `motion.div` and use the `staggerContainer` + `staggerItem` presets from `references/ui-craft.md` Section 1
  - Preserve ALL existing `motion.*` element tags — do NOT convert `motion.div` to `div`
  - Preserve ALL existing Framer Motion props (`animate`, `initial`, `exit`, `variants`, `transition`, `whileHover`, `whileTap`) on existing motion elements — only ADD new ones, never remove or overwrite
  - Do NOT add CSS `transition-*` classes to elements that already use Framer Motion variants
  - Note in output: "Framer Motion detected — spring variants applied (whileTap, whileHover, stagger)
- **CSS-only branch (when Step 3 did NOT detect Framer Motion):** Apply CSS transition classes as specified above (transition-all, active:scale-[0.97], etc.)

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
[Tokens regenerated: ✓]  ← use this line when tokens were reset (default, no --keep-tokens)
[Tokens kept: ✓]         ← use this line when --keep-tokens was passed (one line only, not both)
Polish pass: ✓ (motion, states, typography, icons, accessibility)
```

Note: Output exactly ONE of the two tokens lines based on the flag — `Tokens regenerated: ✓` when tokens were reset and rebuilt, `Tokens kept: ✓` when `--keep-tokens` was passed. Never output both lines.

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
