# Bug Tracker — ui-restructure Skill
> Maintained by the TEST-LOOP. Updated each cycle. Never delete entries — mark them FIXED with date.

---

## Legend
- `[OPEN]` — bug confirmed, not yet fixed
- `[FIXED yyyy-mm-dd]` — resolved, commit hash noted
- `[INVESTIGATING]` — reproducing or diagnosing
- `[WONTFIX]` — acknowledged, not a skill defect

---

## Active Bugs

_None — all Cycle 10 bugs fixed in same cycle._

---

## Fixed Bugs

### BUG-001: Step 2 has no tie-breaking rule for app/ + pages/ coexistence; Step 4 scans pages/ unconditionally
- Status: [FIXED 2026-04-21]
- Persona: Persona 3 — Priya (adversarial: both app/ AND pages/ directories coexist)
- Command: `/ui-restructure --style linear`
- Skill File: `SKILL.md`
- Line: Step 2 (line ~84), Step 4 (line ~112)
- Symptom: When a Next.js project has both `app/` and `pages/` directories, Step 2 provides no conflict-resolution rule — the skill would unpredictably pick either framework. Step 4 unconditionally lists `pages/` in its scan list regardless of detected framework, meaning pages/ files would be scanned and potentially modified even in App Router projects.
- Root Cause: Step 2's framework detection table lists app/ and pages/ as independent signals with no precedence rule for when both are present. Step 4's directory list is static and does not condition on the framework detected in Step 2.
- Fix Applied: Added conflict-resolution section to Step 2 (App Router wins when both exist; Vue 3 wins over Next.js when .vue files are present). Rewrote Step 4's directory scan list to be conditional on detected framework — pages/ only scanned for Pages Router projects; when App Router is detected and pages/ also exists, only app/ is scanned.
- Commit: see cycle 2 commit
- Regression Risk: Any project with hybrid Next.js setup (app/ + pages/). Also affects detection for Vue 3 projects that happen to have a pages/ directory.

---

### BUG-002: No Vue SFC handling guidance in Step 6 strip-and-rebuild logic
- Status: [FIXED 2026-04-21]
- Persona: Persona 4 — Alex (adversarial: Vue SFC with script setup AND style scoped)
- Command: `/ui-restructure --style dashboard --density compact`
- Skill File: `SKILL.md`
- Line: Step 6 (line ~176)
- Symptom: Step 6's strip/rebuild logic is documented exclusively with React/JSX examples (className, key props). There is no guidance for Vue SFC block handling: (1) class="" vs className="" for stripping, (2) <style scoped> preservation — the spec could be misread to strip scoped styles as token files, (3) :key Vue binding preservation in v-for, (4) <script setup> block being treated as fully protected logic, (5) template literal classNames with embedded conditional logic.
- Root Cause: The step 6 examples were written with React/JSX in mind and were never extended to cover Vue 3 SFC specifics. This creates ambiguity for all Vue 3 projects.
- Fix Applied: Added a "Vue SFC handling" subsection to Step 6 that explicitly covers: <template> class stripping (class= and :class= attributes), <script setup> and <script> blocks as fully protected, <style scoped> preservation (not treated as token files), v-for :key binding preservation, and template literal className conditional logic handling.
- Commit: see cycle 2 commit
- Regression Risk: All Vue 3 projects. Step 6 previously provided no Vue-specific guidance, risking scoped style destruction and incorrect attribute handling.

---

### BUG-003: commands.md references wrong step number for --keep-tokens
- Status: [FIXED 2026-04-21]
- Persona: Persona 2 — Dev (adversarial: no token files at all)
- Command: `/ui-restructure --mode layout --keep-tokens`
- Skill File: `parser/commands.md`
- Line: Line 101
- Symptom: `commands.md` says `--keep-tokens` skips "Step 6 (token reset)" — but in SKILL.md, the token reset step is Step 7, not Step 6. This creates a documentation inconsistency that could confuse implementations reading the parser spec.
- Root Cause: Stale step reference — the step number was not updated when SKILL.md steps were renumbered or reorganized.
- Fix Applied: Updated commands.md `--keep-tokens` section to reference "Step 7 (token reset)" — matching the actual step number in SKILL.md.
- Commit: see cycle 2 commit
- Regression Risk: Low — documentation inconsistency only. Any implementation that blindly follows the parser spec's step number would skip the wrong step.

---

### BUG-004: modes/theme.md Token Replacement Strategy for Tailwind does not cover dual token files (tailwind.config.ts + tokens.ts coexisting)
- Status: [FIXED 2026-04-21]
- Persona: Probe P2 — dual token files with --mode theme --style apple
- Command: `/ui-restructure --mode theme --style apple`
- Skill File: `modes/theme.md`
- Line: Line 65 (Token Replacement Strategy → For Tailwind projects section)
- Symptom: When a project has BOTH `tailwind.config.ts` AND a separate `tokens.ts` file, the theme.md "For Tailwind projects" strategy only instructs updating `tailwind.config.ts`. A strict reading leaves `tokens.ts` untouched, causing the two files to fall out of sync — `tailwind.config.ts` gets Apple token values but `tokens.ts` retains the old values.
- Root Cause: The Token Replacement Strategy section was written assuming a Tailwind project only has `tailwind.config.ts` as its token source. The Scope table correctly lists `tokens.ts / theme.ts` as "Reset + rebuilt", but the procedural steps (the how-to) didn't mention the dual-file case.
- Fix Applied: Added step 4 to the "For Tailwind projects" strategy: explicitly instructs that if a separate `tokens.ts` / `theme.ts` / `design-system.ts` exists alongside `tailwind.config.ts`, that file must ALSO be updated with style engine values. Both files must stay in sync.
- Commit: see cycle 3 commit
- Regression Risk: Any project that uses both a Tailwind config AND a separate tokens file (a common pattern in design-system-aware Next.js apps).

---

### BUG-005: All 4 engine files missing Motion/transition section
- Status: [FIXED 2026-04-21]
- Persona: Cycle 4 A1 — Engine Completeness Audit
- Command: any `/ui-restructure --style apple|linear|minimal|dashboard` command
- Skill File: `engines/apple.md`, `engines/linear.md`, `engines/dashboard.md`, `engines/minimal.md`
- Line: End of each engine file (before Tailwind Config Additions)
- Symptom: SKILL.md Step 8 says "read the engine file fully — it defines the complete design system." SKILL.md Step 11 instructs applying engine-specific motion (spring, easing, durations). But none of the 4 engine files defined a Motion/transition section — each engine has distinct motion character (Apple = smooth/organic/dramatic, Linear = instant/precise, Dashboard = functional/unobtrusive, Minimal = understated), but this was undocumented, forcing the skill to use only generic ui-craft.md Section 1 values for all engines.
- Root Cause: The engine files were designed as visual/typography/color systems and motion specs were never added. This is a gap against the Cycle 4 audit requirement (required sections list: "Motion/transition specs (duration, easing)").
- Fix Applied: Added a `## Motion & Transition` section to all 4 engine files. Each section includes a duration scale, easing curves, and Tailwind class patterns appropriate for that engine's design character.
- Commit: see cycle 4 commit
- Regression Risk: Medium — any command using these engines. Before the fix, the skill would apply generic motion from ui-craft.md instead of engine-specific motion. After the fix, each engine provides its own motion spec.

---

### BUG-006: ui-craft.md Section 11 checklist missing aria-hidden on decorative icons
- Status: [FIXED 2026-04-21]
- Persona: Cycle 4 A4 — ui-craft.md structure audit
- Command: any command that triggers Step 11 polish pass
- Skill File: `references/ui-craft.md`
- Line: Section 11 (Polished Details Checklist), Icons block (line ~617)
- Symptom: SKILL.md Step 11 explicitly requires: "Add aria-hidden='true' to all decorative icons." The Section 11 checklist had an "Icons" block but only listed: consistent library, consistent stroke weight, optically aligned. The `aria-hidden` requirement from SKILL.md was absent from the checklist that the skill is instructed to "run ... before finalizing."
- Root Cause: The Section 11 checklist Icons block was written focusing on visual consistency (library/weight/size) but the accessibility requirement for decorative icons was not transferred from SKILL.md Step 11 into the checklist.
- Fix Applied: Added two items to the Icons block in Section 11: `aria-hidden="true" on all decorative icons (no semantic meaning)` and `aria-label on all icon-only interactive buttons` (combining the existing ARIA label item from the Accessibility block for completeness).
- Commit: see cycle 4 commit
- Regression Risk: Low — checklist-only gap. The SKILL.md Step 11 text already required it; this ensures the checklist reinforces it.

---

### BUG-007: grid.md missing --grid list (reverse: grid → list) handling section
- Status: [FIXED 2026-04-21]
- Persona: Cycle 4 T1 — `--mode grid --grid list` runtime test
- Command: `/ui-restructure --mode grid --grid list`
- Skill File: `modes/grid.md`
- Line: Grid Conversion section (line ~73), after "With --grid cards:"
- Symptom: `commands.md` defines `--grid list` as "Force list layout (reverses existing grids to lists)." `grid.md` is the executing mode file, but it only documented the forward direction: list → grid. There was no `### With --grid list:` section explaining how to detect existing CSS grids and convert them to vertical list layouts. Without this guidance, the skill had no concrete instructions for the reverse conversion case.
- Root Cause: grid.md was originally written to solve the common use case (list to grid) and the reverse case was defined in commands.md but never implemented in the mode file.
- Fix Applied: Added a `### With --grid list:` section to grid.md. This includes: grid detection patterns (CSS grid, flex-wrap), a before/after conversion example (grid-cols-* → flex flex-col + horizontal list rows), and explicit rules for what to preserve (map loops, event handlers, key props) and what to convert (grid container classes → flex-col, gap values, item structure).
- Commit: see cycle 4 commit
- Regression Risk: Any user passing `--mode grid --grid list`. Previously the skill had no guide for this path, which could result in either no change or incorrect output.

---

### BUG-008: SKILL.md Steps 6 and 10 lack inline style={{}} handling guidance
- Status: [FIXED 2026-04-21]
- Persona: Cycle 4 T5 — inline styles as the only styling system
- Command: `/ui-restructure --style minimal` on a project using only inline style={{}} props
- Skill File: `SKILL.md`
- Line: Step 6 (strip UI structure, line ~184), Step 10 (rebuild UI, line ~328)
- Symptom: Step 3 correctly detects `style={{}}` as "Inline styles." However, Step 6's strip guidance only shows className removal examples. Step 10's rebuild instructions only mention regenerating tokens, applying spacing/color/typography via classes. For a pure inline-styles project, there are no classNames to strip or rebuild. The skill had no instructions for: (1) stripping inline style values (padding, background, fontSize, etc.) while preserving conditional/dynamic logic, (2) injecting new style engine values back into style={{}} props during rebuild.
- Root Cause: Steps 6 and 10 were written with className-based styling systems (Tailwind, CSS Modules) in mind. The inline styles detection in Step 3 was never wired to corresponding strip/rebuild behavior in Steps 6 and 10.
- Fix Applied: Added "Inline styles handling" subsection to Step 6: explains stripping static visual values (colors, padding, etc.) while preserving conditional logic in ternary expressions, with a before/after example. Added "Inline styles rebuild" subsection to Step 10: explains injecting style engine values directly into style={{}} props (not converting to className), with rules for both static and dynamic conditional values.
- Commit: see cycle 4 commit
- Regression Risk: All projects using inline styles as primary styling. Before the fix, the skill had no guidance for this case, risking either no visual change (styles left as-is) or broken output.

---

### BUG-009: `--prompt` + `--god-mode` combination undocumented in parser spec
- Status: [FIXED 2026-04-21]
- Persona: Cycle 5 C2 — `--god-mode --prompt "minimalist task manager"` (undocumented combination)
- Command: `/ui-restructure --god-mode --prompt "minimalist task manager"`
- Skill File: `parser/commands.md`
- Line: `--god-mode` section, "God Mode can combine with:" list (line ~38)
- Symptom: The `--god-mode` section in commands.md lists what God Mode can and cannot combine with (`--remove-tokens`, `--style`, `--mode`, `--keep-tokens`) but makes no mention of `--prompt`. When both `--god-mode` and `--prompt` are provided, the skill has no guidance on whether to: (1) use the prompt to guide god mode decisions, (2) silently ignore it, (3) error. God Mode's 7-phase pipeline in godmode.md has no "apply user prompt" step, so the prompt is functionally inapplicable to god mode. The silence creates ambiguity.
- Root Cause: The commands.md god mode compatibility list was written before the `--prompt` parameter interaction was considered. The god mode pipeline is self-contained (user behavior analysis drives all decisions) and has no hook for a custom prompt, but this was never documented.
- Fix Applied: Added a "God Mode and `--prompt`:" subsection to commands.md `--god-mode` section. Clarifies that `--prompt` is silently noted but not functionally used in God Mode (god mode uses its own 7-phase user behavior analysis, not a description string). Instructs the skill to acknowledge the prompt in output with a note and continue with standard God Mode execution — no crash, no error.
- Commit: see cycle 5 commit
- Regression Risk: Low — documentation gap only. Any user passing both `--god-mode` and `--prompt` would previously get undefined behavior (prompt silently ignored with no explanation). After fix, behavior is well-defined and communicated to the user.

---

### BUG-010: SKILL.md has no guidance for Next.js App Router Server vs Client Component distinction
- Status: [FIXED 2026-04-21]
- Persona: Cycle 6 B1 — `/ui-restructure --style apple` on Next.js App Router project with Server and Client Components
- Command: `/ui-restructure --style apple`
- Skill File: `SKILL.md`
- Line: Behavior Rules section (line ~37), Step 4 (Scan UI Files), Step 6 (Strip UI Structure), Step 10 (Rebuild UI)
- Symptom: Next.js 13+ App Router uses Server Components (no `"use client"`) and Client Components (`"use client"` at top). SKILL.md had zero guidance on this distinction. The skill could: (1) add `"use client"` to Server Components (breaking async data access like `db.query()`, `getServerSession()`), (2) remove `"use client"` from Client Components (breaking hooks and event handlers), or (3) fail to recognize server-side data access (`db.query`, `getServerSession`) as protected logic equivalent to API calls. B1-V1 through B1-V5 (directive preservation) and B1-V9/B1-V10 (server-side data access) all had no guidance.
- Root Cause: The Behavior Rules and step-by-step guidance were written before App Router RSC architecture became prevalent. The `"use client"` directive is a React/Next.js compile-time boundary marker — not a className or logic statement — and fell through all existing protection categories.
- Fix Applied: Added a "Next.js App Router — Server vs Client Component Rules (Non-Negotiable)" section to SKILL.md immediately after the Behavior Rules block. This section: (1) defines how to identify Server vs Client Components by presence/absence of `"use client"`, (2) lists 5 explicit rules (never add `"use client"` to Server Components, never remove from Client Components, never modify the directive, treat server-side data access like db.query/getServerSession as protected logic, treat async Server Components as Server Components), (3) instructs Step 4 to record component type during scan, (4) instructs Step 6 to preserve `"use client"` during strip, (5) instructs Step 10 to not change component type during rebuild.
- Commit: see cycle 6 commit
- Regression Risk: High — all Next.js App Router projects. Any project using the App Router has Server Components by default. Without this fix, the skill had no protection for the `"use client"` directive or server-side data access patterns unique to RSC architecture.

---

### BUG-011: SKILL.md Steps 6 and 10 have no guidance for plain CSS class-value rebuilding
- Status: [FIXED 2026-04-21]
- Persona: Cycle 6 C1 — `/ui-restructure --style minimal` on React CRA project using plain `*.css` imports
- Command: `/ui-restructure --style minimal`
- Skill File: `SKILL.md`
- Line: Step 6 (Strip UI Structure, line ~220), Step 10 (Rebuild UI, line ~373)
- Symptom: Step 3 correctly detects "Plain CSS" from `*.css` imports. However, Steps 6 and 10 had no guidance for plain CSS projects. Without guidance: (1) Step 6 would incorrectly strip `className="card"` (a CSS selector reference) as if it were a Tailwind utility class — breaking the CSS link, (2) Step 6 could strip or corrupt `import './styles/card.css'` import statements, (3) Step 10 had no instructions for updating CSS property VALUES inside `.card { ... }` rules while preserving selector names, (4) there was no guidance that class names in JSX must NOT change (only CSS values change). C1-V4 (import not removed), C1-V5 (classNames preserved), C1-V6 (CSS values updated), C1-V8 (CSS structure preserved) all failed.
- Root Cause: Steps 6 and 10 were written assuming className-based styling (Tailwind, CSS Modules) where classNames map to utility values. For plain CSS, classNames are selector references — stripping them destroys the CSS connection. The distinction between "className as utility class" (Tailwind) and "className as selector reference" (plain CSS) was never documented.
- Fix Applied: Added "Plain CSS handling" subsection to Step 6: instructs preserving `className` attributes and CSS import statements for plain CSS projects; CSS files are not touched during strip. Added "Plain CSS rebuild" subsection to Step 10: provides before/after example of updating CSS property values while keeping selectors intact; lists explicit rules (never rename selectors, never remove rules/properties, only update values); covers `variables.css` custom property updating; clarifies that JSX files need no className changes — only the CSS files change.
- Commit: see cycle 6 commit
- Regression Risk: All projects using plain CSS imports as their primary styling system. Before the fix, the skill had no guidance for this styling pattern, risking either no visual change (CSS not touched at all) or broken output (classNames stripped, CSS links destroyed).

---

### BUG-013: SKILL.md Step 3 does not detect Framer Motion; Step 11 has no concrete application guidance for Framer Motion variants
- Status: [FIXED 2026-04-21]
- Persona: Cycle 7 D1 — `/ui-restructure --style apple` on Next.js project using `framer-motion`
- Command: `/ui-restructure --style apple`
- Skill File: `SKILL.md`
- Line: Step 3 (Detect Styling System, line ~130), Step 11 (Polish Pass, Motion bullet)
- Symptom: Step 3 detects styling systems (Tailwind, CSS Modules, styled-components, shadcn, inline styles, plain CSS) but has no detection for `framer-motion` — an animation library used in many React projects. Step 11 Motion bullet says "Use Framer Motion spring presets if the project already uses Framer Motion" — but provides NO guidance on: (1) how to detect Framer Motion (check package.json? scan imports?), (2) which specific props to add (`whileHover`, `whileTap`?), (3) that `motion.div` tags must not be converted to plain `div`, (4) that existing Framer Motion props must not be overwritten. Without detection, the skill cannot know to use Framer Motion. Without application guidance, the skill defaults to CSS transitions (`transition-all active:scale-[0.97]`) even on projects where Framer Motion is already in use, producing a mixed and inconsistent motion system.
- Root Cause: Step 3's detection table was written listing CSS/styling systems only. `framer-motion` is an animation library (not a CSS system) and was never added to the detection scan. Step 11's single-line Framer Motion reference was aspirational but not actionable — no concrete instructions existed for the Framer Motion branch of Step 11.
- Fix Applied: (1) Added Framer Motion detection subsection to Step 3: scan `package.json` and component imports for `framer-motion`. Record `framer-motion: true`. (2) Expanded Step 11 Motion bullet with a full "Framer Motion branch" instruction block: use `buttonPress` preset (whileTap), `whileHover={{ y: -2 }}` for cards, `staggerContainer`/`staggerItem` for lists, preserve all existing `motion.*` tags and props, do not add CSS transition classes when Framer Motion variants are in use. Added "CSS-only branch" label for the existing CSS transition instructions.
- Commit: see cycle 7 commit
- Regression Risk: All projects using Framer Motion. Before the fix, the skill had no detection for framer-motion and no concrete application path, causing CSS transitions to be applied alongside or instead of Framer Motion variants — resulting in a broken/inconsistent animation system.

---

### BUG-014: Step 4 has no exclusion rule for barrel files (pure re-export index.ts files)
- Status: [FIXED 2026-04-21]
- Persona: Cycle 7 E1 — `/ui-restructure --style linear` on project with `components/index.ts` barrel file
- Command: `/ui-restructure --style linear`
- Skill File: `SKILL.md`
- Line: Step 4 (Scan UI Files, line ~162), Step 6 (Strip UI Structure)
- Symptom: Step 4 scans `components/`, `src/`, `layouts/` for UI files. A barrel file like `components/index.ts` — containing only `export { default as Card } from './Card'` statements — would be encountered during the directory scan. There is no exclusion rule for barrel files. Step 6 would attempt to "process" the barrel file, and Step 10's rebuild could accidentally modify it (adding import statements, changing export structure). Modifying a barrel file would break all re-export paths and cause compile errors across the entire project.
- Root Cause: Step 4's scan instructions were written assuming all files in `components/` are UI component files with JSX and classNames. The barrel file pattern (pure re-exports, no JSX) was never considered or excluded.
- Fix Applied: Added "File exclusion rules" section to Step 4 after the conditional scan instructions. Rule 1 covers barrel files: detect files with only `export {...} from '...'` statements and no JSX (no `<` tags, no `className`, no `style=`) — skip them. Applies to any file name (commonly `index.ts`/`index.tsx` but can be any name). Also added exclusion rules for test files (Rule 2, addressed by BUG-015) and type definition files (Rule 3, `*.d.ts`).
- Commit: see cycle 7 commit
- Regression Risk: Any project that uses barrel files for component exports (extremely common in React/Next.js projects). Before the fix, barrel files were at risk of being processed and potentially corrupted during the rebuild phase.

---

### BUG-015: Step 4 has no exclusion rule for test files (*.test.tsx, *.spec.tsx)
- Status: [FIXED 2026-04-21]
- Persona: Cycle 7 E1 — `/ui-restructure --style linear` on project with `Button.test.tsx`
- Command: `/ui-restructure --style linear`
- Skill File: `SKILL.md`
- Line: Step 4 (Scan UI Files), Step 6 (Strip UI Structure)
- Symptom: Step 4 scans `components/` unconditionally. A project with `components/Button.test.tsx` alongside `components/Button.tsx` would have the test file encountered during the scan. Test files contain `describe`, `it`, `expect` blocks, mock implementations, and render calls — NOT UI layout classes. Step 6 would attempt to strip the test file and Step 10 would attempt to rebuild it. This would corrupt test files by removing test assertions, breaking the test suite.
- Root Cause: Step 4's scan scope was never restricted to exclude files by naming convention. Test files (`*.test.*`, `*.spec.*`) are standard in any well-maintained React project and coexist in the same directories as component files. No exclusion was ever defined.
- Fix Applied: Added Rule 2 to the "File exclusion rules" section in Step 4 (added alongside BUG-014 fix): exclude `*.test.tsx`, `*.test.ts`, `*.test.jsx`, `*.test.js`, `*.spec.tsx`, `*.spec.ts`, `*.spec.jsx`, `*.spec.js`, and `__tests__/` directory contents. Also identifies by content pattern: files with `describe(`, `it(`, `test(`, `expect(` as primary content.
- Commit: see cycle 7 commit
- Regression Risk: Any project with a co-located test suite (the most common React testing pattern). Before the fix, test files were at risk of being scanned, stripped of "className" patterns (which are valid in render calls), and corrupted during rebuild.

---

### BUG-016: Step 7 lacks explicit protection for @tailwind directives and non-variable CSS in globals.css
- Status: [FIXED 2026-04-21]
- Persona: Cycle 7 F1 — `/ui-restructure --style apple` on Next.js project with globals.css containing @tailwind directives
- Command: `/ui-restructure --style apple`
- Skill File: `SKILL.md`
- Line: Step 7 (Reset Design Tokens, line ~349), Step 10 (Rebuild UI)
- Symptom: Step 7 identifies `globals.css` as a token file (it contains `:root { --color: ... }` CSS variables) and instructs: "Delete or clear the existing token values. Do NOT delete the file structure — just reset values." However, a typical Next.js `globals.css` also contains `@tailwind base;`, `@tailwind components;`, `@tailwind utilities;` directives, `@media (prefers-color-scheme: dark) { :root { ... } }` blocks, `body { ... }` rules, and `@layer` directives. The instruction "reset values" is ambiguous — a naive implementation could interpret this as rewriting the entire file with only `:root { --color: ; }` blocks, destroying the `@tailwind` directives and causing Tailwind compilation to fail entirely.
- Root Cause: Step 7 was written assuming `globals.css` contains only CSS variable declarations. It did not account for the presence of `@tailwind` directives, `@media` blocks, `body` rules, and `@layer` blocks that must be preserved verbatim. "Do NOT delete the file structure" is insufficient guidance when the file structure includes Tailwind build directives.
- Fix Applied: Added a "globals.css — Protected Content (Non-Negotiable)" subsection to Step 7 with explicit rules: NEVER touch `@tailwind base/components/utilities` directives, NEVER remove `@media` query blocks (only update variable values inside them), preserve `body {}` and other CSS rules, preserve `@layer` directives and `@import` statements. Added a before/after annotated example showing exactly what changes (CSS variable values) vs what is preserved (all directives and rules). The same protection applies in Step 10 during the rebuild/update phase.
- Commit: see cycle 7 commit
- Regression Risk: All Next.js projects (App Router and Pages Router) — `app/globals.css` or `styles/globals.css` is a standard file in every Next.js project and almost always contains `@tailwind` directives. Before the fix, the skill could destroy Tailwind compilation by removing these directives during token reset.

---

### BUG-017: Step 4 has no exclusion rule for "use server" Server Action files
- Status: [FIXED 2026-04-21]
- Persona: Cycle 8 B1 — `/ui-restructure --style apple` on Next.js App Router project with `app/actions/createProject.ts` (Server Action)
- Command: `/ui-restructure --style apple`
- Skill File: `SKILL.md`
- Line: Step 4 (Scan UI Files), File exclusion rules section
- Symptom: Step 4 scans `app/` for App Router projects. Server Action files (e.g., `app/actions/createProject.ts`) begin with `"use server"` and contain only business logic (database calls, `revalidatePath`, form handling) — no JSX, no classNames. The three existing exclusion rules (barrel files, test files, `.d.ts` files) do not cover `"use server"` files: they are not pure re-export barrel files (they have function bodies and `import` statements), not test files, not declaration files. Without an exclusion, the skill scans these files, attempts UI model analysis on them, and could include them in the modified files output — violating the Behavior Rule "NEVER modify: server actions."
- Root Cause: The exclusion rules in Step 4 were designed to catch the common non-UI patterns (barrel re-exports, tests, type declarations). The `"use server"` directive as a scan-exclusion signal was never added. The Behavior Rules protect content from modification but a scan-level exclusion rule was missing.
- Fix Applied: Added Rule 4 to Step 4's "File exclusion rules" section: detect files whose first non-empty line is `"use server"` — these are Server Action files; skip them entirely. Added explanation that Server Components (no directive) are NOT excluded (they may have JSX), only files with `"use server"` as their leading directive are excluded.
- Commit: see cycle 8 commit
- Regression Risk: Any Next.js App Router project using Server Actions (a core Next.js 13+ feature). The `app/actions/` directory pattern is extremely common. Without this fix, server action files would be picked up by the scan even though they contain zero UI content.

---

### BUG-018: Step 4 has no exclusion rule for API route files (app/api/**/route.ts)
- Status: [FIXED 2026-04-21]
- Persona: Cycle 8 C1 — `/ui-restructure --style linear` on Next.js App Router project with `app/api/projects/route.ts`
- Command: `/ui-restructure --style linear`
- Skill File: `SKILL.md`
- Line: Step 4 (Scan UI Files), File exclusion rules section
- Symptom: Step 4 scans `app/` for App Router projects. Next.js App Router API routes live at `app/api/**/route.ts` and export `GET`, `POST`, `PUT`, `DELETE` handler functions. These files contain only server-side HTTP logic — no JSX, no classNames. The three existing exclusion rules do not cover `route.ts` files (they are not barrel files, test files, or `.d.ts` files). Without an exclusion, the skill encounters `route.ts` files during the `app/` scan, could add them to the UI model, process them through Steps 5–6, and potentially include them in the "Files modified" output count — violating the Behavior Rule "NEVER modify: Route handlers / API routes."
- Root Cause: The file exclusion rules in Step 4 addressed naming conventions (barrel, test, .d.ts) but not Next.js App Router's naming convention for API routes (`route.ts`). This is a well-defined Next.js convention where the filename `route.ts` itself signals "API route handler."
- Fix Applied: Added Rule 5 to Step 4's "File exclusion rules" section: detect files named `route.ts`, `route.js`, `route.tsx`, or `route.jsx` in any directory — these are Next.js App Router API route files; skip them entirely. This is a filename-based exclusion that is unambiguous (no other convention uses this exact filename in App Router projects).
- Commit: see cycle 8 commit
- Regression Risk: Any Next.js App Router project with API routes (the standard App Router API pattern). Every `app/api/` directory will contain `route.ts` files. Without this fix, these files would be scanned despite containing no UI content, potentially corrupting the "Files modified" count and output.

---

### BUG-019: Step 4 scans src/ recursively without excluding service layer subdirectories (lib/, utils/, services/)
- Status: [FIXED 2026-04-21]
- Persona: Cycle 8 D1 — `/ui-restructure --style dashboard` on Vite React project with `src/lib/api.ts`, `src/utils/cn.ts`, `src/services/userService.ts`
- Command: `/ui-restructure --style dashboard`
- Skill File: `SKILL.md`
- Line: Step 4 (Scan UI Files), File exclusion rules section
- Symptom: Step 4 always scans `src/`. When a project has service layer files under `src/lib/`, `src/utils/`, `src/services/`, all of these files are encountered during the recursive `src/` scan. These files contain business logic, API utilities, and helper functions — no JSX, no classNames. The existing exclusion rules don't catch them: `src/lib/api.ts` has function bodies and imports (not a barrel), `src/utils/cn.ts` has `twMerge` logic (not a test or .d.ts file). Without a subdirectory exclusion, all service layer files would be scanned and could be processed through Step 5–6, included in the UI model output, or listed as modified files — violating the Behavior Rule "NEVER modify: Service layer files." Additionally, `src/utils/cn.ts` (a classname utility) contains `clsx` and `twMerge` which could be misidentified as class-related UI code.
- Root Cause: Step 4's scan target `src/` was defined as a single directory sweep without any subdirectory exclusion list. In a typical React project, `src/` is a mixed directory containing both UI components (`src/components/`) and non-UI code (`src/lib/`, `src/utils/`, `src/services/`, `src/hooks/`). The distinction between UI subdirectories and non-UI subdirectories was never encoded.
- Fix Applied: Added Rule 6 to Step 4's "File exclusion rules" section: when scanning `src/` or `app/`, skip files inside these known non-UI subdirectory segments: `lib/`, `utils/`, `services/`, `helpers/`, `hooks/` (pure hook files with no JSX), `store/`, `context/`, `config/`, `db/`, `models/`, `types/`. Added exception for `src/components/` and `app/components/` which are always scanned regardless. Added nuance for `hooks/` files: pure hook files (no JSX) are skipped; render hooks that return JSX are scanned.
- Commit: see cycle 8 commit
- Regression Risk: Any React/Next.js project using the `src/` convention with service layer code (extremely common). Without this fix, `src/lib/`, `src/utils/`, `src/services/` files would all be swept into the scan on every restructure, wasting processing cycles and risking incorrect output in the files-modified count.

---

### BUG-020: Step 4 scan is not declared recursive; deeply-nested components and route group directories missed
- Status: [FIXED 2026-04-21]
- Persona: Cycle 9 B1 — `/ui-restructure --style apple` on Next.js App Router project with deeply nested components at depth 4–5 and route group paths
- Command: `/ui-restructure --style apple`
- Skill File: `SKILL.md`
- Line: Step 4 (Scan UI Files), directory scan instructions
- Symptom: Step 4 lists directories to scan (`components/`, `src/`, `layouts/`, `app/`) but never states that scanning is recursive. A naive implementation scanning only the top level of each directory would miss files like `app/dashboard/analytics/components/ReportCard.tsx` (depth 4) and `app/dashboard/analytics/components/charts/LineChart.tsx` (depth 5). Additionally, Next.js App Router route group directories use parenthesized names like `(auth)/` — e.g., `app/(auth)/login/components/LoginForm.tsx`. Without explicit guidance, a scanner might skip `(auth)/` directories entirely or fail to handle the parenthesis naming convention. Both omissions mean deeply nested real-world components would not be restructured.
- Root Cause: Step 4's directory scan instruction was written as a flat list of top-level directories. No explicit statement said "scan recursively" or addressed the Next.js route group `(groupName)/` directory naming convention.
- Fix Applied: (1) Added "recursively — all nested subdirectories included" to the scan headers for both "always scan" and "conditionally scan" blocks. (2) Added an explicit "Scanning is always recursive" paragraph with a depth 4/5 example. (3) Added a "Route group directories" paragraph explaining that `(auth)/`, `(marketing)/` etc. are real directories containing files that must be scanned — treat them as regular directories during recursive scanning, with a concrete path example.
- Commit: see cycle 9 commit
- Regression Risk: Any Next.js App Router project with deeply nested component trees or route groups (extremely common in real Next.js apps). Before the fix, only top-level files in the scan directories would be caught by the skill, leaving all nested components unmodified.

---

### BUG-021: Step 2 framework detection does not handle `src/app/layout.tsx` (App Router inside src/)
- Status: [FIXED 2026-04-21]
- Persona: Cycle 9 C1 — `/ui-restructure --style linear` on Next.js project with `src/app/layout.tsx` instead of `app/layout.tsx` at root
- Command: `/ui-restructure --style linear`
- Skill File: `SKILL.md`
- Line: Step 2 (Detect Framework), detection table
- Symptom: Step 2's detection table maps `app/ directory + layout.tsx` → Next.js App Router. This only checks for `app/` at the project root. Many Next.js projects use the `src/` layout convention where the App Router directory is at `src/app/layout.tsx` — not `app/layout.tsx`. For such projects, Step 2 would find no `app/` at root and fall through to "React (Vite)" or "unknown" — incorrectly detecting the framework. Without correct framework detection, Step 4 would not scan `src/app/` as an App Router directory, and the Server vs. Client Component rules (and all App Router-specific behavior) would not apply.
- Root Cause: The Step 2 detection table was written assuming Next.js always places `app/` at the project root. The `src/app/` convention is officially supported by Next.js 13+ and is widely used, but was never added as a detection signal.
- Fix Applied: (1) Added `src/app/` directory + `layout.tsx` inside `src/app/` as a second row in the detection table, labeled "Next.js App Router (src/ layout convention)". (2) Added a dedicated paragraph explaining the `src/app/` pattern and its scan implications: when detected, Step 4 should use `src/app/` as the App Router scan root instead of `app/`. (3) Added a conflict-resolution rule for `src/app/` + `pages/` coexistence (App Router wins). Updated scan header note to check both project root and `src/` for framework signals.
- Commit: see cycle 9 commit
- Regression Risk: All Next.js projects using the `src/` layout convention (extremely common — the Next.js docs show both root `app/` and `src/app/` as equivalent, and many create-next-app scaffolds default to `src/`). Before the fix, these projects would be misidentified as React (Vite) or unknown.

---

### BUG-022: Step 4 has no exclusion rule for `middleware.ts` files
- Status: [FIXED 2026-04-21]
- Persona: Cycle 9 D1 — `/ui-restructure --style dashboard` on Next.js project with `src/middleware.ts`
- Command: `/ui-restructure --style dashboard`
- Skill File: `SKILL.md`
- Line: Step 4 (Scan UI Files), File exclusion rules section
- Symptom: `middleware.ts` (Next.js middleware) intercepts requests before they reach the app. It contains routing/auth logic, no JSX, no className. While `middleware.ts` at the project root is not directly in any scanned directory and is thus safe, `src/middleware.ts` IS inside the `src/` directory which Step 4 always scans. The existing 6 exclusion rules do not catch `middleware.ts`: it's not a barrel file (has function bodies), not a test file, not a `.d.ts`, not a server action (`"use server"`), not a route file (`route.ts`), and not inside a service layer subdirectory (it lives directly in `src/`). Without an exclusion, `src/middleware.ts` would be scanned, potentially processed through Steps 5–6, and listed in the "Files modified" output — corrupting middleware logic that has no UI classes.
- Root Cause: The `middleware.ts` filename as a scan-exclusion signal was never added to Step 4's exclusion rules. The existing exclusions covered barrel/test/declaration/server-action/route/service-layer patterns but not the middleware filename convention.
- Fix Applied: Added Rule 7 to Step 4's "File exclusion rules" section: detect files named `middleware.ts`, `middleware.js`, `middleware.tsx`, or `middleware.jsx` in any directory — skip them entirely. Applies to both root-level `middleware.ts` and `src/middleware.ts`. Updated the exclusion explanation summary to include point (7).
- Commit: see cycle 9 commit
- Regression Risk: Any Next.js project with `src/middleware.ts` (the alternate middleware location documented by Next.js). Root-level `middleware.ts` was already safe (not in scan dirs), but `src/middleware.ts` was unprotected.

---

### BUG-023: Step 6 has no guidance for `cn()` and `clsx()` className wrapper calls
- Status: [FIXED 2026-04-21]
- Persona: Cycle 9 F1 — `/ui-restructure --style apple` on project using `cn()` (shadcn) and `clsx()` wrappers for conditional classNames
- Command: `/ui-restructure --style apple`
- Skill File: `SKILL.md`
- Line: Step 6 (Strip UI Structure), after template literal className guidance
- Symptom: Many real-world Tailwind projects use `cn()` (from shadcn: `import { cn } from '@/lib/utils'`) or `clsx()` for merging conditional className strings. These appear as `className={cn("base classes", condition && "conditional classes")}` or `className={clsx("base", { "modifier": bool })}`. Step 6's strip guidance only covers plain string `className="..."` and template literal `` className={`...${cond ? 'a' : 'b'}`} ``. Without `cn()`/`clsx()` guidance, a strip pass could: (1) strip the entire `cn(...)` expression as if it were a className string, destroying the function wrapper, (2) fail to recognize `cn()` as a className at all and leave classes unstripped, (3) corrupt the `import { cn } from '@/lib/utils'` import. The wrapper call must be preserved; only the string values inside it should be updated.
- Root Cause: Step 6 was written assuming className values are always plain strings or template literals. The `cn()`/`clsx()` utility wrapper pattern (extremely common in shadcn/Tailwind projects) was never documented or handled.
- Fix Applied: Added a "`cn()` and `clsx()` utility wrapper handling" subsection to Step 6, after the template literal guidance. Rules: (1) NEVER strip the `cn()` or `clsx()` wrapper call itself, (2) NEVER strip the `import { cn }` or `import { clsx }` import statements, (3) treat arguments to `cn()`/`clsx()` the same as className strings — strip layout class values but preserve conditional logic (`variant === 'error' &&`), ternaries, and object syntax (`{ "opacity-50": dismissed }`), (4) during Step 10 rebuild, apply new style engine class values inside the wrapper while keeping the wrapper and all conditional structure intact. Added a complete before/after example showing strip and rebuild behavior.
- Commit: see cycle 9 commit
- Regression Risk: All projects using shadcn/ui (which mandates `cn()` from `@/lib/utils`) or any project using `clsx` for conditional classNames (extremely common in modern React/Tailwind apps). Without this fix, the skill could corrupt className wrapper expressions or fail to update class values inside them.

---

### BUG-024: Step 6 strip guidance does not explicitly protect non-className HTML attributes
- Status: [FIXED 2026-04-21]
- Persona: Cycle 10 B1 — `/ui-restructure --style apple` on component with id, data-testid, tabIndex, aria-*, suppressHydrationWarning, onKeyDown, onFocus, onBlur attributes
- Command: `/ui-restructure --style apple`
- Skill File: `SKILL.md`
- Line: Step 6 (Strip UI Structure), after the "Do NOT strip logic" note
- Symptom: Step 6's strip instruction says "Identify all layout/spacing/typography classes. Remove them." This is scoped to className values in principle, but there was no explicit "NEVER strip these" list for non-className HTML attributes. A naive implementation could accidentally strip: `id="search-wrapper"` (not a layout class but not explicitly protected), `data-testid="search-container"` (data attributes not mentioned), `tabIndex={0}` (could be confused with layout), `suppressHydrationWarning` (not a styling prop but not explicitly protected), `aria-labelledby` / `aria-autocomplete` (ARIA attributes), `onKeyDown` / `onFocus` / `onBlur` (non-onClick event handlers covered by "logic" rules but not enumerated). Without an explicit "NEVER strip" list, any attribute not covered by "logic" or "semantic HTML" could be at risk.
- Root Cause: Step 6 relied on the reader understanding that ONLY className values are stripped. There was no explicit enumeration of the many categories of non-className attributes that must be preserved (form attributes, data-*, aria-*, tabIndex, event handlers beyond onClick, etc.). The Behavior Rules protect "event handlers and callbacks" but do not enumerate the full set of JSX attributes covered by that rule.
- Fix Applied: Added a "Non-className HTML attributes — NEVER strip (Non-Negotiable)" block to Step 6, immediately after the "Do NOT strip logic/tags/keys" note. This block explicitly lists all protected attribute categories: element identity (id, name), type/role, form values, form constraints, input hints, accessibility (all aria-*, htmlFor, tabIndex), data attributes (all data-*), React-specific (key, ref, suppressHydrationWarning), ALL on* event handlers by name, media/link attributes (src, href, alt), and content props (dangerouslySetInnerHTML, style). Ends with: "If uncertain whether an attribute is a layout class or logic — PRESERVE it."
- Commit: see cycle 10 commit
- Regression Risk: All components with form inputs, interactive elements, data attributes, or ARIA attributes. Without this fix, any of these non-className attributes could be silently dropped by a strip pass that focused too broadly.

---

### BUG-025: Step 7 tailwind.config token reset is too broad — no explicit protection for content[], plugins[], safelist[], darkMode
- Status: [FIXED 2026-04-21]
- Persona: Cycle 10 C1 — `/ui-restructure --style linear` on Next.js project with tailwind.config.ts containing content array, safelist, plugins (forms + typography), and darkMode: 'class'
- Command: `/ui-restructure --style linear`
- Skill File: `SKILL.md`
- Line: Step 7 (Reset Design Tokens), tailwind.config section
- Symptom: Step 7's token reset instruction says "Delete or clear the existing token values. Do NOT delete the file structure — just reset values." For globals.css, an explicit "Protected Content" block was added (BUG-016 fix) specifying exactly what NOT to touch (@tailwind directives, @media blocks, body rules). But for tailwind.config.ts, no equivalent protection existed. The instruction "reset values" could be interpreted to mean clearing the entire config object — which would destroy: (1) the `content` array (Tailwind stops generating CSS for all project files), (2) the `plugins` array (all Tailwind plugin functionality breaks), (3) the `safelist` array (dynamically-needed classes stop being generated), (4) the `darkMode` setting (dark mode strategy resets to default), (5) plugin import statements at the top of the file. Only `theme.extend.*` values should be reset.
- Root Cause: The tailwind.config handling in Step 7 was added as a brief line ("tailwind.config.ts (theme.extend section)") in the token files list, but no explicit "protected fields" block was ever written — unlike globals.css which got a full protection section. The modes/theme.md "For Tailwind projects" strategy correctly scopes to `theme.extend.*` only, but SKILL.md Step 7 itself lacked the same explicitness.
- Fix Applied: Added a "tailwind.config.ts / tailwind.config.js — Protected Fields (Non-Negotiable)" block to Step 7, immediately before the globals.css protected content block. Lists 6 protected fields: (1) content array, (2) plugins array, (3) safelist array, (4) darkMode setting, (5) import statements, (6) presets array. Includes a before/after annotated example showing exactly what changes (only theme.extend values) vs what is preserved (all other config sections). Matches the same format and explicitness as the globals.css protection block.
- Commit: see cycle 10 commit
- Regression Risk: Any Next.js/React project using tailwind.config with non-default settings — which is effectively every production project. The content array especially is always customized in real apps. Without this fix, a token reset could render Tailwind non-functional by clearing the content array.

---

### BUG-027: Step 4 has no exclusion rule for Storybook story files (*.stories.*, *.story.*)
- Status: [FIXED 2026-04-21]
- Persona: Cycle 10 E1 — `/ui-restructure --style minimal` on project with Button.stories.tsx, Card.stories.mdx, Card.story.tsx alongside components
- Command: `/ui-restructure --style minimal`
- Skill File: `SKILL.md`
- Line: Step 4 (Scan UI Files), File exclusion rules section (Rule 2)
- Symptom: Storybook story files (`Button.stories.tsx`, `Card.stories.mdx`, `Card.story.tsx`) co-exist in the same `components/` directories as component files. Step 4 scans `components/` recursively. The existing exclusion rules only cover: barrel files, `*.test.*`/`*.spec.*` test files, `*.d.ts` declarations, `"use server"` server actions, `route.ts` API routes, service layer subdirectories, and middleware files. Story files are NOT covered: they are not pure re-exports (they have `const meta`, `export const Primary`, etc. — not barrel files), they don't contain `describe/it/test/expect` (not caught by content-pattern detection), they are not `.d.ts` files. Step 6 would attempt to strip Storybook files, potentially removing story `args` property values or component `meta.args` that contain className-like strings (e.g., `args: { variant: 'primary' }`). Step 10 could add layout classes to story files, corrupting story configurations.
- Root Cause: BUG-015's fix only covered the `*.test.*` and `*.spec.*` naming conventions. Storybook uses a separate `.stories.*` and `.story.*` convention. These were never added as exclusion patterns. Storybook is extremely common in production component libraries and design systems.
- Fix Applied: Added Rule 2b to Step 4's "File exclusion rules" section, immediately after Rule 2 (test files). Rule 2b covers: `*.stories.tsx/ts/jsx/js`, `*.stories.mdx`, `*.story.tsx/ts/jsx/js`. Detection: filename contains `.stories.` or `.story.` before the extension. Updated the exclusion summary to include "(2b) Storybook story files being treated as UI components."
- Commit: see cycle 10 commit
- Regression Risk: Any project using Storybook for component documentation (extremely common in design systems, component libraries, and production apps). Co-located story files in `components/` directories would be scanned and potentially modified on every restructure, corrupting story configurations.

---

## Cycle History

| Cycle | Date | Personas Run | Pass | Fail | Avg Score | Notes |
|-------|------|-------------|------|------|-----------|-------|
| 1 | 2026-04-21 | 10 | 10 | 0 | 100% | Cycle 1 — all personas passed, 0 bugs found |
| 2 | 2026-04-21 | 10 | 8 | 2 | 83% | Cycle 2 — adversarial — 3 bugs found and fixed (BUG-001, BUG-002, BUG-003) |
| 3 | 2026-04-21 | 15 | 14 | 1 | 97% | Cycle 3 — regression + probes — regressions all passed, 1 new bug (BUG-004) found and fixed |
| 4 | 2026-04-21 | 13 | 9 | 4 | 75% | Cycle 4 — engine audit + untested paths — 4 bugs found and fixed (BUG-005: motion missing all engines; BUG-006: aria-hidden checklist gap; BUG-007: grid list reverse missing; BUG-008: inline styles no guidance) |
| 5 | 2026-04-21 | 19 | 19 | 0 | 100% | Cycle 5 — regression sweep + new combinations — 4 regressions all pass, 10 personas all pass, 5 new combos: 1 doc bug (BUG-009: --prompt + --god-mode undocumented) found and fixed |
| 6 | 2026-04-21 | 20 | 50 | 13 | 79% | Cycle 6 — RSC, plain CSS, cross-imports, mixed styling, God Mode phases — 2 bugs found and fixed (BUG-010: RSC Server/Client Component distinction missing; BUG-011: plain CSS class-value rebuild guidance missing) [63 total assertions: 50 pass, 13 fail] |
| 7 | 2026-04-21 | 26 | 53 | 8 | 87% | Cycle 7 — regressions, output format, layouts/, Framer Motion, barrel files, globals.css, shadcn variants — 4 bugs found and fixed (BUG-013: Framer Motion no detection/guidance; BUG-014: barrel files not excluded; BUG-015: test files not excluded; BUG-016: @tailwind directives unprotected in globals.css) [61 total assertions: 53 pass, 8 fail] |
| 8 | 2026-04-21 | 29 | 57 | 9 | 86% | Cycle 8 — server actions, API routes, service layer, useReducer, .d.ts, config files, dynamic CSS Modules — 3 bugs found and fixed (BUG-017: "use server" files no scan exclusion; BUG-018: API route files no scan exclusion; BUG-019: service layer subdirs no scan exclusion) [66 total assertions: 57 pass, 9 fail] |
| 9 | 2026-04-21 | 30+ | 93 | 0 | 100% | Cycle 9 — deep nesting, src/app/, middleware, useLayoutEffect/forwardRef/memo, cn()/clsx(), ARIA preservation, ref props, multi-export files — 4 bugs found and fixed (BUG-020: scan not recursive + route groups; BUG-021: src/app/ pattern not detected; BUG-022: middleware.ts no exclusion rule; BUG-023: cn()/clsx() wrappers no guidance) [93 total assertions: 93 pass, 0 fail — all bugs fixed in-cycle] |
| 10 | 2026-04-21 | 35+ | 55 | 8 | 87% | Cycle 10 — attribute preservation, tailwind content array, prisma, storybook, React.lazy, dangerouslySetInnerHTML, env vars — FINAL HARDENING — 3 bugs found and fixed (BUG-024: non-className HTML attributes no explicit protection; BUG-025: tailwind.config content/plugins/safelist unprotected; BUG-027: *.stories.* files not excluded) [63 total assertions: 55 pass, 8 fail — all bugs fixed in-cycle] |

---

## How to Read This File

Each bug entry follows this format:

```
### BUG-NNN: [Short title]
- Status: [OPEN | FIXED yyyy-mm-dd | INVESTIGATING]
- Persona: [Which test persona caught it]
- Command: [Exact /ui-restructure command that triggers it]
- Skill File: [engines/apple.md | modes/full.md | SKILL.md | etc.]
- Line: [Approximate line number if known]
- Symptom: [What went wrong — be specific]
- Root Cause: [What in the skill caused it]
- Fix Applied: [What was changed to fix it]
- Commit: [git hash]
- Regression Risk: [What other personas/commands could be affected]
```
