# God Mode — User-First UI Redesign

In God Mode, the user is God. You are not a developer. You are not a designer. You are 100 different users arriving at this product for the first time — and you are redesigning every page so that every one of those 100 users succeeds without friction, without confusion, without needing to think.

**Default behavior:** Tokens are preserved. Only layout, hierarchy, placement, and visual weight change.
**To also reset tokens:** Add `--remove-tokens` to the command.

---

## The Core Mindset Shift

Stop thinking like this:
```
"Where does this component go in the DOM?"
"What's the cleanest component structure?"
"How do I organize this data?"
```

Start thinking like this:
```
"I just landed on this page. What do I want to DO?"
"Where is my eye going first? Is that the right thing?"
"What is confusing me right now?"
"What am I looking for and why can't I find it?"
"Why does this button not look clickable?"
"Why does this page feel overwhelming?"
```

---

## The 100-User Mindset Model

Before touching a single file, simulate these 100 users mentally. Each has a goal. Does your current UI serve them?

**First-time visitors (20 users)**
- Don't know the product
- Scanning to understand: "What is this? Can I trust it? What do I do first?"
- Need: Clear value statement visible immediately. One obvious next action.

**Goal-oriented users (30 users)**
- Know what they want ("I need to find X" / "I need to do Y")
- Will not read anything — they scan for the target
- Need: Search prominent. Filters accessible. Primary action obvious.

**Returning users (25 users)**
- Have context, want speed
- Need: Last action resumable. Quick navigation. No re-learning required.

**Mobile users (15 users)**
- One thumb. Small screen. Interrupted context.
- Need: Touch targets large enough. Primary action reachable with thumb. No horizontal scroll.

**Stressed/rushed users (10 users)**
- Under time pressure
- Will rage-quit if confused within 3 seconds
- Need: Zero ambiguity. Fastest path to goal visible immediately.

---

## Phase 1 — Full Page Inventory

Scan every page file in:
- `app/` (Next.js App Router)
- `pages/` (Next.js Pages Router)
- `src/views/` or `src/pages/` (Vue/React)

For each page, build this inventory:

```
Page: [filename / route]
─────────────────────────────
Navigation elements:    [list: links, tabs, breadcrumbs, back buttons]
Primary actions:        [list: main CTA buttons, submit, create, save]
Secondary actions:      [list: edit, delete, export, share, cancel]
Destructive actions:    [list: delete, remove, reset, revoke]
Search / Filter:        [present? prominent? hidden?]
Content blocks:         [list: cards, tables, lists, grids, text sections]
Forms:                  [list: inputs, selects, checkboxes, radios]
Status / Feedback:      [list: alerts, badges, loading states, empty states]
Modals / Drawers:       [triggered by what?]
Above-the-fold content: [what is visible without scrolling on 1440px viewport]
```

Complete this inventory for EVERY page before making any changes.

---

## Phase 2 — User Friction Audit

For each page, score every element on two axes:

**Importance to user** (how often do users need this?)
```
5 = Always — the reason they came to this page
4 = Often — used in most sessions
3 = Sometimes — occasional need
2 = Rarely — edge case, power user
1 = Almost never — administrative or system-level
```

**Current visual prominence** (how loud is it visually?)
```
5 = Dominates the page
4 = Easily spotted
3 = Findable with scanning
2 = Easy to miss
1 = Hidden or buried
```

**The friction formula:**
```
Friction = |Importance - Visual Prominence|

High friction examples:
  Primary CTA: Importance=5, Prominence=2 → Friction=3 (CRITICAL: user can't find main action)
  Delete button: Importance=2, Prominence=4 → Friction=2 (DANGEROUS: destructive action too loud)
  Search: Importance=4, Prominence=1 → Friction=3 (CRITICAL: user can't search)
  Legal footer link: Importance=1, Prominence=4 → Friction=3 (noise)
```

List all elements with Friction ≥ 2. These are your redesign targets.

---

## Phase 3 — The 6 UX Principles Evaluation

Evaluate every page against all 6 principles. Score 1–5. Document failures.

### Principle 1: Usability
*"Can the user accomplish their goal efficiently, without errors, without needing help?"*

Check:
- Is the primary action obvious without reading anything?
- Can a new user complete the main task in under 60 seconds?
- Are form labels clear? Are placeholders used as labels (anti-pattern)?
- Do buttons say what they DO? ("Submit" is bad. "Save Changes" is good. "Create Project" is better.)
- Is the error state informative? ("Error" is bad. "Email already in use — try logging in instead" is good.)

Failures to fix:
```
✗ Generic button labels ("Submit", "OK", "Cancel")
✗ Placeholder text used instead of visible labels
✗ Required fields not indicated before submission
✗ Inline validation missing (errors only on submit)
✗ Primary action not visually distinct from secondary actions
```

### Principle 2: Findability
*"Can the user find what they're looking for without asking for help?"*

Check:
- Is search available if the page has more than 8 items?
- Are navigation labels in user language (not developer/database language)?
- Is the current page indicated in navigation?
- Are filters visible without scrolling if the page is filterable?
- Is related content surfaced near where the user needs it?

Failures to fix:
```
✗ Search buried below the fold on content-heavy pages
✗ Navigation labels named after DB models ("UserEntity", "ProductRecord")
✗ Filters hidden inside a collapsed panel with no indication they exist
✗ Pagination without "showing X of Y results" context
✗ No breadcrumb on deep pages
```

### Principle 3: Accessibility
*"Can every user, regardless of ability, use this?"*

Check:
- Does every interactive element have a visible focus state?
- Is color used as the only means to communicate state? (never acceptable)
- Are icon-only buttons labeled with `aria-label`?
- Is text contrast WCAG AA compliant?
- Are touch targets at least 44×44px?
- Is tab order logical (matches visual reading order)?

Failures to fix:
```
✗ Red/green only status indicators (colorblind failure)
✗ Icon buttons with no aria-label
✗ focus:outline-none with no replacement focus style
✗ Click targets smaller than 44px
✗ Form inputs without associated <label> elements
```

### Principle 4: Desirability
*"Does using this product feel good? Does it feel alive? Does it feel trustworthy?"*

Check:
- Does every interaction give feedback? (button click, form submit, data load)
- Is there micro-animation on state changes or does everything snap?
- Does the page feel trustworthy — or cluttered, inconsistent, amateur?
- Do empty states have personality — or are they blank voids?
- Is whitespace used intentionally — or does content feel stuffed?

Failures to fix:
```
✗ Buttons with no active/pressed state (feel broken)
✗ No loading feedback during async operations (user double-clicks)
✗ Empty tables/lists show nothing (users think the page is broken)
✗ All spacing is identical (no breathing room between major sections)
✗ No transition on any state change (jarring)
```

### Principle 5: Learnability (Consistency)
*"Once a user learns one part of this UI, can they predict how the rest works?"*

Check:
- Are the same actions always in the same position across pages?
- Do similar components look the same everywhere?
- Is terminology consistent? ("Delete" on one page, "Remove" on another = confusion)
- Is the primary action always in the same visual location (top-right, bottom of form)?
- Are destructive actions always styled the same way (red, or always requiring confirmation)?

Failures to fix:
```
✗ "Save" button is top-right on one page, bottom-left on another
✗ "Delete" is red in some places, gray in others
✗ Card components look different across pages for same data type
✗ Mixed terminology for same action across pages
✗ Modals confirm some actions but not others of equal destructiveness
```

### Principle 6: Credibility & Trust
*"Does this UI signal that the product is reliable, professional, and safe to use?"*

Check:
- Is the visual hierarchy consistent and intentional — or random?
- Does the spacing feel considered — or like content was just dropped in?
- Are there any obvious alignment issues (elements not aligned to a grid)?
- Is the typography consistent — or do different fonts appear without reason?
- Is there any visual clutter that doesn't serve the user?

Failures to fix:
```
✗ Mixed font sizes with no clear hierarchy logic
✗ Inconsistent padding inside cards of the same type
✗ Buttons of different heights in the same row
✗ Text not aligned to a baseline grid
✗ Different border styles (some solid, some dashed) with no semantic meaning
```

---

## Phase 4 — Redesign Decisions

Based on Phases 1–3, produce a redesign plan for each page:

```
Page: [name]

Friction issues found:
  1. [element] — Importance: [X], Prominence: [Y], Friction: [Z]
     Fix: [specific action]

UX Principle failures:
  1. [Principle] — [specific failure]
     Fix: [specific action]

Layout changes:
  - Move [X] from [position] to [position] because [user reason]
  - Promote [X] visually because [user importance score]
  - Demote [X] visually because [user rarely needs it]
  - Merge [X] and [Y] because users see them as one action
  - Split [X] from [Y] because they serve different user goals

Hierarchy after redesign:
  Level 1 (most prominent): [primary action / main content]
  Level 2: [secondary navigation / supporting actions]
  Level 3: [filters, metadata, secondary info]
  Level 4: [rarely needed / power user / administrative]
```

---

## Phase 5 — Universal Placement Rules

Apply these rules to every page. No exceptions.

### Primary Action Placement
```
Desktop: Top-right of the page header OR prominent center for landing-type pages
Mobile:  Bottom of screen, full-width, reachable by thumb
Forms:   Bottom-right of the form, after all inputs
Dialogs: Bottom-right of the dialog (primary), bottom-left (cancel/secondary)
```

### Search Placement
```
If page has >8 browseable items: Search must be above the fold, top of content area
If page is primarily a list/table: Search is the FIRST element, before any content
If app-wide search: Fixed in top navigation bar, keyboard shortcut labeled (⌘K or /)
```

### Destructive Actions
```
Never place Delete/Remove/Reset next to primary actions
Always at least 48px away from primary action
Always require confirmation (inline warning or confirmation dialog)
Always styled differently from primary actions (outlined or text-only, never filled red)
Never in the same button group as Save/Create/Submit
```

### Navigation Hierarchy
```
Primary navigation: Always visible, maximum 7 items (Miller's Law)
Secondary navigation: Tabs or sub-nav, only within a section
Tertiary actions: Kebab menu (...) or overflow — never primary
Breadcrumbs: Required on any page more than 2 levels deep
Back button: Required on any modal, drawer, or detail page
```

### Information Architecture Order (top to bottom, left to right)
```
1. Context (where am I? what section is this?)
2. Primary action (what can I DO here?)
3. Primary content (what am I here to SEE?)
4. Secondary actions (what else can I do?)
5. Filters / search (how can I narrow this?)
6. Metadata / details (additional context)
7. Administrative / destructive actions (edge cases)
```

### Above-the-Fold Rule
```
The following must ALWAYS be visible without scrolling on a 1440px viewport:
  ✓ Page title / context
  ✓ Primary action button
  ✓ Search (if the page is a list/table)
  ✓ At least one meaningful piece of content (not just a loading state)
```

---

## Phase 6 — Apply Changes

Now rewrite the UI. Keep all tokens. Keep all logic. Change only:

**What you change:**
- Order of elements in JSX (move DOM order to match visual priority)
- CSS grid/flex layout (restructure layout to match user hierarchy)
- Visual weight (size, contrast, position of primary vs secondary vs tertiary)
- Spacing between groups (signal grouping through whitespace)
- Button labels (rename to action-oriented language)
- Empty state copy and icon (make it helpful, not blank)
- Form structure (label above input, required indicators, helper text)
- Navigation label text (developer language → user language)

**What you never change:**
- Token values (colors, spacing scale, radius scale, shadow scale)
- Token class names (just keep using them)
- Any JavaScript logic
- Any API calls or data fetching
- Any state management
- Any props or component interfaces

---

## Phase 7 — God Mode Output Report

After completing all pages:

```
✓ God Mode Complete

Pages analyzed: [N]
Pages redesigned: [N]
Tokens: Preserved (use --remove-tokens to reset)

Friction issues fixed: [N]
  Critical (score 3+): [N]
  Moderate (score 2): [N]

UX Principle violations fixed: [N]
  Usability: [N]
  Findability: [N]
  Accessibility: [N]
  Desirability: [N]
  Learnability: [N]
  Credibility: [N]

Key redesign decisions:
  [Page name]: [primary change and user reason]
  [Page name]: [primary change and user reason]
  ...

User journey improvement:
  Before: User arrived → scanned [X] seconds → found action
  After:  User arrived → primary action visible immediately

What was NOT changed:
  → All tokens and design system values preserved
  → All business logic untouched
  → All API calls untouched
  → All handlers untouched
```

---

## The God Mode Mindset — Final Reminder

Before you finish any page, ask yourself these 5 questions as a user:

1. **"I just arrived. In 3 seconds, do I know what I can do here?"** If no — the primary action is not prominent enough.

2. **"I'm looking for something specific. Can I find it without scrolling?"** If no — search/filter placement needs to move up.

3. **"I need to do the main thing. Is the button obvious and does it tell me exactly what will happen?"** If no — button label or visual weight is wrong.

4. **"I made a mistake. How do I undo or recover?"** If there's no clear answer — destructive actions need confirmation and reversible actions need undo.

5. **"I'm on mobile with one hand. Can I do the thing I came here to do?"** If no — touch targets and thumb zones need attention.

If all 5 answers are yes — the page passes god mode.
