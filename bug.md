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

_None — all Cycle 6 bugs fixed in same cycle._

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

## Cycle History

| Cycle | Date | Personas Run | Pass | Fail | Avg Score | Notes |
|-------|------|-------------|------|------|-----------|-------|
| 1 | 2026-04-21 | 10 | 10 | 0 | 100% | Cycle 1 — all personas passed, 0 bugs found |
| 2 | 2026-04-21 | 10 | 8 | 2 | 83% | Cycle 2 — adversarial — 3 bugs found and fixed (BUG-001, BUG-002, BUG-003) |
| 3 | 2026-04-21 | 15 | 14 | 1 | 97% | Cycle 3 — regression + probes — regressions all passed, 1 new bug (BUG-004) found and fixed |
| 4 | 2026-04-21 | 13 | 9 | 4 | 75% | Cycle 4 — engine audit + untested paths — 4 bugs found and fixed (BUG-005: motion missing all engines; BUG-006: aria-hidden checklist gap; BUG-007: grid list reverse missing; BUG-008: inline styles no guidance) |
| 5 | 2026-04-21 | 19 | 19 | 0 | 100% | Cycle 5 — regression sweep + new combinations — 4 regressions all pass, 10 personas all pass, 5 new combos: 1 doc bug (BUG-009: --prompt + --god-mode undocumented) found and fixed |
| 6 | 2026-04-21 | 20 | 50 | 13 | 79% | Cycle 6 — RSC, plain CSS, cross-imports, mixed styling, God Mode phases — 2 bugs found and fixed (BUG-010: RSC Server/Client Component distinction missing; BUG-011: plain CSS class-value rebuild guidance missing) [63 total assertions: 50 pass, 13 fail] |

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
