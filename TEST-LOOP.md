# ui-restructure Skill — End-to-End Test Loop

You are the **QA Orchestrator** for the `ui-restructure` skill located at `ui-restructure/SKILL.md`.

Your job: run all 10 personas below, end-to-end. Each persona creates a real test project, invokes the skill, verifies results by reading the modified files, scores the outcome, then either deletes the project (pass) or fixes the skill and commits (fail). After all personas complete, update `bug.md` and print the cycle summary.

---

## Pre-Flight (run once before any persona)

1. Read `ui-restructure/SKILL.md` fully.
2. Read `ui-restructure/parser/commands.md` fully.
3. Read `bug.md` — load all `[OPEN]` bugs so you know what is already broken and can watch for regressions.
4. Set `CYCLE_DATE` = today's date (YYYY-MM-DD).
5. Create a scratch workspace: `mkdir -p /tmp/ui-test-workspace && cd /tmp/ui-test-workspace`.
6. Confirm the workspace is empty before starting.

---

## The 10 Test Personas

Run each persona sequentially. For each:
- **SETUP** — scaffold a minimal project with deliberate logic markers
- **INVOKE** — run the skill on that project (simulate what the skill would do, applying every step in SKILL.md to those files)
- **VERIFY** — check every assertion listed; record PASS/FAIL per assertion
- **SCORE** — compute persona score (passed assertions / total assertions × 100)
- **OUTCOME** — if score ≥ 80: delete project, record success. If score < 80: diagnose, fix skill files, commit, log bug.

---

### Persona 1 — Maya
**Profile:** Senior frontend dev, Next.js App Router, Tailwind CSS, wants Apple-style redesign.
**Command:** `/ui-restructure --style apple`

**SETUP — create `/tmp/ui-test-workspace/maya/`:**

```
app/
  layout.tsx        ← root layout
  page.tsx          ← dashboard page
  components/
    ProjectCard.tsx ← card with logic markers
    Header.tsx      ← nav header
tailwind.config.ts
tokens.ts
package.json        ← { "dependencies": { "next": "^14" } }
```

**Logic markers to embed (MUST survive the restructure):**
- `ProjectCard.tsx`: `const [isDeleting, setIsDeleting] = useState(false)` + `const handleDelete = async (id: string) => { await fetch('/api/projects/' + id, { method: 'DELETE' }) }` + `{projects.map(p => (<div key={p.id}>`)
- `Header.tsx`: `const { user } = useAuth()` + `const handleLogout = () => signOut({ callbackUrl: '/' })`
- `page.tsx`: `const { data: projects } = useSWR('/api/projects', fetcher)`

**INVOKE — apply skill steps 1–11 to these files:**

Execute every step in SKILL.md against this project:
- Step 1: Parse → `style: apple, mode: full, keep-tokens: false`
- Step 2: Detect → Next.js App Router (app/ + layout.tsx present)
- Step 3: Detect styling → Tailwind CSS
- Step 4: Build UI model
- Step 5: Identify logic to preserve
- Step 6: Strip UI structure
- Step 7: Reset tokens (default — no --keep-tokens)
- Step 8: Load `engines/apple.md`
- Step 9: Load `modes/full.md`
- Step 10: Rebuild UI
- Step 11: Polish pass

**VERIFY (12 assertions):**

| # | Assertion | How to Check |
|---|-----------|--------------|
| V1 | `useState(false)` still present in ProjectCard.tsx | grep for `useState(false)` |
| V2 | `handleDelete` function body unchanged | grep for `await fetch('/api/projects/'` |
| V3 | `{projects.map(p => (` still present | grep for `.map(p =>` |
| V4 | `useAuth()` still present in Header.tsx | grep for `useAuth` |
| V5 | `handleLogout` function still present | grep for `handleLogout` |
| V6 | `useSWR('/api/projects'` still present in page.tsx | grep for `useSWR` |
| V7 | CSS classes have changed (Apple-style classes present: `backdrop-blur`, `bg-white/80`, or `rounded-2xl`) | grep modified files for apple tokens |
| V8 | `tokens.ts` has been updated with new values | read tokens.ts, verify it differs from original |
| V9 | `tailwind.config.ts` theme.extend has been updated | read file |
| V10 | No `import` statements were removed or added | compare import lines before/after |
| V11 | Skill outputs framework detection line "Next.js App Router" | check output |
| V12 | Polish pass: interactive elements have transition classes | grep for `transition` in modified components |

---

### Persona 2 — Dev
**Profile:** React developer, Vite + CSS Modules, wants layout restructure only (keep tokens).
**Command:** `/ui-restructure --mode layout --keep-tokens`

**SETUP — create `/tmp/ui-test-workspace/dev/`:**

```
src/
  App.jsx
  components/
    TaskList.jsx    ← list component with logic
    TaskItem.jsx    ← item with handlers
  styles/
    TaskList.module.css
    TaskItem.module.css
vite.config.ts
package.json        ← { "dependencies": { "react": "^18", "vite": "^5" } }
```

**Logic markers:**
- `TaskList.jsx`: `const [tasks, setTasks] = useState([])` + `useEffect(() => { fetchTasks().then(setTasks) }, [])` + `const handleReorder = (from, to) => { const copy = [...tasks]; copy.splice(to, 0, copy.splice(from, 1)[0]); setTasks(copy) }`
- `TaskItem.jsx`: `const handleToggle = (id) => onToggle(id)` + `const handleEdit = async (id, data) => { const res = await fetch('/api/tasks/' + id, { method: 'PATCH', body: JSON.stringify(data) }) }`

**INVOKE — apply skill steps 1–11:**
- Step 2: Detect → React (Vite) — `vite.config.ts` present, no `app/` dir
- Step 3: Detect → CSS Modules
- Step 7: SKIP (--keep-tokens set)
- Step 9: Load `modes/layout.md`

**VERIFY (10 assertions):**

| # | Assertion | How to Check |
|---|-----------|--------------|
| V1 | `useState([])` still in TaskList.jsx | grep |
| V2 | `useEffect` fetch call preserved | grep for `fetchTasks` |
| V3 | `handleReorder` splice logic intact | grep for `.splice(` |
| V4 | `handleToggle` and `handleEdit` in TaskItem.jsx | grep |
| V5 | `fetch('/api/tasks/'` preserved | grep |
| V6 | CSS Module class names in `.module.css` files were changed (layout restructured) | read files, check they differ |
| V7 | Token files NOT touched (--keep-tokens) | check if any token file was modified — should be unchanged |
| V8 | Framework detected as "React (Vite)" in output | check output |
| V9 | Styling detected as "CSS Modules" | check output |
| V10 | Mode applied as "layout" in summary | check summary output |

---

### Persona 3 — Priya
**Profile:** Full-stack dev, Next.js Pages Router + shadcn/ui, wants Linear style.
**Command:** `/ui-restructure --style linear`

**SETUP — create `/tmp/ui-test-workspace/priya/`:**

```
pages/
  _app.tsx
  index.tsx
  dashboard.tsx
components/
  DataTable.tsx   ← table with data logic
  FilterBar.tsx   ← filter with state
components.json   ← shadcn config
tailwind.config.ts
package.json      ← { "dependencies": { "next": "^14", "@shadcn/ui": "^0.9" } }
```

**Logic markers:**
- `DataTable.tsx`: `const [sortCol, setSortCol] = useState('name')` + `const [filters, setFilters] = useState({})` + `const sortedData = useMemo(() => [...data].sort((a,b) => a[sortCol].localeCompare(b[sortCol])), [data, sortCol])`
- `FilterBar.tsx`: `const handleFilterChange = (key, value) => { setFilters(prev => ({...prev, [key]: value})); onFilterChange({...filters, [key]: value}) }`

**INVOKE — apply skill steps 1–11:**
- Step 2: Detect → Next.js Pages Router
- Step 3: Detect → Tailwind CSS + shadcn/ui
- Step 8: Load `engines/linear.md`

**VERIFY (11 assertions):**

| # | Assertion | How to Check |
|---|-----------|--------------|
| V1 | `useState('name')` still in DataTable.tsx | grep |
| V2 | `useMemo` sort logic preserved | grep for `useMemo` |
| V3 | `handleFilterChange` callback body intact | grep for `setFilters(prev =>` |
| V4 | `onFilterChange` call preserved | grep |
| V5 | Linear-style classes applied: `text-xs`, `border`, `rounded` (tight/dense) | grep modified files |
| V6 | shadcn component imports not removed | grep for `from "@/components/ui/` |
| V7 | Framework detected as "Next.js Pages Router" | check output |
| V8 | shadcn detected in styling systems list | check output |
| V9 | Token files updated with Linear dark palette | read tokens/tailwind.config |
| V10 | `components.json` NOT modified (shadcn config is not a UI file) | verify untouched |
| V11 | Summary shows `Style: linear` | check output |

---

### Persona 4 — Alex
**Profile:** Vue developer, Vue 3 + Tailwind, wants dashboard style for data-dense layout.
**Command:** `/ui-restructure --style dashboard --density compact`

**SETUP — create `/tmp/ui-test-workspace/alex/`:**

```
src/
  App.vue
  views/
    Dashboard.vue   ← main view
  components/
    MetricCard.vue  ← metric with computed
    ChartWidget.vue ← chart wrapper
vite.config.ts
tailwind.config.ts
package.json        ← { "dependencies": { "vue": "^3", "vite": "^5" } }
```

**Logic markers:**
- `Dashboard.vue`: `const metrics = ref([])` + `onMounted(async () => { metrics.value = await fetchMetrics() })` + `const totalRevenue = computed(() => metrics.value.reduce((sum, m) => sum + m.revenue, 0))`
- `MetricCard.vue`: `const props = defineProps({ metric: Object, onDrilldown: Function })` + `const handleClick = () => props.onDrilldown(props.metric.id)`

**INVOKE — apply skill steps 1–11:**
- Step 2: Detect → Vue 3 (`.vue` files + vite.config.ts)
- Step 3: Detect → Tailwind CSS
- Step 8: Load `engines/dashboard.md`
- Step 9: Load `modes/full.md` with `density: compact`

**VERIFY (10 assertions):**

| # | Assertion | How to Check |
|---|-----------|--------------|
| V1 | `const metrics = ref([])` preserved | grep |
| V2 | `onMounted` fetch preserved | grep for `fetchMetrics` |
| V3 | `computed()` revenue logic intact | grep for `reduce` |
| V4 | `defineProps` not modified | grep |
| V5 | `handleClick` callback intact | grep for `onDrilldown` |
| V6 | Framework detected as "Vue 3" | check output |
| V7 | Dashboard-style classes applied (data-dense grid) | grep modified files |
| V8 | Compact density applied (smaller spacing scale) | check spacing values in rebuilt files |
| V9 | `.vue` file structure (template/script/style blocks) preserved | read files |
| V10 | `<template>` tags not corrupted | verify valid Vue SFC structure |

---

### Persona 5 — Jordan
**Profile:** React dev, React CRA + styled-components, theme-only change (no layout changes).
**Command:** `/ui-restructure --mode theme --style minimal`

**SETUP — create `/tmp/ui-test-workspace/jordan/`:**

```
src/
  App.jsx
  components/
    AuthForm.jsx    ← form with validation logic
    UserProfile.jsx ← profile with async handlers
  styles/
    theme.js        ← styled-components theme
public/
  index.html
package.json        ← { "dependencies": { "react": "^18", "styled-components": "^6" } }
```

**Logic markers:**
- `AuthForm.jsx`: `const [errors, setErrors] = useState({})` + `const validate = (data) => { const errs = {}; if (!data.email.includes('@')) errs.email = 'Invalid'; return errs }` + `const handleSubmit = async (e) => { e.preventDefault(); const errs = validate(formData); if (Object.keys(errs).length) { setErrors(errs); return }; const res = await fetch('/api/auth/login', { method: 'POST', body: JSON.stringify(formData) }) }`
- `UserProfile.jsx`: `const handleAvatarUpload = async (file) => { const form = new FormData(); form.append('avatar', file); await fetch('/api/users/avatar', { method: 'POST', body: form }) }`

**INVOKE:**
- Step 2: Detect → React (CRA) — `src/App.jsx` + `public/index.html`
- Step 3: Detect → styled-components
- Step 7: Reset theme tokens in `styles/theme.js`
- Step 9: Load `modes/theme.md` — tokens only, layout unchanged

**VERIFY (11 assertions):**

| # | Assertion | How to Check |
|---|-----------|--------------|
| V1 | `useState({})` preserved in AuthForm.jsx | grep |
| V2 | `validate()` function with email check intact | grep for `includes('@')` |
| V3 | `handleSubmit` fetch to `/api/auth/login` preserved | grep |
| V4 | `setErrors(errs)` call preserved | grep |
| V5 | `handleAvatarUpload` FormData logic intact | grep for `new FormData` |
| V6 | Styled-components theme tokens UPDATED in `styles/theme.js` | read file — should have new minimal colors |
| V7 | Layout structure of components NOT changed (mode=theme) | check that JSX structure/DOM hierarchy unchanged |
| V8 | Framework detected as "React (CRA)" | check output |
| V9 | styled-components detected in styling systems | check output |
| V10 | Mode summary shows "theme" | check output |
| V11 | No new styled-component definitions added to component files | only theme.js should change |

---

### Persona 6 — Sam
**Profile:** Startup founder, Next.js App Router + Tailwind, god-mode user-first redesign.
**Command:** `/ui-restructure --god-mode`

**SETUP — create `/tmp/ui-test-workspace/sam/`:**

```
app/
  page.tsx           ← onboarding/landing
  dashboard/
    page.tsx         ← main dashboard
  components/
    Sidebar.tsx      ← nav sidebar
    NotificationBell.tsx ← notification logic
tailwind.config.ts
tokens.ts
package.json         ← { "dependencies": { "next": "^14" } }
```

**Logic markers:**
- `Sidebar.tsx`: `const { pathname } = useRouter()` + `const isActive = (href: string) => pathname === href` + `const handleNavClick = (href: string) => { logNavEvent(href); router.push(href) }`
- `NotificationBell.tsx`: `const [unread, setUnread] = useState(0)` + `useEffect(() => { const interval = setInterval(() => fetch('/api/notifications/count').then(r => r.json()).then(d => setUnread(d.count)), 30000); return () => clearInterval(interval) }, [])` + `const handleMarkAllRead = async () => { await fetch('/api/notifications/read-all', { method: 'POST' }); setUnread(0) }`

**INVOKE — god-mode path:**
- Read `references/user-mindset.md` + `modes/godmode.md`
- Skip standard Steps 1–11
- Execute 7-phase God Mode pipeline from `modes/godmode.md`
- Tokens PRESERVED by default (no --remove-tokens)

**VERIFY (12 assertions):**

| # | Assertion | How to Check |
|---|-----------|--------------|
| V1 | `useRouter()` and `pathname` logic intact | grep |
| V2 | `isActive` function preserved | grep |
| V3 | `handleNavClick` with `logNavEvent` preserved | grep |
| V4 | `setInterval` polling logic in NotificationBell intact | grep for `setInterval` |
| V5 | `clearInterval` cleanup in useEffect return preserved | grep for `clearInterval` |
| V6 | `handleMarkAllRead` fetch call intact | grep for `read-all` |
| V7 | Token files NOT reset (god-mode keeps tokens by default) | verify tokens.ts unchanged |
| V8 | UI layout/hierarchy HAS changed (god-mode still rebuilds layout) | verify JSX structure modified |
| V9 | Skill read both `user-mindset.md` and `modes/godmode.md` (confirm in reasoning) | check that god-mode pipeline was followed, not standard |
| V10 | God Mode output references user perspectives (first-time visitor, goal-oriented, etc.) | check output narrative |
| V11 | Page inventory was built (Phase 1 of God Mode) | check output for page analysis |
| V12 | Summary does NOT show standard mode (should show God Mode path) | check summary |

---

### Persona 7 — Taylor
**Profile:** Design-conscious engineer, Next.js App Router, god-mode + apple style + remove tokens.
**Command:** `/ui-restructure --god-mode --style apple --remove-tokens`

**SETUP — create `/tmp/ui-test-workspace/taylor/`:**

```
app/
  page.tsx
  settings/
    page.tsx
  components/
    SettingsForm.tsx  ← form with complex logic
    ConfirmDialog.tsx ← modal with state
tokens.ts
tailwind.config.ts
package.json
```

**Logic markers:**
- `SettingsForm.tsx`: `const [isDirty, setIsDirty] = useState(false)` + `const handleChange = (field, value) => { setFormData(prev => ({...prev, [field]: value})); setIsDirty(true) }` + `const handleSave = async () => { if (!isDirty) return; const res = await fetch('/api/settings', { method: 'PUT', body: JSON.stringify(formData) }); if (res.ok) setIsDirty(false) }`
- `ConfirmDialog.tsx`: `const [isOpen, setIsOpen] = useState(false)` + `const handleConfirm = async () => { await onConfirm(); setIsOpen(false) }` + `const handleCancel = () => { setIsOpen(false); onCancel?.() }`

**INVOKE:**
- God Mode pipeline (from `modes/godmode.md`)
- After God Mode: run Step 7 (token reset) + Step 10 (token rebuild) with apple engine
- `engines/apple.md` defines tokens

**VERIFY (13 assertions):**

| # | Assertion | How to Check |
|---|-----------|--------------|
| V1 | `useState(false)` for isDirty preserved | grep |
| V2 | `handleChange` spread logic intact | grep for `({...prev,` |
| V3 | `handleSave` with early return `if (!isDirty) return` preserved | grep |
| V4 | `fetch('/api/settings'` PUT call preserved | grep |
| V5 | `setIsDirty(false)` after success preserved | grep |
| V6 | `isOpen` state in ConfirmDialog intact | grep |
| V7 | `handleConfirm` and `handleCancel` logic intact | grep |
| V8 | `onCancel?.()` optional chaining preserved | grep |
| V9 | Tokens WERE reset (--remove-tokens) | verify tokens.ts changed from original |
| V10 | Apple-style tokens injected (glassmorphism values) | grep tokens.ts for `backdrop-blur` or `bg-white/80` |
| V11 | God Mode pipeline ran (not standard pipeline) | check output |
| V12 | Both `--god-mode` and `--remove-tokens` correctly combined | check parser output confirms both flags |
| V13 | Polish pass applied (spring motion, 5-state interactions) | grep modified files for `active:scale` or `transition` |

---

### Persona 8 — Morgan
**Profile:** Product manager turned dev, React Vite, convert messy lists into card grid.
**Command:** `/ui-restructure --mode grid --grid cards`

**SETUP — create `/tmp/ui-test-workspace/morgan/`:**

```
src/
  components/
    ProjectList.jsx   ← list with items (MUST become card grid)
    TeamList.jsx      ← another list
vite.config.ts
tailwind.config.ts
package.json
```

**Logic markers:**
- `ProjectList.jsx`: `const [projects, setProjects] = useState([])` + `const [loading, setLoading] = useState(true)` + `useEffect(() => { fetch('/api/projects').then(r => r.json()).then(data => { setProjects(data); setLoading(false) }) }, [])` + `const handleArchive = async (id) => { await fetch('/api/projects/' + id + '/archive', { method: 'POST' }); setProjects(prev => prev.filter(p => p.id !== id)) }`

**Layout before (list):**
```jsx
<ul>
  {projects.map(p => (
    <li key={p.id}>
      <span>{p.name}</span>
      <button onClick={() => handleArchive(p.id)}>Archive</button>
    </li>
  ))}
</ul>
```

**INVOKE:**
- Load `modes/grid.md`
- Convert list structure to card grid

**VERIFY (10 assertions):**

| # | Assertion | How to Check |
|---|-----------|--------------|
| V1 | `useState([])` and `useState(true)` preserved | grep |
| V2 | `useEffect` fetch to `/api/projects` preserved | grep |
| V3 | `handleArchive` logic with filter intact | grep for `.filter(p =>` |
| V4 | `{projects.map(p => (` still present (data mapping preserved) | grep |
| V5 | `key={p.id}` preserved on mapped items | grep |
| V6 | `onClick={() => handleArchive(p.id)}` preserved on button | grep |
| V7 | `<ul>/<li>` converted to grid layout (e.g., `<div className="grid ...">/<div>`) | check structure changed |
| V8 | Grid classes applied (e.g., `grid-cols-2`, `grid-cols-3`) | grep for `grid-col` |
| V9 | Mode summary shows "grid" | check output |
| V10 | Both ProjectList.jsx and TeamList.jsx converted | check both files modified |

---

### Persona 9 — Riley
**Profile:** Design system engineer, Next.js App Router, rebuild layout but KEEP existing tokens.
**Command:** `/ui-restructure --keep-tokens --style linear`

**SETUP — create `/tmp/ui-test-workspace/riley/`:**

```
app/
  page.tsx
  components/
    NavBar.tsx       ← navigation
    ContentArea.tsx  ← main content
tokens.ts            ← CAREFULLY CRAFTED tokens that MUST NOT change
tailwind.config.ts
package.json
```

**Original tokens.ts content (record exactly):**
```ts
export const tokens = {
  colors: { primary: '#6366f1', surface: '#1e1e2e', text: '#cdd6f4' },
  spacing: { base: 8, scale: [4,8,16,24,40,64] },
  radius: { sm: 4, md: 8, lg: 16, xl: 24 }
}
```

**Logic markers:**
- `NavBar.tsx`: `const [isMobileOpen, setIsMobileOpen] = useState(false)` + `const handleMobileToggle = () => setIsMobileOpen(prev => !prev)` + `const { user, isLoading } = useSession()`

**INVOKE:**
- Step 7: SKIP (--keep-tokens)
- Linear style engine for layout shapes, but existing token values preserved

**VERIFY (10 assertions):**

| # | Assertion | How to Check |
|---|-----------|--------------|
| V1 | `tokens.ts` is bit-for-bit identical to original | read file, compare |
| V2 | `tailwind.config.ts` token values unchanged | read file |
| V3 | `useState(false)` and `handleMobileToggle` preserved | grep |
| V4 | `useSession()` call preserved | grep |
| V5 | Layout structure of NavBar.tsx HAS changed | verify JSX structure differs |
| V6 | ContentArea.tsx layout HAS changed | verify JSX structure differs |
| V7 | Linear structural patterns applied (compact spacing, sharp borders) without overriding tokens | verify layout classes changed but token references unchanged |
| V8 | Summary shows `Keep tokens: true` | check output |
| V9 | Framework detected as "Next.js App Router" | check output |
| V10 | No token import path changed | grep for `import.*tokens` — same path |

---

### Persona 10 — Casey
**Profile:** SaaS founder, Next.js App Router, custom prompt for AI analytics dashboard.
**Command:** `/ui-restructure --prompt "AI analytics dashboard with dark sidebar, floating metric cards, and real-time chart area" --style dashboard --density compact`

**SETUP — create `/tmp/ui-test-workspace/casey/`:**

```
app/
  dashboard/
    page.tsx           ← analytics dashboard
  components/
    MetricsPanel.tsx   ← metrics with complex polling
    ChartArea.tsx      ← chart wrapper with websocket
    AIInsights.tsx     ← AI suggestions panel
tokens.ts
tailwind.config.ts
package.json
```

**Logic markers:**
- `MetricsPanel.tsx`: `const [metrics, setMetrics] = useState(null)` + `useEffect(() => { const ws = new WebSocket('wss://api.example.com/metrics'); ws.onmessage = (e) => setMetrics(JSON.parse(e.data)); return () => ws.close() }, [])`
- `ChartArea.tsx`: `const chartRef = useRef(null)` + `useEffect(() => { if (!chartRef.current || !data) return; const chart = new Chart(chartRef.current, { type: 'line', data, options: chartOptions }); return () => chart.destroy() }, [data])`
- `AIInsights.tsx`: `const [insights, setInsights] = useState([])` + `const fetchInsights = async () => { const res = await fetch('/api/ai/insights', { headers: { 'Authorization': 'Bearer ' + token } }); const data = await res.json(); setInsights(data.insights) }`

**INVOKE:**
- Parse: `style: dashboard, mode: full, prompt: "AI analytics...", density: compact`
- Load `engines/dashboard.md`
- Custom prompt guides the UI generation decisions

**VERIFY (13 assertions):**

| # | Assertion | How to Check |
|---|-----------|--------------|
| V1 | WebSocket logic in MetricsPanel.tsx intact | grep for `new WebSocket` |
| V2 | `ws.onmessage` handler preserved | grep for `onmessage` |
| V3 | WebSocket cleanup `ws.close()` preserved | grep |
| V4 | `chartRef` and `Chart.js` instantiation logic preserved | grep for `new Chart(` |
| V5 | `chart.destroy()` cleanup preserved | grep |
| V6 | `fetchInsights` with auth header intact | grep for `Authorization` |
| V7 | `setInsights(data.insights)` preserved | grep |
| V8 | Dashboard layout applied (dark sidebar pattern visible in JSX) | check output structure |
| V9 | Compact density applied (small spacing values) | grep for compact spacing classes |
| V10 | Custom prompt influenced UI (dark sidebar referenced in output) | check output narrative references prompt |
| V11 | Token files updated with dashboard palette | read tokens |
| V12 | Polish pass: chart area has proper container sizing | check ChartArea.tsx |
| V13 | Summary shows custom prompt in output | check output |

---

## Scoring & Outcome Rules

After all 12 (or 10/11/13) assertions for a persona:

```
persona_score = (passed_assertions / total_assertions) * 100
```

**Outcome thresholds:**

| Score | Outcome |
|-------|---------|
| 90–100 | EXCELLENT — delete test project, log to cycle history |
| 80–89  | PASS — delete test project, note minor issues |
| 60–79  | FAIL — diagnose, fix skill, commit, log to bug.md |
| < 60   | CRITICAL FAIL — full diagnosis required, multiple fixes, commit each fix separately |

---

## Failure Protocol (when score < 80)

If a persona fails, execute this protocol before moving to the next persona:

### Step F1 — Identify Root Cause
1. Which assertions failed?
2. Which skill file is responsible?
   - Logic preservation failures → `SKILL.md` Step 5 or Step 6
   - Wrong framework detected → `SKILL.md` Step 2
   - Wrong styling detected → `SKILL.md` Step 3
   - Tokens modified when --keep-tokens → `SKILL.md` Step 7
   - God Mode ran standard pipeline → `SKILL.md` God Mode section
   - Engine styles not applied → `engines/<style>.md`
   - Mode not applied correctly → `modes/<mode>.md`
   - Argument parsing error → `parser/commands.md`
   - Polish pass incomplete → `references/ui-craft.md` + SKILL.md Step 11

### Step F2 — Read Failing File
Read the exact skill file responsible. Find the exact instruction that is ambiguous, missing, or wrong.

### Step F3 — Write the Fix
Edit the skill file to fix the root cause. The fix must:
- Be minimal (don't rewrite sections that aren't broken)
- Be specific (add explicit examples if ambiguity caused the failure)
- Not break other personas (check the fix against other personas mentally)

### Step F4 — Log to bug.md
Add a new entry to `bug.md` using the format:
```
### BUG-NNN: [Short title]
- Status: [FIXED yyyy-mm-dd] or [OPEN]
- Persona: [name]
- Command: [command]
- Skill File: [file]
- Symptom: [what failed]
- Root Cause: [why]
- Fix Applied: [what changed]
- Commit: [pending — fill after commit]
- Regression Risk: [which other personas to recheck]
```

### Step F5 — Commit
```bash
git add ui-restructure/<modified-file>
git add bug.md
git commit -m "fix(<skill-file>): <short description of fix>

Caught by TEST-LOOP persona <name>: <failing assertion>.
Root cause: <one sentence>.
"
```

### Step F6 — Re-run the Failing Persona
After fixing, re-run that persona from SETUP → VERIFY. Only proceed to next persona when score ≥ 80 or a second fix has been committed with the residual issue logged.

---

## Post-All-Personas — Cycle Summary

After all 10 personas complete, do the following:

### 1. Compute final scores
```
Total assertions: [sum]
Passed: [sum]
Failed: [sum]
Overall pass rate: [%]
```

### 2. Print per-persona scoreboard
```
Persona    | Score | Outcome   | Bugs Found
-----------|-------|-----------|------------
Maya       | XX%   | PASS      | 0
Dev        | XX%   | PASS      | 0
Priya      | XX%   | FAIL→FIX  | 1
Alex       | XX%   | PASS      | 0
Jordan     | XX%   | PASS      | 0
Sam        | XX%   | FAIL→FIX  | 2
Taylor     | XX%   | PASS      | 0
Morgan     | XX%   | PASS      | 0
Riley      | XX%   | PASS      | 0
Casey      | XX%   | PASS      | 0
TOTAL      | XX%   |           | X bugs
```

### 3. Update bug.md cycle history table
Append a new row to the "Cycle History" table in `bug.md`:
```
| N | YYYY-MM-DD | 10 | X | Y | XX% | [notes] |
```

### 4. Final commit
```bash
git add bug.md
git commit -m "test(loop): cycle N complete — XX% pass rate, Y bugs fixed

Personas: Maya, Dev, Priya, Alex, Jordan, Sam, Taylor, Morgan, Riley, Casey
Bugs fixed: [list]
Open bugs remaining: [count]
"
```

### 5. Clean up workspace
```bash
rm -rf /tmp/ui-test-workspace
```

### 6. Print final line
```
CYCLE COMPLETE — Overall: XX% | Bugs fixed this cycle: Y | Open: Z | Next loop ready.
```

---

## Cross-Cycle Continuity Rules

These rules ensure each loop iteration builds on the last without repeating fixed bugs:

1. **Always read `bug.md` at the start** — load open bugs and check if any current run reproduces them.
2. **If a fixed bug reappears**, mark it `[OPEN]` again with a note "regressed in cycle N" and treat it with priority.
3. **Do not re-fix bugs that are marked `[FIXED]`** — if behavior is now correct, leave them fixed.
4. **BUG numbers are permanent** — never renumber. BUG-001 stays BUG-001 forever.
5. **If a new skill version is released** (version in `SKILL.md` frontmatter changes), run all 10 personas even if they previously passed.

---

## Quick Assertion Checklist (for fast reference during VERIFY)

**Logic Preservation (always check these):**
- [ ] `useState` declarations present
- [ ] `useEffect` hooks present
- [ ] `useMemo`/`useCallback`/`useRef` hooks present
- [ ] All `fetch()` calls present and URLs unchanged
- [ ] All event handlers (`handle*` functions) present
- [ ] Data `.map()` iterations present with `key` props
- [ ] Custom hooks (`use*`) not removed
- [ ] Props interfaces unchanged
- [ ] WebSocket/interval/timeout logic with cleanup preserved

**UI Rebuild (always check these):**
- [ ] Layout classes changed (old flat structure → new structured layout)
- [ ] Typography classes updated
- [ ] Color/token classes updated (unless --keep-tokens)
- [ ] Token files updated (unless --keep-tokens)
- [ ] Polish pass: `transition`, `active:scale`, `focus-visible` present

**Skill Output (always check):**
- [ ] Framework correctly detected and stated
- [ ] Styling system correctly detected
- [ ] Summary line present with Style/Mode/Files/Logic/Tokens/Polish
