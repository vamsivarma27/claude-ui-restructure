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

_None — all Cycle 3 bugs fixed in same cycle._

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

## Cycle History

| Cycle | Date | Personas Run | Pass | Fail | Avg Score | Notes |
|-------|------|-------------|------|------|-----------|-------|
| 1 | 2026-04-21 | 10 | 10 | 0 | 100% | Cycle 1 — all personas passed, 0 bugs found |
| 2 | 2026-04-21 | 10 | 8 | 2 | 83% | Cycle 2 — adversarial — 3 bugs found and fixed (BUG-001, BUG-002, BUG-003) |
| 3 | 2026-04-21 | 15 | 14 | 1 | 97% | Cycle 3 — regression + probes — regressions all passed, 1 new bug (BUG-004) found and fixed |

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
