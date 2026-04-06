# Command Parser

Parse and validate `/ui-restructure` command arguments.

---

## Command Grammar

```
/ui-restructure [OPTIONS]

OPTIONS:
  --god-mode                 User-first redesign mode (see modes/godmode.md)
  --style    <style-name>    Named style engine to apply
  --mode     <mode-name>     Restructure mode (ignored when --god-mode is set)
  --prompt   "<string>"      Custom UI description
  --keep-tokens              Preserve existing tokens (flag)
  --remove-tokens            Explicitly reset and rebuild all tokens
  --grid     <grid-type>     Force grid layout type
  --density  <density-type>  Apply density preset
```

---

## Argument Values

### `--god-mode`

Boolean flag (no value). Activates the user-first redesign pipeline.

When set:
- Read `references/user-mindset.md` and `modes/godmode.md` fully
- Skip standard modes (full/layout/theme/grid) — god mode replaces them
- Run the 7-phase god mode pipeline
- Tokens are **preserved by default** in god mode
- Add `--remove-tokens` alongside `--god-mode` to also reset tokens

God Mode can combine with:
- `--remove-tokens` — resets tokens after god mode redesign
- `--style` — defines which token system to apply when `--remove-tokens` is also used

God Mode cannot combine with:
- `--mode` — ignored when `--god-mode` is present (warn user)
- `--keep-tokens` — redundant in god mode (tokens are kept by default; warn user)

### `--remove-tokens`

Boolean flag (no value). Explicitly resets all design tokens.

When set:
- Run Step 7 (token reset) and regenerate tokens with the active style engine
- Default behavior WITHOUT `--remove-tokens` in god mode: tokens preserved
- Default behavior WITHOUT `--remove-tokens` in standard mode: tokens ARE reset

This exists primarily for god mode, where tokens are kept by default.
In standard mode, tokens are reset by default unless `--keep-tokens` is set.

### `--style`

| Value | Description | Engine File |
|---|---|---|
| `apple` | Apple HIG glassmorphism style | `engines/apple.md` |
| `linear` | Linear.app compact dark style | `engines/linear.md` |
| `minimal` | Clean neutral typography-first | `engines/minimal.md` |
| `dashboard` | Data-dense analytics style | `engines/dashboard.md` |

Default: `minimal` (if no `--style` provided)

### `--mode`

| Value | Description | Mode File |
|---|---|---|
| `full` | Rebuild everything | `modes/full.md` |
| `layout` | Structure only, preserve tokens | `modes/layout.md` |
| `theme` | Tokens only, preserve layout | `modes/theme.md` |
| `grid` | Convert lists to card grids | `modes/grid.md` |

Default: `full` (if no `--mode` provided)

### `--prompt`

Free-form string describing desired UI.

Examples:
```
--prompt "AI analytics dashboard with dark sidebar and floating metric cards"
--prompt "Clean SaaS pricing page with toggle and feature comparison table"
--prompt "Mobile-first onboarding flow with progress steps"
```

When `--prompt` is provided:
- Combine prompt with style engine definition
- Use mode constraints
- Generate UI that matches the natural language description

### `--keep-tokens`

Boolean flag (no value).

When set:
- Skip Step 6 (token reset)
- Skip Step 7 token regeneration
- Only rebuild layout/structure using existing token classes

### `--grid`

| Value | Description |
|---|---|
| `cards` | Responsive card grid (default when --grid is set) |
| `list` | Force list layout (reverses existing grids to lists) |

### `--density`

| Value | Spacing multiplier | Description |
|---|---|---|
| `compact` | 0.75x base | Tight — more content visible |
| `comfortable` | 1.0x base | Default — balanced |
| `spacious` | 1.5x base | Airy — breathable layout |

---

## Parsing Rules

1. Arguments can appear in any order
2. `--prompt` value must be quoted if it contains spaces
3. Unknown flags should be ignored (warn user)
4. Conflicting flags:
   - `--keep-tokens` + `--mode theme` → warn: theme mode replaces tokens; `--keep-tokens` is ignored
   - `--mode layout` + `--mode theme` → error: only one mode allowed; use first occurrence
5. If no arguments at all → run `/restructure --style minimal --mode full`

---

## Command Examples with Parsed Output

### Example 1
```
/restructure
```
Parsed:
```
style:        minimal
mode:         full
prompt:       none
keep-tokens:  false
grid:         none
density:      comfortable
```

### Example 2
```
/restructure --style apple --density spacious
```
Parsed:
```
style:        apple
mode:         full
prompt:       none
keep-tokens:  false
grid:         none
density:      spacious
```

### Example 3
```
/restructure --style linear --mode layout --grid cards
```
Parsed:
```
style:        linear (layout rules only)
mode:         layout
prompt:       none
keep-tokens:  false
grid:         cards
density:      comfortable
```

### Example 4
```
/restructure --prompt "enterprise SaaS dashboard" --style dashboard --mode full --density compact
```
Parsed:
```
style:        dashboard
mode:         full
prompt:       "enterprise SaaS dashboard"
keep-tokens:  false
grid:         none
density:      compact
```

### Example 5
```
/restructure --keep-tokens --mode layout --style apple
```
Parsed:
```
style:        apple (layout rules only)
mode:         layout
prompt:       none
keep-tokens:  true
grid:         none
density:      comfortable
```

---

## Validation Output

After parsing, output parsed values before executing:

```
Restructure plan:
  Style:        [value]
  Mode:         [value]
  Prompt:       [value or "none"]
  Keep tokens:  [true/false]
  Grid:         [value or "default"]
  Density:      [value]

Proceeding with restructure...
```
