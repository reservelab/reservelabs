# Changelog

## [0.2.0] - 2026-07-28

### Added
- Six Tailwind v4 drift rules: `STACK-TW-003` dead theme variable,
  `STACK-TW-004` broken theme bridge, `STACK-TW-005` split source of truth,
  `STACK-TW-006` color format drift, `STACK-TW-007` custom utility duplication,
  `STACK-TW-008` arbitrary value escape
- Full v4 theme namespace table (`--color-*`, `--text-*`, `--font-weight-*`,
  `--tracking-*`, `--leading-*`, `--spacing`, `--container-*`, `--shadow-*`,
  `--ease-*`, and the rest)
- Three-bucket classification for CSS custom properties:
  registered token / runtime token / rogue variable
- shadcn/ui on Tailwind v4 section: `tw-animate-css`, OKLCH palette,
  `:root` + `@theme inline` bridge
- v4 scan scope guidance — `@source`, `source(…)`, `source(none)` replace v3's `content`

### Fixed
- **`--font-size-*` is not a Tailwind v4 namespace** — font sizes live under
  `--text-*`. The old mapping produced tokens that generate no utility.
- **`@theme inline` was described as CSS-only** — it does generate utilities;
  it only changes how the value is resolved
- **Version detection no longer treats `tailwind.config.*` as proof of v3** — v4 can
  load it via `@config`. The `tailwindcss` version in package.json is now the
  primary evidence, with the CSS entry point as fallback
- Off-scale numeric utilities (`p-17`, `w-29`) are no longer off-grid findings in v4 —
  the spacing scale is derived dynamically via `calc(var(--spacing) * n)`
- Color comparisons now require normalization before flagging (OKLCH vs hex)
- Context loading in `skill.md`, both checklists, `visual-rules.md` and
  `design-reviewer.md` now reads the v4 theme graph instead of assuming a JS config

## [0.1.0] - 2026-03-23

### Added
- Stage-aware UIX checkpoint system (planning, implementation, pre-release)
- Design reviewer agent with 7 audit layers
- React/Next.js/Tailwind support
- Zero-config with optional .reservelabs.yml
- Bilingual README (TR + EN)
- Reference rules: visual, UX patterns, responsive
- WCAG 2.2 mapping for accessibility checks
- Redaction rules for secrets/PII in scan reports
