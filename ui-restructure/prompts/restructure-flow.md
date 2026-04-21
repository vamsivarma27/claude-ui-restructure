# Restructure Flow — Master Execution Prompt

This file defines the complete internal reasoning flow for the `/restructure` skill. Reference this when executing any restructure command.

---

## Phase 0: God Mode Check (run FIRST — before any other phase)

Before doing anything else, check if the command contains `--god-mode`.

```
If --god-mode flag is present:
  → DO NOT execute Phases 1–7 below
  → Read references/user-mindset.md fully (internalize 10 Laws + 10 developer mistakes)
  → Read modes/godmode.md fully (this is your complete execution guide)
  → Execute the 7-phase God Mode pipeline defined in godmode.md
  → Default behavior: tokens are PRESERVED (do NOT reset tokens unless --remove-tokens is also passed)
  → If --remove-tokens is also passed: after God Mode completes, run Phase 3 (Token Reset)
     and Phase 4 (Token Generation) using the style from --style (default: minimal)
  → God Mode CANNOT combine with --mode (warn user; --mode is ignored)
  → God Mode CAN combine with --style (defines token system when --remove-tokens is used)
  → If --prompt is also provided: acknowledge it in output with a note
     ("Note: --prompt was provided but God Mode uses its own user-first analysis pipeline")
     and proceed with standard God Mode execution. Do NOT error or crash.
  → Output the God Mode report from godmode.md Phase 7, then STOP
```

Only if `--god-mode` is NOT present: continue to Phase 1 below.

---

## Pre-Flight Checklist

Before starting any restructure:

- [ ] Parse all arguments from the command
- [ ] Confirm parsed arguments with user (one-line summary)
- [ ] Verify you understand the codebase structure (run a quick scan)
- [ ] Confirm you will not touch logic files
- [ ] Start restructure

---

## Phase 1: Reconnaissance

### 1A — Project scan

```
Scan:
  package.json → dependencies → detect React/Next/Vue + styling libraries
                              → also check for framer-motion (record if found)
  tsconfig.json → path aliases
  tailwind.config.* → current token setup
  globals.css / variables.css → CSS variable setup
  app/ → routing model (Next.js App Router at project root)
  src/app/ → routing model (Next.js App Router inside src/ — src/ layout convention)
  pages/ → routing model (Next.js Pages Router)
  src/views/ or src/pages/ → routing model (Vue/React)
  components/ → component inventory
  src/components/ → component inventory
```

**Framework detection signals (check project root AND src/):**

| Signal | Framework |
|---|---|
| `app/` directory (at root) + `layout.tsx` inside it | Next.js App Router |
| `src/app/` directory + `layout.tsx` inside `src/app/` | Next.js App Router (src/ layout convention) |
| `pages/` directory + `_app.tsx` | Next.js Pages Router |
| `vite.config.ts` + no `app/` dir | React (Vite) |
| `src/App.jsx` + `public/index.html` | React (CRA) |
| `*.vue` files + `vite.config.ts` | Vue 3 |

**Conflict resolution — when multiple signals match:**
- If both `app/` (with `layout.tsx`) and `pages/` exist simultaneously: **App Router wins.** Treat as Next.js App Router. Skip Pages Router scanning.
- If `src/app/` (with `layout.tsx`) and `pages/` exist simultaneously: **App Router wins** — same rule applies.
- If `*.vue` files exist alongside `app/` or `pages/`: Vue 3 wins.

**`src/app/` pattern:** When detected, use `src/app/` as the App Router scan root in Phase 1B (instead of `app/`). All exclusion rules and Server/Client Component rules apply identically.

### 1B — Build component inventory

List every UI file that will be modified. Before listing any file, apply ALL exclusion rules below.

**Scan directories (always recursive — all nested subdirectories at all depths):**

```
Always scan (recursively):
  - components/
  - src/
  - layouts/

Scan conditionally (recursively):
  - app/ — only if framework is Next.js App Router (root-level app/)
  - src/app/ — only if framework is Next.js App Router (src/ layout convention)
  - pages/ — only if framework is Next.js Pages Router (NOT when App Router detected)
```

**Route group directories** (Next.js App Router): directories whose names are wrapped in parentheses — e.g., `(auth)/`, `(marketing)/`, `(dashboard)/` — are route groups. They do NOT affect the URL path but they DO contain real component and page files. Treat them as regular directories during recursive scanning. Example: `app/(auth)/login/components/LoginForm.tsx` is in scope.

**When both `app/` and `pages/` exist and App Router was detected:** scan `app/` only. Do NOT scan or modify files in `pages/`.

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
   - NEVER modify test files.

2b. **Storybook story files** — files that define Storybook stories:
   - `*.stories.tsx` / `*.stories.ts` / `*.stories.jsx` / `*.stories.js`
   - `*.stories.mdx` — MDX story format
   - `*.story.tsx` / `*.story.ts` / `*.story.jsx` / `*.story.js`
   - Detection: if the filename contains `.stories.` or `.story.` before the file extension — skip it.
   - NEVER modify Storybook files.

3. **Type definition files** — not UI files:
   - `*.d.ts` files

4. **Server Action files** — files with `"use server"` directive are pure server logic, no UI:
   - Files whose first non-empty line is `"use server"`
   - Detection: if the first non-empty line of a file is `"use server"` — skip it.
   - Note: Server Components (no directive at all) are NOT excluded — they may have JSX and layout classes that need rebuilding.

5. **API Route files** — Next.js App Router API routes are pure server handlers, no UI:
   - Files named `route.ts` / `route.js` / `route.tsx` / `route.jsx` in any directory
   - Detection: if the filename is `route.ts` (or variants) — skip it.

6. **Service layer directories** — non-UI subdirectories inside scanned parent directories:
   - When scanning `src/` or `app/`, skip files inside these subdirectories: `lib/`, `utils/`, `services/`, `helpers/`, `hooks/` (pure hook files with no JSX), `store/`, `context/`, `config/`, `db/`, `models/`, `types/`
   - Exception: `src/components/` and `app/components/` are always scanned regardless.
   - Exception: `hooks/` files that contain JSX (render hooks that return components) — scan these. Pure hook files (no JSX) — skip.
   - Exception: `context/` files that contain JSX (React provider components that render layout wrappers, e.g., ThemeProvider, LayoutProvider) — scan these. Pure context definition files (no JSX — only `createContext`, `useContext`, type definitions) — skip. Detection: if the file contains JSX (`<` tags, className props) — scan it; otherwise skip.

7. **Middleware files** — Next.js middleware contains routing/auth logic, no JSX:
   - Files named `middleware.ts` / `middleware.js` / `middleware.tsx` / `middleware.jsx` in any directory
   - Detection: if the filename is `middleware.ts` (or variants) — skip it.
   - Applies to root-level `middleware.ts` AND `src/middleware.ts`.

8. **Instrumentation files** — Next.js observability hooks (OpenTelemetry, Sentry, Datadog); no JSX:
   - Files named `instrumentation.ts` / `instrumentation.js` / `instrumentation.node.ts` / `instrumentation.node.js` in any directory
   - These export a `register()` function — pure server-side monitoring setup, no UI classes.
   - Detection: if the filename is `instrumentation.ts`, `instrumentation.js`, `instrumentation.node.ts`, or `instrumentation.node.js` — skip it.
   - Applies to root-level `instrumentation.ts` AND `src/instrumentation.ts`.

Applying these exclusions prevents: (1) barrel files from being accidentally modified, (2) test files from being treated as UI components, (2b) Storybook story files from being treated as UI components, (3) TypeScript declarations from being touched, (4) Server Action files from being scanned, (5) API route files from being scanned, (6) service layer utility files from being scanned, (7) middleware files from being scanned, (8) instrumentation files from being scanned.

Categorize files after exclusions:

```
Layout files (modify first):
  - app/layout.tsx  (or src/app/layout.tsx)
  - components/Layout.tsx
  - components/Navbar.tsx
  - components/Sidebar.tsx
  - components/Footer.tsx

Page files (modify second):
  - app/page.tsx  (or src/app/page.tsx)
  - app/[route]/page.tsx
  - pages/index.tsx

Component files (modify last):
  - components/*.tsx
  - src/components/*.tsx
```

For each file, also record:
- Is it a Client Component? (starts with `"use client"`)
- Is it a Server Component? (no `"use client"` directive — may use `async/await`, call `db.query()`, `getServerSession()`, etc.)

### 1C — Logic audit

Before stripping, identify in each file:

```
Logic to preserve:
  □ useState / useReducer calls
  □ useEffect calls
  □ Custom hook calls
  □ API calls (fetch, axios, SWR, React Query)
  □ Server actions (Next.js)
  □ Event handler functions
  □ .map() / .filter() / .reduce() on data
  □ Conditional renders based on data (not layout)
  □ Form submission logic
  □ Authentication checks
  □ Server-side data access (db.query(), getServerSession(), prisma.findMany(), etc.)
  □ import type / export type statements (TypeScript type-only imports/exports)
  □ Class component lifecycle methods (componentDidCatch, getDerivedStateFromError,
     componentDidMount, componentDidUpdate, componentWillUnmount) — treat as protected logic
  □ "use client" directive (Client Components — preserve exactly; do NOT add to Server Components)
  □ Async Server Component functions (export default async function — do NOT add "use client")
```

---

## Phase 2: Stripping

For each component file:

```
Read file
↓
Identify logic blocks → mark as PRESERVE
↓
Identify layout blocks → mark as STRIP
↓
Remove STRIP blocks
↓
Verify PRESERVE blocks still intact
↓
Write stripped file
```

**Strip only these patterns:**

```
Tailwind layout:     flex, grid, gap-*, p-*, m-*, space-*, max-w-*, w-*, h-*
Tailwind colors:     bg-*, text-*, border-*, ring-*, shadow-*
Tailwind type:       text-sm, text-lg, font-*, leading-*, tracking-*
Tailwind radius:     rounded-*
Tailwind effects:    opacity-*, blur-*, backdrop-*
Inline style values: static visual values inside style={{}} (padding, background, borderRadius,
                     fontSize, fontWeight) — strip values but preserve the style={{}} prop structure
                     and any dynamic/conditional logic referencing state or props
Plain CSS class names: DO NOT strip className="card" style selector references in plain CSS
                       projects (stripping them would break the CSS link). Only strip when
                       Tailwind utility classes are confirmed as the styling system.
```

**cn() and clsx() wrapper handling:**

When className is passed through `cn()` (shadcn: `import { cn } from '@/lib/utils'`) or `clsx()`:

1. **NEVER strip the wrapper function call.** `className={cn(...)}` remains `className={cn(...)}`.
2. **NEVER strip the `clsx()` wrapper.** `className={clsx(...)}` remains `className={clsx(...)}`.
3. **Preserve the import statements** for `cn` and `clsx` — they are NOT layout classes.
4. **Strip the layout class string values INSIDE the wrapper** — same as plain className strings.
5. **Preserve all conditional logic inside the wrapper** — `variant === 'error' && "bg-red-50"` strips the class value but preserves `variant === 'error' &&`.

**`cva()` variant definitions — flag for Phase 5 rebuild (do NOT strip):**

Files using `cva()` from `class-variance-authority` (shadcn/ui pattern) have module-level variant definitions that contain class strings but are NOT `className=` JSX attributes. Do NOT strip `cva()` calls during Phase 2 stripping.

During Phase 2 scan: if a file contains `import { cva } from "class-variance-authority"` — flag it for `cva()` rebuild in Phase 5. Do not strip the `cva(...)` call or its import. The `className={cn(buttonVariants({ variant, size }), className)}` JSX expression also does NOT need stripping (it contains no literal class strings — only a function call result).

`twMerge()` from `tailwind-merge` used directly as a className wrapper (not via `cn()`) follows the same rules as `cn()` — never strip the wrapper; treat string arguments inside it as className strings; preserve the import.

**Vue SFC handling (when framework is Vue 3):**

Vue Single File Components — handle each block differently:

- **`<template>` block:** Strip `class="..."` and `:class="..."` attributes that contain only layout/spacing/typography classes. Preserve `:class` expressions that contain conditional logic. Preserve `v-for`, `:key`, `@click`, `v-if`, and all other Vue directives and bindings.
- **`<script setup>` block (and `<script>` block):** Do NOT modify. Fully protected under PRESERVE rules.
- **`<style scoped>` block (and `<style>` block):** **Preserve as-is. Do NOT strip, reset, or modify scoped styles.** They are component-specific styles, not token files. Token files (Phase 3) are separate global token sources. This is non-negotiable.

**Never strip (non-className HTML attributes — NEVER strip these — Non-Negotiable):**

The strip pass targets ONLY the `className` attribute value (and `class=` on Vue template elements). Every other HTML and JSX attribute on every element MUST be left completely untouched:

```
Element identity:     id, name
Element type/role:    type, role
Form values:          value, defaultValue, checked, defaultChecked
Form constraints:     required, disabled, readOnly, maxLength, minLength, min, max,
                      step, pattern, multiple
Input hints:          placeholder, autoComplete, spellCheck, autoFocus, autoCapitalize
Accessibility:        aria-* (ALL ARIA attributes), htmlFor, tabIndex
Data attributes:      data-* (ALL data attributes — data-testid, data-analytics, etc.)
React-specific:       key, ref, suppressHydrationWarning, suppressContentEditableWarning
Event handlers (ALL): onClick, onKeyDown, onKeyUp, onKeyPress, onFocus, onBlur, onChange,
                      onSubmit, onMouseEnter, onMouseLeave, onMouseDown, onMouseUp,
                      onTouchStart, onTouchEnd, and ALL other on* event handlers
Media/link:           src, href, alt, target, rel, download, action, method
Content:              dangerouslySetInnerHTML, style (inline style prop — handled separately)
Spread props:         {...props}, {...btn}, {...rest} — NEVER strip spread prop expressions
Key props:            key={item.id}
Data props:           {...item}, value={data.name}
Conditional:          {isLoading && <Spinner />}
Data access:          item.name, user.email, product.price
Semantic tags:        <main>, <section>, <article>, <header>, <footer>
Form elements:        <form>, <input>, <select>, <textarea>
React.Fragment:       <Fragment key={...}> and shorthand <> — preserve exactly as-is
```

If you are uncertain whether an attribute is a layout class or a logic attribute — **PRESERVE it.** The only thing that changes is the string value(s) inside `className="..."` (and `class=`/`:class=` in Vue templates).

**Template literal classNames with embedded logic:**
`` className={`base-classes ${condition ? 'a' : 'b'}`} `` → strip the static CSS class strings but preserve the ternary/conditional logic and the template literal structure itself.

---

## Phase 3: Token Reset

If NOT `--keep-tokens`:

```
1. Open tailwind.config.ts (or tailwind.config.js)
2. Find theme.extend section
3. Clear ONLY: colors, fontSize, spacing, borderRadius, boxShadow (inside theme.extend)
4. Open globals.css
5. Find :root { } block
6. Clear ONLY the CSS custom property VALUES inside :root { } blocks
   (e.g., --background: ; --primary: ;)
```

**tailwind.config.ts / tailwind.config.js — Protected Fields (Non-Negotiable):**

When resetting tokens in `tailwind.config.ts` or `tailwind.config.js`, ONLY update `theme.extend` values. NEVER touch:

1. **`content` array** — tells Tailwind which files to scan. PRESERVE exactly.
2. **`plugins` array** — installed Tailwind plugins. PRESERVE exactly.
3. **`safelist` array** — class names that must always be generated. PRESERVE exactly.
4. **`darkMode` setting** — dark mode strategy. PRESERVE exactly.
5. **Import statements** — `import forms from '@tailwindcss/forms'` etc. PRESERVE exactly.
6. **`presets` array** — if present, PRESERVE exactly.

**globals.css — Protected Content (Non-Negotiable):**

When resetting CSS variables in `globals.css`, ONLY update the CSS custom property values inside `:root { }` blocks. NEVER touch:

1. **`@tailwind` directives** — `@tailwind base;`, `@tailwind components;`, `@tailwind utilities;` MUST remain exactly as-is.
2. **`@media` query blocks** — preserve the `@media (prefers-color-scheme: dark) { ... }` structure. Only update CSS variable VALUES inside `:root { }` within those blocks.
3. **`body { }` and other standard CSS rules** — preserve all non-variable CSS rules.
4. **`@layer` wrapper structure** — Preserve the `@layer base { }`, `@layer components { }`, and `@layer utilities { }` keyword and block wrapper. **Important: the `@layer` WRAPPER is protected, but `@apply` rules inside `@layer components { }` blocks are NOT fully protected.** If a `@layer components { }` block contains custom class definitions using `@apply` with hardcoded Tailwind layout/color/spacing/radius class names (e.g., `.btn { @apply flex gap-2 px-4 py-2 bg-blue-500 rounded-md; }`), those `@apply` class values must be rebuilt in Phase 5 — the same way as `className` values in JSX. Preserve: the selector name, the `@apply` keyword, and any CSS variable references inside `@apply`. Fully preserve `@layer base {}` HTML element reset rules as-is.
5. **Import statements** — `@import` lines must not be removed.
6. **`@font-face` blocks** — `@font-face { font-family: ...; src: ...; font-weight: ...; font-display: ...; }` blocks define custom web fonts. **NEVER remove or modify them** — they are font loading declarations, not CSS variable tokens.
7. **`@keyframes` blocks** — `@keyframes animName { from { ... } to { ... } }` blocks define animations. **NEVER remove or modify them** — they are animation definitions, not design tokens. Preserve ALL keyframe stops (0%, 25%, `from`, `to`, etc.).

```
Preserve:
- tailwind.config.ts: content array, plugins list, safelist, darkMode setting, presets, imports
- globals.css: @tailwind directives, @font-face blocks, @keyframes blocks,
               @layer wrapper structure, @media query structures, body {} rules,
               CSS selector rules that reference animations (e.g., .animate-fade-in { ... })
Update in Phase 5:
- globals.css: @apply class values inside @layer components {} custom class definitions
               (selector names and @apply keyword preserved; class values rebuilt with engine)
```

---

## Phase 4: Token Generation

Load style engine. Extract:

```
From engine file:
  spacing.scale → generate spacing variables
  typography.scale → generate font-size variables
  color.system → generate color tokens
  shadow.scale → generate shadow tokens
  radius.scale → generate border-radius tokens
```

Apply to:

```
tailwind.config.ts → theme.extend (colors, borderRadius, boxShadow, fontFamily, spacing)
globals.css → :root { } CSS variables (VALUES only — structure preserved from Phase 3)
globals.css → @apply values inside @layer components {} custom class definitions
              (selector names + @apply keyword preserved; class values rebuilt with engine)
tokens.ts (if exists) → export const tokens = { ... }
theme.ts / design-system.ts (if exist alongside tailwind.config.ts) → update in sync
```

---

## Phase 5: Component Rebuild

For each stripped component:

```
Read stripped component
↓
Determine component role:
  - Navigation? → Apply nav pattern from engine
  - Card? → Apply card pattern from engine
  - Form? → Apply form pattern from engine
  - Table/list? → Apply list/grid pattern from engine
  - Page section? → Apply section pattern from engine
↓
Apply layout from mode file
↓
Apply tokens from style engine
↓
Write rebuilt component
↓
Verify:
  □ All original map() calls present
  □ All original handlers present
  □ All original data bindings present
  □ No hardcoded data (only original data refs)
  □ "use client" directive preserved exactly (Client Components)
  □ No "use client" added to Server Components
  □ import type / export type statements preserved
  □ Class component lifecycle methods preserved (componentDidCatch, etc.)
  □ Spread props preserved ({...btn}, {...props})
  □ React.Fragment / Fragment preserved (including named Fragment with key)
  □ cva() calls: variant/size NAMES unchanged, defaultVariants unchanged, import preserved
```

**`cva()` variant definitions rebuild (for files flagged in Phase 2):**

For any file flagged as having `import { cva } from "class-variance-authority"` during Phase 2 scan, also rebuild the `cva()` class strings in this phase:

- **Update:** the base class string (first `cva()` argument) with engine radius, font, spacing, and transition values
- **Update:** every variant VALUE string (e.g., `default: "bg-primary ..."`, `outline: "border ..."`) with engine color/background values
- **Update:** every size VALUE string with engine spacing/height values
- **Preserve:** all variant NAME keys (`default`, `destructive`, `outline`, `ghost`, `sm`, `lg`, `icon`, etc.) — only VALUES change
- **Preserve:** `defaultVariants` object — never change
- **Preserve:** `VariantProps<typeof ...>` TypeScript types
- **Preserve:** `import { cva, type VariantProps } from "class-variance-authority"` — never remove

The JSX `className={cn(buttonVariants({ variant, size }), className)}` does NOT need separate changes — it references the now-rebuilt `cva()` definition automatically.

---

## Phase 5.5: Polish Pass (mandatory — never skip)

After rebuilding all component files (Phase 5), apply the full polish pass from `references/ui-craft.md`.

**Run this for every component before finalizing:**

**Motion:**

Check if Framer Motion was detected in Phase 1A (`framer-motion` in package.json or component imports).

- **Framer Motion branch (framer-motion detected):**
  - Wrap interactive buttons with `motion.button` or add `whileTap={{ scale: 0.97 }}` using the `buttonPress` preset from ui-craft.md Section 1
  - Add `whileHover={{ y: -2 }}` + `transition={{ type: "spring", stiffness: 300, damping: 20 }}` to clickable card elements
  - For list/grid entrances, use `staggerContainer` + `staggerItem` presets from ui-craft.md Section 1
  - **Preserve ALL existing `motion.*` element tags** — do NOT convert `motion.div` to `div`
  - **Preserve ALL existing Framer Motion props** (`animate`, `initial`, `exit`, `variants`, `transition`, `whileHover`, `whileTap`) — only ADD new ones, never remove or overwrite
  - Do NOT add CSS `transition-*` classes to elements that already use Framer Motion variants
  - Note in output: "Framer Motion detected — spring variants applied (whileTap, whileHover, stagger)"
- **CSS-only branch (no Framer Motion):**
  - Add `transition-all` or specific transition properties to every interactive element
  - Apply correct easing curves (ease-out for entrance, ease-in for exit, spring for press)
  - Add `active:scale-[0.97]` to all buttons
  - Add `hover:-translate-y-1` + shadow transition to all clickable cards

**States:**
- Every button: default, hover, focus-visible, active, disabled styles
- Every input: default, focus (ring + border), error, disabled styles
- Every list/table: empty state component
- Every async action: loading state (skeleton or spinner)
- Every error path: error state component

**Typography:**
- Verify line-height is proportional to font size
- Add `text-balance` to all headings to prevent orphaned words
- Verify `max-w-prose` on body copy blocks
- Apply negative letter-spacing to headings ≥ 24px

**Icons:**
- Verify one icon library used consistently
- Verify icon size matches adjacent text size optically
- Add `aria-hidden="true"` to all decorative icons
- Add `aria-label` to all icon-only buttons

**Spacing:**
- Verify all spacing values are on 4/8px grid
- Verify inner border-radius = outer border-radius − padding on nested cards

**Accessibility:**
- Replace `outline: none` with `focus-visible` custom ring styles
- Verify all interactive elements are keyboard reachable
- Verify touch targets ≥ 44×44px

Run the complete checklist from `references/ui-craft.md` Section 11 before finalizing.

---

## Phase 6: Verification

After all files rebuilt and polish pass applied:

```
For each modified file:
  □ Open file
  □ Search for original handlers → confirm present
  □ Search for original .map() calls → confirm present
  □ Search for API calls → confirm present
  □ Confirm "use client" directive status unchanged
  □ Confirm no Server Component was given "use client"
  □ Confirm import type / export type statements intact
  □ Run mental diff: "What did I change? Only UI."
```

If any logic is missing → immediately fix before proceeding.

---

## Phase 7: Report

Output final report (this format matches SKILL.md Step 11 output):

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

Note: Output exactly ONE of the two tokens lines based on the flag.

---

## Error Handling

### If a file cannot be determined:

Skip it, note it in the report:
```
⚠ Skipped: components/UnknownWidget.tsx (could not determine role)
   Manual review recommended.
```

### If logic is ambiguous:

When it's unclear if something is UI or logic, **keep it** (err on the side of preservation):
```
// Keeping this — may be data-driven conditional
{showSidebar && <Sidebar />}
```

### If user has custom components (e.g., shadcn):

For shadcn components, only modify the **wrapper** and **className props**, not the component internals:

```jsx
// OK to change:
<Card className="[new classes]">

// Do NOT change:
<CardHeader> → <CardContent> → <CardFooter> structure (shadcn internal)
```

---

## Framework-Specific Notes

### Next.js App Router

- `app/layout.tsx` (or `src/app/layout.tsx`) is the root shell — rebuild this first
- **`"use client"` directive must be preserved exactly as-is** in Client Component files
- **NEVER add `"use client"` to Server Components** — this would break async data access (db.query, getServerSession, etc.)
- **NEVER remove `"use client"` from Client Components** — this would cause a build error (hooks and event handlers are not allowed in Server Components)
- Server Components: can have layout JSX but no hooks — rebuild layout, preserve server-side data access
- Client Components: full rebuild including preserving hooks context
- Async Server Components (`export default async function Page()` with no `"use client"`): DO NOT add `"use client"` to make them non-async or to enable hooks

### Vue 3

- `<template>` section → strip and rebuild (layout classes only)
- `<script setup>` section → preserve everything (fully protected)
- `<style scoped>` section → **preserve as-is (not a token file)** — do NOT replace, do NOT modify
- `:key` bindings in `v-for` → preserve
- Vue directives (`v-if`, `v-for`, `@click`, `:class` conditional logic) → preserve

### styled-components

- Replace `styled.div` template literals layout properties
- Keep interpolated props and conditional logic
- Theme provider → update theme object with new engine values
