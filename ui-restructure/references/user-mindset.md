# User Mindset Reference — How Real Users Actually Behave

This is not theory. This is how real users actually interact with software. Apply every insight here during God Mode.

---

## The 10 Laws of User Behavior

### Law 1: Users Don't Read — They Scan
Eye-tracking studies show users follow two patterns:

**F-Pattern** (for content-heavy pages: articles, lists, tables)
```
████████████████████  ← First horizontal scan (full width)
██████████░░░░░░░░░░  ← Second horizontal scan (less far right)
████░░░░░░░░░░░░░░░░  ← Vertical scan down the left side
████░░░░░░░░░░░░░░░░
████░░░░░░░░░░░░░░░░
```
Implication: Most important information must be LEFT-aligned and near the TOP.

**Z-Pattern** (for sparse pages: landing pages, dashboards, forms)
```
████████████████████  ← Read left to right
          ░░░░░░████  ← Diagonal scan
████████████████████  ← Read left to right again
```
Implication: Place: Logo top-left → CTA top-right → key info bottom-left → primary action bottom-right.

**What this means for layout:**
- Never put critical information only in the right column
- Never put the primary CTA only at the bottom (unless it's a form completion)
- Navigation that only exists in the right sidebar will be missed
- Long horizontal text blocks will not be read — use hierarchy and bullets

---

### Law 2: Users Satisfice — They Take the First Acceptable Option
Users don't look for the BEST option. They take the FIRST one that seems good enough.

**What this means:**
- The first search result gets clicked most often — ordering matters
- The first visible button in the form gets clicked — even if it's "Cancel"
- The first navigation item gets explored first — even if it's wrong
- If the correct option isn't in the first 3 visible results, users assume it doesn't exist

**Fixes:**
```
✓ Put the most common/correct option FIRST in any list
✓ Most used nav item = first in nav
✓ Default form values should be the most common answer
✓ Recommended/default options should be pre-selected
✓ Search should surface exact matches before partial matches
```

---

### Law 3: Miller's Law — 7 ± 2 Items in Working Memory
Humans can hold 5–9 items in short-term memory at once.

**What this means:**
- Navigation menus with >7 items cause cognitive overload
- Form pages with >7 fields at once feel overwhelming
- Dropdown menus with >9 options need grouping or search
- Tab bars with >5 tabs need an overflow menu

**Fixes:**
```
✓ Group navigation into max 5–7 primary items, collapse rest
✓ Break long forms into steps (max 5–7 fields per step)
✓ Group select options under headers if >9 options
✓ Use progressive disclosure: show basic options, hide advanced
```

---

### Law 4: Hick's Law — More Options = Slower Decisions
Decision time increases logarithmically with the number of choices.

**What this means:**
- Two buttons is faster than three buttons
- A single prominent CTA converts better than three equal-weight options
- Showing ALL features at once paralyzes new users

**Fixes:**
```
✓ One primary CTA per page section (not three)
✓ Reduce simultaneous choices wherever possible
✓ Use smart defaults so users can proceed without deciding
✓ Progressive disclosure: start simple, reveal complexity on demand
✓ Highlight recommended option in pricing/feature tables
```

---

### Law 5: Fitts's Law — Larger Targets Are Clicked Faster
Time to hit a target = log(distance/size).

**What this means:**
- Primary actions should be large and close to where the user's attention is
- Corners and edges of screens are easy to reach (infinite size in one direction)
- Small text links for important actions will slow users down and cause misclicks

**Fixes:**
```
✓ Primary CTA: minimum 44px height, generous padding
✓ Mobile: full-width buttons at the bottom of the screen
✓ Don't make users move the cursor far from where they just clicked for the next action
✓ Destructive actions: make them small and far from the primary action
✓ Inline table actions: right-click or hover reveal, not always-visible small icons
```

---

### Law 6: The 3-Second Rule
If a user cannot understand what to do within 3 seconds of arriving at a page, they either leave or feel frustrated.

**What this means:**
- Every page must have a single, visually dominant message or action
- Page titles must be descriptive, not clever
- Hero sections must answer "What is this?" and "What do I do?" immediately
- Loading states must appear within 200ms of user action (or user thinks it's broken)

**The 3-second test:**
Cover the page and uncover it for 3 seconds. Then ask:
- What is this page about?
- What can I do here?
- Where is my eye going?

If you can't answer all three confidently — the hierarchy is wrong.

---

### Law 7: Jakob's Law — Users Expect Familiar Patterns
Users spend most of their time on OTHER products. They come to yours with existing mental models.

**What this means:**
- The logo should be top-left and link to home (always)
- Search should be in the top bar or prominently at top of content (always)
- Shopping cart / notifications should be top-right (always)
- Settings should be in the user avatar dropdown (always)
- Destructive actions should be red and require confirmation (always)
- "Cancel" should be left of "Confirm" in dialogs (always, matches OS standards)

**Fixes:**
```
✓ Never be "creative" with standard navigation patterns
✓ Icons must match their universal meaning (✓ = confirm, ✗ = close, ≡ = menu)
✓ Back button must actually go back
✓ Form submit should be at the bottom of the form
✓ Search icon (🔍) must open search — don't use it for something else
```

---

### Law 8: The Goal Gradient Effect
Users accelerate as they get closer to completing a goal.

**What this means:**
- Progress indicators make users more likely to complete multi-step flows
- Showing "2 of 3 steps" is more motivating than "Step 2"
- Pre-filling data they've already given reduces abandonment
- Showing what they'll get AFTER completing increases completion rate

**Fixes:**
```
✓ Multi-step forms: always show step progress (1 → 2 → 3)
✓ Onboarding: show "X% complete" with checkmarks for done steps
✓ Never ask for information you already have
✓ Show the reward/outcome before the user completes the action
```

---

### Law 9: The Peak-End Rule
Users judge an experience by its most intense moment AND its final moment — not the average.

**What this means:**
- If the end of a flow (success state, confirmation page) is good, the whole experience is remembered as good
- If the worst moment (error state, confusing step) is bad, the whole experience is remembered as bad
- Investment in success states and empty states has outsized impact on perception

**Fixes:**
```
✓ Success pages/states should feel rewarding and clear ("You did it!")
✓ Error messages should solve the problem, not just report it
✓ Empty states should be inviting and guide the next action
✓ Last thing users see in a flow should be positive confirmation
✓ Onboarding success = celebrate it (brief animation, checkmark, kind message)
```

---

### Law 10: The Principle of Least Astonishment
Users should not be surprised by what the UI does.

**What this means:**
- Clicking "Save" should save (not navigate away without confirmation)
- Clicking "Delete" should ask for confirmation (not immediately delete)
- Clicking a link should navigate (not download unless labeled as download)
- Clicking "Cancel" should discard changes (with confirmation if significant changes exist)
- Scrolling should scroll (not trigger unexpected behaviors)

**Fixes:**
```
✓ Destructive irreversible actions: always confirm first
✓ Navigation away from a form with unsaved changes: warn the user
✓ File downloads: label the link clearly as a download with file format
✓ External links: indicate they open in a new tab (↗ icon)
✓ Loading: always show feedback within 200ms when user triggers an action
```

---

## Common Developer Mistakes That Ruin User Experience

These patterns appear in developer-built UIs constantly. God Mode must identify and fix all of them.

### Mistake 1: Button labels that describe the system, not the action
```
Bad:  "Submit"   "OK"   "Confirm"   "Yes"   "Process"
Good: "Save Changes"  "Delete Account"  "Send Message"  "Create Project"  "Confirm Booking"
```

### Mistake 2: Form errors only shown after submission
```
Bad:  User fills 8 fields → submits → sees 3 errors at top of page
Good: Inline validation as user leaves each field (on blur)
      Show: what went wrong + how to fix it next to the field
```

### Mistake 3: Icon-only actions with no labels for primary actions
```
Bad:  A pencil icon with no label for "Edit" (power users know it, new users don't)
Good: "Edit" text + pencil icon for primary actions
      Icon-only acceptable for well-established icons (⋯ menu, ✕ close, ← back)
      But always add aria-label for icon-only buttons
```

### Mistake 4: Placing related actions far apart
```
Bad:  "Edit" button is top-right, "Delete" button is bottom-left for the same item
Good: Actions for the same item are grouped together, in order of frequency of use
```

### Mistake 5: Using color alone to communicate state
```
Bad:  Green text = active, Red text = inactive (colorblind users see the same)
Good: Green dot + "Active" label  /  Red dot + "Inactive" label
      Always pair color with text or icon
```

### Mistake 6: Making destructive actions visually equal to constructive ones
```
Bad:  [Save]  [Delete]  — same visual weight, same size, same position
Good: [Save]  ···  where ··· reveals [Delete] with confirmation
      OR: [Save]  [Delete] where Delete is text-only/outlined, Save is filled/primary
```

### Mistake 7: Navigation reflects the database, not the user's mental model
```
Bad:  "UserRecords"  "ProductEntities"  "OrderInstances"  "ConfigurationSettings"
Good: "Users"  "Products"  "Orders"  "Settings"
Even better: "My Team"  "Catalog"  "Recent Orders"  "Preferences"
```

### Mistake 8: Empty states that are just empty
```
Bad:  Table with no rows. Just column headers and whitespace.
Good: Illustration + headline + description + CTA button
      "No orders yet — create your first one" [Create Order]
```

### Mistake 9: Loading states that are missing or happen too late
```
Bad:  User clicks button → nothing happens for 800ms → UI suddenly changes
Good: User clicks button → button immediately shows spinner/loading state (within 100ms)
      → Content loads → success state
```

### Mistake 10: Modals that require scrolling to find the confirm button
```
Bad:  Modal with long content → Confirm button is off-screen below the fold
Good: Sticky footer inside the modal with Cancel / Confirm always visible
      OR: scrollable content area with fixed action row at the bottom
```
