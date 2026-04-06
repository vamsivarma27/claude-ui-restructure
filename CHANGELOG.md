# Changelog

All notable changes to `ui-restructure` are documented here.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
Versioning follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

---

## [1.2.0] — 2026-04-06

### Added
- `--god-mode` parameter — user-first redesign simulating 100 user mindsets
- `--remove-tokens` flag — explicit token reset (god mode keeps tokens by default)
- `modes/godmode.md` — 7-phase pipeline: page inventory, friction audit, 6 UX principles, redesign decisions, placement rules, apply changes, report
- `references/user-mindset.md` — 10 Laws of User Behavior + 10 common developer UX mistakes
- `ui-restructure/assets/banner.svg` — README banner image
- Professional README overhaul — full problem/solution, every command documented individually
- CI workflow (`validate.yml`) — validates SKILL.md on every PR
- Release workflow (`release.yml`) — auto GitHub Release on git tag push

### Fixed
- SKILL.md description trimmed to under 1024 characters (SkillsMP validation requirement)
- Repo renamed `claude-ui-restructure` → `ui-restructure` for multi-tool generic naming

---

## [1.1.0] — 2026-04-06

### Added
- `references/ui-craft.md` — world-class frontend engineering reference (motion, micro-interactions, typography, iconography, color theory, shadows, accessibility)
- Step 11 (mandatory polish pass) added to SKILL.md — runs after every rebuild
- Framer Motion spring presets, easing curve guide, duration scale
- 35-point polish checklist
- 5-state interaction model for every component
- WCAG AA contrast enforcement

### Changed
- Skill description updated to mention Stripe/Linear/Vercel quality output
- Version bumped 1.0.0 → 1.1.0

---

## [1.0.1] — 2026-04-06

### Fixed
- Renamed skill from `claude-ui-restructure` to `ui-restructure` — SkillsMP rejects reserved word `claude` in skill names
- Slash command is now `/ui-restructure`

### Added
- `CODEOWNERS` — all changes require `@vamsivarma27` review
- `SECURITY.md` — private vulnerability reporting policy
- `.github/PULL_REQUEST_TEMPLATE.md` — contributor checklist
- `.github/CONTRIBUTING.md` — branch naming, commit format rules

---

## [1.0.0] — 2026-04-06

### Added
- Initial release — `ui-restructure` skill
- `SKILL.md` — 10-step execution pipeline solving UI lock-in
- Style engines: `apple`, `linear`, `minimal`, `dashboard`
- Modes: `full`, `layout`, `theme`, `grid`
- `parser/commands.md` — full argument parsing rules
- `prompts/restructure-flow.md` — master execution flow
- Supports: Next.js, React (Vite/CRA), Vue 3, Tailwind, CSS modules, styled-components, shadcn
- Published to SkillsMP (status: approved)
- Branch protection, secret scanning, Dependabot enabled on GitHub repo

---

## Versioning Rules

| Change type | Version bump | Example |
|---|---|---|
| New `--flag` or new mode/engine | Minor (x.**Y**.0) | Adding `--god-mode` → 1.1.0 → 1.2.0 |
| Bug fix, copy fix, typo | Patch (x.x.**Z**) | Fix broken reference → 1.2.0 → 1.2.1 |
| Breaking change to SKILL.md interface | Major (**X**.0.0) | Rename existing flag → 1.x.x → 2.0.0 |
| Security fix | Patch + release note | Same as patch |
| Docs/meta only | Patch | README update → patch |
