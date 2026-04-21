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

_None — all Cycle 17 bugs fixed in same cycle._

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

### BUG-029: Step 7 globals.css protection list does not explicitly cover @font-face and @keyframes blocks
- Status: [FIXED 2026-04-21]
- Persona: Cycle 11 D1 — `/ui-restructure --style minimal` on project with globals.css containing @font-face, @keyframes fadeIn, @keyframes spin, and .animate-fade-in class rule
- Command: `/ui-restructure --style minimal`
- Skill File: `SKILL.md`
- Line: Step 7 (Reset Design Tokens), globals.css Protected Content block
- Symptom: The Step 7 "globals.css — Protected Content (Non-Negotiable)" block listed 5 protected items: @tailwind directives, @media blocks, body/CSS rules, @layer directives, and @import statements. Item 3 said "body {} and other CSS rules — preserve all non-variable CSS rules." However, `@font-face` and `@keyframes` are CSS @-rules (not standard ruleset blocks) and were NOT explicitly named. A naive or strict implementation could interpret "other CSS rules" as covering only standard selector-based ruleset blocks (`.card { ... }`, `body { ... }`) but NOT at-rules with their own inner block syntax. Without explicit protection, @font-face blocks (custom web font definitions) and @keyframes blocks (animation definitions) could be stripped or corrupted during token reset. Additionally, standard CSS classes that REFERENCE keyframes (e.g., `.animate-fade-in { animation: fadeIn 200ms ease-out; }`) were also not explicitly protected, creating risk for any animation utility class.
- Root Cause: The globals.css protection list was written focused on Tailwind-specific concerns (@tailwind directives) and CSS variable handling (body rule, @media dark mode). CSS animation infrastructure — @font-face for web fonts and @keyframes for animations — was never explicitly added to the protection list. The broad "other CSS rules" wording is insufficient because @-rules have distinct syntax from standard rulesets and could be missed by a strict implementation.
- Fix Applied: Added items 6 and 7 to the globals.css Protected Content list in Step 7: (6) `@font-face` blocks — explicitly listed as font loading declarations that must be fully preserved; (7) `@keyframes` blocks — explicitly listed as animation definitions that must be fully preserved including all keyframe stops. Also updated the annotated example to show @font-face, @keyframes fadeIn, and .animate-fade-in all with "PRESERVE entire block" annotations. Changed item 3 wording from "body {} and other CSS rules" to "body {} and other standard CSS rules" for clarity.
- Commit: see cycle 11 commit
- Regression Risk: Any Next.js/React project using web fonts via @font-face in globals.css (common with self-hosted fonts for performance) or custom CSS animations via @keyframes (common for loading spinners, fade-ins, shimmer effects). Before the fix, these blocks could be stripped on token reset, breaking web font loading and all CSS animation references.

---

### BUG-031: `prompts/restructure-flow.md` critically out of sync with SKILL.md — Vue `<style scoped>` contradiction + missing all BUG-002 through BUG-029 protections
- Status: [FIXED 2026-04-21]
- Persona: Cycle 12 QA Pre-Flight — restructure-flow.md audit
- Command: any `/ui-restructure` command (the flow file is used for all executions)
- Skill File: `prompts/restructure-flow.md`
- Line: Multiple sections — Vue note (line ~290), Phase 2 strip list, Phase 3 token reset, Phase 1 scan, missing God Mode and Polish Pass sections
- Symptom: `restructure-flow.md` had 8+ critical divergences from `SKILL.md` (the authoritative source). (1) **Vue `<style scoped>` CONTRADICTION**: the file said `<style scoped>` → "replace with new token-based styles" — directly contradicting BUG-002's fix which requires preserving scoped styles as-is. (2) Phase 2 "Strip only these patterns" listed only Tailwind classes — missing inline styles (BUG-008), cn()/clsx() wrapper preservation (BUG-023), Vue SFC `class=` attribute handling (BUG-002). (3) Phase 2 "Never strip" list was missing all non-className HTML attributes: id, data-*, tabIndex, aria-*, onKeyDown/onFocus/onBlur, ref, htmlFor, required, disabled, type, name, placeholder, maxLength, value, checked, defaultValue, suppressHydrationWarning, spread props, React.Fragment — all added by BUG-024. (4) Phase 1 scan had no `src/app/` detection pattern (BUG-021) and no "Excluded files" section (BUG-014 through BUG-022: barrel files, test files, stories files, .d.ts, API routes, server actions, service layer subdirs, middleware.ts). (5) Phase 3 Token Reset did not protect `@keyframes`/`@font-face` blocks (BUG-029) and did not protect tailwind.config `content`/`plugins`/`safelist`/`darkMode` fields (BUG-025). (6) No Phase 0 God Mode check — the entire god-mode path was missing from the flow file. (7) No Phase 5.5 Polish Pass — Step 11 (ui-craft.md polish pass) was entirely absent. (8) Phase 7 report format differed from SKILL.md Step 11 output format.
- Root Cause: `restructure-flow.md` was written before Cycles 2–11 added protections via BUG-002 through BUG-029. The flow file was never updated as bugs were fixed in SKILL.md, causing it to diverge over 11 cycles of fixes. It became a snapshot of the Cycle 1 state of the skill.
- Fix Applied: Complete rewrite of `restructure-flow.md` synchronized with SKILL.md: (1) Added Phase 0 God Mode check (skip Phases 1–7 if --god-mode present, read user-mindset.md + godmode.md, run 7-phase God Mode pipeline). (2) Rewrote Phase 1 scan with `src/app/` detection, full exclusion rules section (all 7 rules: barrel files, test files, storybook files, .d.ts, server actions, API routes, service layer subdirs, middleware.ts), recursion note, route group note, Server/Client Component recording. (3) Phase 2 strip list updated to include inline styles handling, cn()/clsx() wrapper handling section, Vue SFC handling section with CORRECT `<style scoped>` instruction (preserve as-is — not replace). (4) Phase 2 "Never strip" section completely rewritten with full non-className HTML attribute list (all categories from BUG-024). (5) Phase 3 Token Reset updated with tailwind.config Protected Fields (content, plugins, safelist, darkMode, imports, presets) and globals.css Protected Content (`@font-face`, `@keyframes`, all 7 protection items). (6) Added Phase 5.5 Polish Pass (full ui-craft.md checklist: motion with Framer Motion branch, states, typography, icons, spacing, accessibility). (7) Phase 7 report format aligned with SKILL.md Step 11 output. (8) Vue 3 framework note corrected to say `<style scoped>` → preserve as-is.
- Commit: see cycle 12 commit
- Regression Risk: All commands — `restructure-flow.md` is the master execution prompt referenced for all restructure operations. The Vue `<style scoped>` contradiction was a direct correctness bug that would destroy scoped styles in any Vue 3 project. The missing exclusion rules meant any execution referencing this flow file instead of SKILL.md directly would miss all scan exclusions from Cycles 7–9.

---

### BUG-033: Step 7 `@layer` protection too broad — `@apply` inside `@layer components {}` not rebuilt with engine values
- Status: [FIXED 2026-04-21]
- Persona: Cycle 13 B1 — `@apply in @layer components` probe
- Command: `/ui-restructure --style apple` on Next.js App Router project with globals.css containing `@layer components { .btn-primary { @apply flex items-center gap-2 px-4 py-2 bg-blue-500 text-white rounded-md; } }`
- Skill File: `SKILL.md`
- Line: Step 7 globals.css Protected Content Item 4; Step 10 rebuild section
- Symptom: Step 7 Item 4 said "`@layer base/components/utilities` must be preserved." This over-protected the contents of `@layer components {}` blocks — any custom class definitions using `@apply` with hardcoded Tailwind layout/color class names were left completely unchanged during token reset and rebuild. After restructure: `:root { }` CSS variables updated correctly to apple values, but `.btn-primary { @apply flex gap-2 px-4 py-2 bg-blue-500 rounded-md; }` retained the old pre-apple Tailwind classes. The restructured project therefore applied engine styles to JSX classNames but not to `@layer components` `@apply` definitions — producing a visually inconsistent output where component-class-based buttons looked nothing like the rebuilt JSX-class-based components.
- Root Cause: The `@layer` protection was written to prevent the `@layer` DIRECTIVE WRAPPER from being removed (e.g., accidentally deleting `@tailwind utilities;` or `@layer base { }` base reset rules). But "must be preserved" was written too broadly — it protected the entire block CONTENTS including `@apply` rules. `@apply` values inside `@layer components {}` are design-system token definitions, the same category as `className` values in JSX — they MUST be rebuilt with engine values when tokens are reset.
- Fix Applied: (1) Updated Step 7 globals.css Protected Content Item 4 to distinguish between the `@layer` wrapper (always preserved) and `@apply` values inside `@layer components {}` (rebuilt in Step 10 — same as className values). Added explicit rule: preserve selector names and the `@apply` keyword; update the Tailwind class values. Exception: `@layer base {}` HTML element reset rules preserved fully. CSS variable `@apply` references (e.g., `@apply bg-[--color-primary]`) do not need updating. (2) Added `@apply in @layer components {} rebuild` subsection to Step 10 with before/after example. (3) Updated restructure-flow.md Phase 3 Item 4 and Phase 4 token targets to match.
- Commit: see cycle 13 commit
- Regression Risk: Any project using `@layer components { }` blocks in globals.css with hardcoded Tailwind class names (common in projects following Tailwind's official "Extracting Components" pattern). Before the fix, these projects would get JSX-level visual changes but component-class definitions would remain at old values — producing an inconsistent partially-redesigned output.

---

### BUG-034: Step 4 has no exclusion rule for `instrumentation.ts` / `instrumentation.node.ts` files
- Status: [FIXED 2026-04-21]
- Persona: Cycle 13 B2 — `instrumentation.ts` scan exclusion probe
- Command: `/ui-restructure --style minimal` on Next.js App Router project with `src/instrumentation.ts`
- Skill File: `SKILL.md`
- Line: Step 4 (Scan UI Files), File exclusion rules section (after Rule 7)
- Symptom: `src/instrumentation.ts` (Next.js 14+ observability setup — exports `register()` function for OpenTelemetry / Sentry / Datadog initialization) lives directly in `src/`. Step 4 always scans `src/` recursively. The seven existing exclusion rules do not match `instrumentation.ts`: it is not a barrel file (has a function body), not a test file, not a storybook file, not a `.d.ts`, does not start with `"use server"`, is not named `route.ts`, is not inside `lib/`/`utils/`/`services/` subdirectories, and is not named `middleware.ts`. The file would be scanned on every restructure. While it has no JSX or className (so the strip/rebuild passes would not modify it), the scan is unnecessary and creates a risk that a liberal implementation adds the file to "Files modified" output or attempts to "improve" the observability setup code — violating Behavior Rules.
- Root Cause: The filename-based exclusion rules covered barrel/test/storybook/type-declaration/server-action/route/middleware patterns, but not the `instrumentation.ts` / `instrumentation.node.ts` naming convention (a Next.js 14+ convention for the `register()` hook used by OpenTelemetry integrations). This gap was first noted in the Cycle 11 audit (F3 minor note) but carried forward to Cycle 13 for a definitive fix.
- Fix Applied: Added Rule 8 to Step 4's "File exclusion rules" section: detect files named `instrumentation.ts`, `instrumentation.js`, `instrumentation.node.ts`, or `instrumentation.node.js` in any directory — skip them entirely. Applies to root-level and `src/instrumentation.ts`. Updated the exclusion summary to include "(8) instrumentation files being scanned despite containing no UI." Also updated `restructure-flow.md` Phase 1B exclusion rules with the same Rule 8.
- Commit: see cycle 13 commit
- Regression Risk: Any Next.js 14+ project with `src/instrumentation.ts` (the standard location for OpenTelemetry / Sentry Next.js SDK integration). Root-level `instrumentation.ts` is not inside any scan directory so was already safe, but `src/instrumentation.ts` was unprotected.

---

### BUG-035: Step 6 and Step 10 have no guidance for `cva()` (class-variance-authority) variant definitions
- Status: [FIXED 2026-04-21]
- Persona: Cycle 14 B1 — `cva()` variant definitions probe (shadcn/ui Button pattern)
- Command: `/ui-restructure --style apple` on Next.js App Router project with `components/ui/button.tsx` using `cva()` for buttonVariants
- Skill File: `SKILL.md`
- Line: Step 6 (Strip UI Structure), Step 10 (Rebuild UI)
- Symptom: Step 6's strip pass targets `className=` JSX attributes only. `cva()` calls are module-level function expressions — they contain base class strings and variant class strings (e.g., `"bg-primary text-primary-foreground hover:bg-primary/90"`) but they are NOT `className=` attributes. Step 6 never touches them. Step 10's rebuild targets JSX `className=` attributes, plus `@apply` values and `:root {}` CSS variables — but NOT `cva()` call arguments. After a `/ui-restructure --style apple` run on a shadcn/ui project: `:root {}` CSS variables and JSX classNames updated to apple values, but all `buttonVariants = cva(...)` definitions retained old pre-apple class strings. The button's visual output is driven entirely by `cva()` return values — so the button looked identical before and after the restructure, despite the JSX `className` expression being "updated." 7/10 assertions failed (B1-V1, B1-V2, B1-V3 — all base and variant class values unchanged).
- Root Cause: `cva()` is a module-level className builder (used in shadcn/ui for every component: Button, Badge, Alert, Input, etc.). The skill was designed around JSX `className=` attribute stripping and never considered module-level class string definitions. `cva()` was introduced by shadcn/ui as the standard way to define multi-variant component class strings — it is extremely common in modern Next.js apps using shadcn.
- Fix Applied: (1) Added `cva()` detection + flag section to Step 6: detect `import { cva } from "class-variance-authority"` in scanned files; flag for Step 10 rebuild; do NOT strip `cva()` calls during Step 6. (2) Added `cva()` rebuild subsection to Step 10: update base class string + all variant VALUE strings with engine values; preserve all variant NAMES, `defaultVariants`, `VariantProps` types, and cva import. Includes before/after example. (3) Added same guidance to restructure-flow.md Phase 2 (flag) and Phase 5 (rebuild). Also added `twMerge()` direct usage note to both SKILL.md and restructure-flow.md (same rules as `cn()` — never strip wrapper, treat string args as className strings).
- Commit: see cycle 14 commit
- Regression Risk: All projects using shadcn/ui (which uses `cva()` for every UI primitive: Button, Badge, Alert, Card, Dialog, Input, Select, etc.). Any project importing `class-variance-authority` would have unupdated cva() definitions after a restructure. This affects the majority of modern Next.js apps using the shadcn/ui component library.

---

### BUG-036: Step 4 Rule 6 excludes `context/` entirely — provider components with JSX (ThemeProvider, LayoutProvider) never scanned
- Status: [FIXED 2026-04-21]
- Persona: Cycle 15 B1 — `context/ThemeProvider.tsx` with JSX probe
- Command: `/ui-restructure --style apple` on Next.js App Router project with `src/context/ThemeContext.tsx` containing a `ThemeProvider` component with `className="min-h-screen bg-background text-foreground font-sans antialiased transition-colors duration-300"`
- Skill File: `SKILL.md`
- Line: Step 4 (Scan UI Files), File exclusion rules Rule 6
- Symptom: Rule 6 lists `context/` as a service layer directory to skip entirely. A file at `src/context/ThemeContext.tsx` was excluded from the scan even though it contained a `ThemeProvider` React component with JSX and layout classNames. After restructure: all JSX components in `components/` and `app/` were updated to apple style, but the layout wrapper inside `ThemeProvider` (`<div className="min-h-screen bg-background ...">`) was untouched — retaining old values and potentially creating a visual inconsistency (the ThemeProvider outer div determines the page-level background and font settings for the whole app).
- Root Cause: Rule 6 was added (BUG-019 fix) to exclude non-UI service layer code from scan. The `context/` exclusion was intended for pure context definition files (`createContext`, `useContext`, type exports). However, many React codebases put React **provider components** in `context/` directories — these providers render JSX layout wrappers that need to be restructured. The `hooks/` exception (scan hooks with JSX, skip pure hooks) was never extended to `context/`.
- Fix Applied: Added a `context/` exception to Rule 6 that mirrors the `hooks/` exception: `context/` files that contain JSX (React provider components with `<` tags and `className` props) — scan these. Pure context definition files (only `createContext`, `useContext`, type definitions, no JSX) — skip. Detection: if the file contains any JSX — scan it; otherwise skip.
- Commit: see cycle 15 commit
- Regression Risk: Any project with React provider components in a `context/` directory (extremely common pattern — `ThemeProvider`, `AuthProvider`, `LayoutProvider`, `ToastProvider` etc.). These providers frequently render layout-level wrappers (min-h-screen, bg-background, font settings) that need updating on every restructure. Before the fix, these files were silently skipped, leaving top-level layout shells with old design values.

---

### BUG-037: Step 10 Plain CSS rebuild has no guidance for `@apply` directives in standalone CSS files
- Status: [FIXED 2026-04-21]
- Persona: Cycle 16 B1 — `@apply` in standalone CSS file probe (`src/styles/button.css`)
- Command: `/ui-restructure --style apple` on Next.js App Router + Tailwind project with `src/styles/button.css` containing `.btn-primary { @apply flex items-center gap-2 px-4 py-2 bg-blue-500 text-white rounded-md; }`
- Skill File: `SKILL.md`
- Line: Step 10 (Rebuild UI), Plain CSS rebuild section
- Symptom: Step 10 "Plain CSS rebuild" documents updating "property VALUES inside each rule block" — with examples showing standard CSS properties (padding: 16px; background: #fff; border-radius: 8px). `@apply` is a Tailwind preprocessor directive (`@apply flex gap-2 px-4 py-2 bg-blue-500 rounded-md`), NOT a standard CSS property-value pair. The Step 10 guidance for plain CSS only addressed standard CSS properties, not `@apply` directives. After restructure: JSX classNames updated to apple values, globals.css `:root{}` variables updated, `@layer components{}` @apply rebuilt (BUG-033 fix) — but `src/styles/button.css` `.btn-primary { @apply ... }` left with old pre-apple Tailwind class values. Components using `className="btn-primary"` would visually remain in old style.
- Root Cause: The BUG-033 fix addressed `@apply` in `globals.css @layer components {}` blocks specifically. Standalone CSS files using `@apply` (a common Tailwind pattern for component CSS extraction, e.g., `.btn { @apply flex gap-2 bg-blue-500 rounded-md; }`) were not covered in the Step 10 plain CSS rebuild guidance. The "update property VALUES" instruction only applies to standard CSS properties, not Tailwind's `@apply` directive.
- Fix Applied: Added `@apply` guidance to Step 10 Plain CSS rebuild section (before the Rules block): when any CSS file contains `@apply` directives inside class rules, update the Tailwind class values after `@apply` with style engine values — the same way as `className` in JSX. Preserve: selector names, `@apply` keyword, non-Tailwind CSS properties, CSS custom property references in `@apply`. Updated the Rules block to include the `@apply` update rule. Also updated restructure-flow.md Phase 4 token targets and Phase 5 rebuild section with equivalent guidance. Also added `useCallback`, `useMemo`, `useRef`, `useContext` as explicit items to the Phase 1C logic audit checklist and Step 5 PRESERVE list (previously only covered implicitly by "custom hooks").
- Commit: see cycle 16 commit
- Regression Risk: Any project using Tailwind's `@apply` in standalone CSS component files (a documented Tailwind pattern for extracting component classes). Common in projects that prefer CSS files over JSX utility classes for component styling, or transitioning from CSS-based to utility-based styling. Before the fix, these files would have their @apply class values left unchanged after restructure, producing visually inconsistent output (JSX updated but CSS file styles unchanged).

---

### BUG-038: `@layer utilities {}` contents not explicitly protected — spec ambiguity could cause incorrect rebuild
- Status: [FIXED 2026-04-21]
- Persona: Cycle 17 B1 — `@layer utilities {}` vs `@layer components {}` differentiation probe
- Command: `/ui-restructure --style apple` on Next.js App Router + Tailwind project with `globals.css` containing both `@layer utilities { .no-scrollbar { ... } .flex-center { @apply flex items-center justify-center; } }` and `@layer components { .btn { @apply flex gap-2 px-4 py-2 bg-blue-500 rounded-md; } }`
- Skill File: `SKILL.md`
- Line: Step 7 (Reset Design Tokens), globals.css Protected Content, Item 4
- Symptom: The BUG-033 fix (Cycle 13) explicitly distinguished `@layer base {}` (fully preserved) from `@layer components {}` (wrapper preserved, `@apply` values rebuilt). However, it never mentioned `@layer utilities {}`, leaving an ambiguity: an implementation reading the spec could reasonably treat `@layer utilities {}` like `@layer components {}` (rebuild `@apply` values) instead of like `@layer base {}` (fully preserve). This would incorrectly rebuild utility helper classes like `.no-scrollbar`, `.flex-center`, `.text-balance` — single-purpose helpers that are not component design token definitions. Score before fix: 4/6 assertions fail (67%).
- Root Cause: The three-layer taxonomy (`@layer base`, `@layer components`, `@layer utilities`) was never explicitly documented in the spec. BUG-033 covered only the two layers it needed to distinguish — base vs components — without stating the rule for the third layer (utilities). This spec gap left the behavior for `@layer utilities {}` undefined.
- Fix Applied: Added explicit `@layer utilities {}` protection clause to Step 7 Item 4: "Fully preserve `@layer utilities {}` contents as-is — custom utility classes are single-purpose helper utilities, NOT component design token definitions. NEVER rebuild `@layer utilities {}` contents with engine values." Added three-way rule summary: `@layer base {}` → fully preserved; `@layer utilities {}` → fully preserved; `@layer components {}` → wrapper preserved, `@apply` values rebuilt. Updated `restructure-flow.md` Phase 3 Item 4 to match.
- Commit: see cycle 17 commit
- Regression Risk: Any project using Tailwind `@layer utilities {}` in globals.css (a documented Tailwind pattern for custom utility classes). Common classes: `.no-scrollbar` (hide scrollbars), `.flex-center` (flex shorthand), `.text-balance` (balanced text wrapping), `.overlay` (absolute overlay). Before the fix, the spec was ambiguous and an implementation could have incorrectly treated these like component classes and rebuilt them with engine values.

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
| 11 | 2026-04-21 | 35+ | 56 | 9 | 86% | Cycle 11 — next/image, next/link, @keyframes/@font-face, next/font, spec consistency audit, --mode full explicit — 1 bug found and fixed (BUG-029: @font-face/@keyframes not explicitly protected in globals.css Step 7); BUG-028 and BUG-030 NOT triggered; F3 minor doc note (instrumentation.ts gap, low severity) [65 total assertions: 56 pass, 9 at-risk-before-fix — all resolved in-cycle] |
| 12 | 2026-04-21 | 35+ | 62 | 0 | 100% | Cycle 12 — restructure-flow.md sync (BUG-031), class components, import type, spread props, Fragment, SSR guard, async RSC — 1 bug fixed (BUG-031: restructure-flow.md critically out of sync — Vue scoped contradiction + 11 missing protections); all Parts B–G assertions passed [62 total assertions: 62 pass, 0 fail — all resolved in-cycle] |
| 13 | 2026-04-21 | 12 | 116 | 4 | 97% | Cycle 13 — full regression suite (10/10 pass, 100%) + new probes (2 bugs found and fixed): BUG-033 (@apply in @layer components not rebuilt — @layer protection too broad), BUG-034 (instrumentation.ts no scan exclusion rule). restructure-flow.md updated in sync. [120 total: 110 Part-A pass + 4/6 B1 pass + 2/2 B2 pass = 116 pass, 4 fail before fix → 0 fail after fix] |
| 14 | 2026-04-21 | 13 | 122 | 3 | 98% | Cycle 14 — full regression suite (10/10 pass, 100%) + new probes (1 high-severity bug found and fixed): BUG-035 (cva() variant definitions in shadcn/ui components not rebuilt — module-level class strings invisible to strip/rebuild pass). Also added twMerge() explicit protection note. Probe B2 (next/font className) PASSED with no changes needed. [125 total: 112 Part-A + 7/10 B1 before fix + 4/4 B2 + 2/2 B3 = 122 pass before fix → 125 after fix] |
| 15 | 2026-04-21 | 13 | 118 | 6 | 95% | Cycle 15 — full regression suite (10/10 pass, 100%) + new probes: BUG-036 (context/ excluded entirely — ThemeProvider/LayoutProvider JSX not scanned). Also added globals.css path resolution note for src/ convention and Tailwind v4 @import coverage confirmed (PASS). [124 total: 112 Part-A + 0/6 B1 before fix + 4/4 B2 + 2/2 B3 = 118 pass before fix → 124 after fix] |
| 16 | 2026-04-21 | 14 | 124 | 3 | 98% | Cycle 16 — full regression suite (10/10 pass, 100%) + new probes: BUG-037 (@apply in standalone CSS files not rebuilt — Plain CSS rebuild guidance covered standard CSS properties but not @apply directive). Added useCallback/useMemo/useRef/useContext to explicit PRESERVE list. Probes B2–B4 (useCallback, providers.tsx, compound components) all PASSED. [127 total: 112 Part-A + 3/6 B1 before fix + 3/3 B2 + 4/4 B3+B4 = 124 pass → 127 after fix] |
| 17 | 2026-04-21 | 13 | 118 | 4 | 97% | Cycle 17 — full regression suite (10/10 pass, 100%, 112/112 assertions) + new probes: BUG-038 (@layer utilities {} protection ambiguous — spec only covered base vs components, never stated utilities are fully preserved). Probes B2 (string concat className) and B3 (React.memo + displayName) PASSED. [122 total: 112 Part-A + 4/6 B1 before fix + 2/2 B2 + 2/2 B3 = 118 pass before fix → 122 after fix] |

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
