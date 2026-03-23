# ReserveLabs

**EN** Design drift detector for AI-generated codebases.
**TR** Yapay zeka ile uretilen kod tabanlari icin tasarim kaymasi dedektoru.

---

## What is this? / Bu nedir?

**EN** Every component looks right on its own. But after 50 prompts, your app looks like Frankenstein. ReserveLabs catches that.

**TR** Her bileseni tek tek inceleyin, hepsi dogru gorunur. Ama 50 prompt sonra uygulamaniz Frankenstein'a donmustur. ReserveLabs bunu yakalar.

ReserveLabs is a Claude Code skill suite that adds UIX quality checkpoints to vibecoding workflows. It watches for design drift — the gradual loss of visual coherence that happens when AI generates code across many sessions.

---

## The Problem

AI coding tools are great at building individual components. They are terrible at maintaining UI consistency across a full codebase.

After enough prompts, you get:

- **Component duplication** — three different card components, two modal wrappers, four button variants that do the same thing
- **Style inconsistency** — `#3b82f6` here, `blue-500` there, `rgb(59,130,246)` somewhere else — same color, three representations
- **Spacing entropy** — `p-4` on one card, `p-5` on the next, `p-[18px]` on a third. No grid, no rhythm
- **Typography sprawl** — eight font sizes when four would do. Weights all over the place
- **Dependency bloat** — lucide-react AND heroicons AND phosphor-react in the same package.json
- **Missing states** — loading spinners on some pages, nothing on others. Error handling in half the forms

This is design drift. It is invisible at the component level and obvious at the application level. Linters do not catch it. Design systems prevent it only if you already have one. ReserveLabs catches it after the fact, when AI-generated code quality starts degrading.

---

## How it works

```
  Stage 1               Stage 2                Stage 3
  PLANNING              IMPLEMENTATION         PRE-RELEASE
  ────────              ──────────────         ───────────
  Gap analysis          Live drift detection   Full codebase
  before coding         during coding          design review

  "I'm going to         "Check this            "We're ready
   build a dashboard"    component"              to ship"

  ┌─────────┐           ┌─────────┐            ┌─────────┐
  │ Checklist│           │ Targeted│            │ 7-Layer  │
  │ + drift  │    ───>   │ scan of │    ───>    │ full     │
  │ prevention│          │ changed │            │ codebase │
  │ questions│           │ files   │            │ audit    │
  └─────────┘           └─────────┘            └─────────┘
```

**Stage 1** asks the right questions before you write code — do you have a spacing grid? are you reusing existing components or creating duplicates?

**Stage 2** is developer-invoked during implementation. Point it at a component or page and it checks for drift against existing patterns.

**Stage 3** dispatches the design reviewer agent for a full scan: 7 audit layers across every frontend file. This is the pre-release checkpoint.

---

## Quick Start

```bash
git clone https://github.com/reservelab/reservelabs.git
cp -r reservelabs/ your-project/.claude/skills/reservelabs/
```

That is it. No config file required. ReserveLabs reads your Tailwind config, CSS variables, and existing component patterns automatically.

Then in Claude Code:

```
> "I'm going to build a settings page"        → Stage 1 (planning)
> "Check this component for design drift"      → Stage 2 (implementation)
> "Run a full design review before release"    → Stage 3 (pre-release)
```

---

## What it checks

| Layer | What | Looks for |
|---|---|---|
| Spacing Grid | Padding, margin, gap values | Broken grid, arbitrary values like `p-[13px]` |
| Color Discipline | All color references | Hardcoded hex/rgb, palette sprawl (>15 unique colors) |
| Typography | Font sizes, weights, families | Too many sizes, inconsistent weight usage |
| State Coverage | Loading, empty, error states | Missing loading spinners, unhandled errors, no empty states |
| A11y (WCAG 2.2) | Images, buttons, forms, focus, contrast | Missing alt text, unlabeled inputs, no focus-visible |
| Component Dedup | Structural similarity between components | Two components that do the same thing differently |
| Dependency Bloat | package.json redundancies | Multiple icon libs, multiple animation libs, duplicate UI kits |

---

## Example output

```
============================================
  ReserveLabs Design Review Report
============================================
  Files Scanned: 47
  Issues: 3 critical, 8 warning, 4 info
============================================

CRITICAL

[A11Y] src/components/ProductCard.tsx:23
  Image missing alt text.
  Fix: Add descriptive alt prop.

[STATE] src/app/dashboard/page.tsx:45
  Data fetch with no error boundary or catch.
  Fix: Add error state UI.

[A11Y] src/components/ContactForm.tsx:12
  Input "email" has no associated label.
  Fix: Add <label htmlFor="email"> or aria-label.

WARNING

[SPACING] src/components/Header.tsx:8
  Uses p-[18px] — breaks 4px grid.
  Suggestion: p-4 (16px) or p-5 (20px).
  Intentional? [Y/n]

[COLOR] src/components/Badge.tsx:15
  Hardcoded #ef4444 — not a theme color.
  Suggestion: Use text-red-500 or define a semantic token.
  Intentional? [Y/n]

[DEDUP] src/components/Modal.tsx + src/components/Dialog.tsx
  Very similar structure and props — consider merging.
  Intentional? [Y/n]

──────────────────────────────
  Summary
──────────────────────────────
  Spacing Grid      needs-work
  Color Discipline  needs-work
  Typography        good
  State Coverage    poor
  A11y              poor
  Component Dedup   needs-work
  Dependency Bloat  good
──────────────────────────────
```

Assessments are qualitative: **good**, **needs-work**, or **poor**. No percentages. No fake precision.

---

## Configuration

Create `.reservelabs.yml` in your project root. Everything is optional — smart defaults apply.

```yaml
spacing_base: 4               # grid unit in px (default: 4)
a11y_level: AA                 # AA or AAA (default: AA)
max_font_sizes: 6              # flag if exceeded (default: 6)
allow_hardcoded_colors: false  # flag hex/rgb in components (default: false)

exclude:
  - "node_modules"
  - "dist"
  - ".next"

critical:                      # categories that block ship recommendation
  - a11y
  - states
warning:
  - spacing
  - color
  - typography
```

---

## Non-Goals

ReserveLabs is deliberately narrow in scope.

- **Not a linter.** Does not replace ESLint, Stylelint, or Biome. Those check syntax and single-file rules. ReserveLabs checks cross-file design consistency.
- **Not a CI/CD tool.** v1 is interactive only, invoked inside Claude Code.
- **Not a design system.** Does not create tokens, components, or Figma specs. It detects when your existing patterns are drifting.
- **Not a security scanner.** Use [Codex-Sentinel](https://github.com/reservelab/codex-sentinel) for security checkpoints.
- **Not deterministic.** LLM-based analysis. Two runs may produce slightly different results. This is expected and by design.

---

## Known Limitations

1. **LLM-based analysis** — Results are qualitative, not calculated metrics. Findings depend on the model's interpretation of your code.
2. **No persistence between runs** — Each scan starts fresh. There is no history, no trend tracking, no "this got worse since last week."
3. **Qualitative, not quantitative** — You get "needs-work" not "73% consistent." This is intentional. Fake precision is worse than honest assessment.
4. **React/Next.js/Tailwind focus** — v1 has first-class support for this stack. Other frameworks work but with less depth.
5. **Large codebases** — Projects with 100+ frontend files are scanned in chunks. Coverage is comprehensive but analysis quality may vary at scale.

---

## Security

ReserveLabs scans source code. It never copies sensitive data into reports.

When the agent encounters API keys, tokens, passwords, connection strings, email addresses, phone numbers, or PII:

- Reports **only** the file path + line number + `[REDACTED]`
- Suggests remediation: "Move to .env" or "Use environment variable"
- Never echoes the actual value

---

## vs Codex-Sentinel

**Codex-Sentinel** = security checkpoints. Catches vulnerabilities, injection risks, auth gaps.

**ReserveLabs** = design quality checkpoints. Catches UI consistency drift, component duplication, spacing entropy, a11y gaps.

Different domains. Complementary tools. Use both.

---

## Roadmap

**v1 — Claude Code Skill** (current)
- Interactive design lint inside Claude Code sessions
- 3-stage checkpoint system (planning, implementation, pre-release)
- 7-layer design reviewer agent
- React/Next.js/Tailwind support

**v2 — CLI Tool**
```bash
npx reservelabs scan
npx reservelabs scan --stage 3
npx reservelabs scan --fix
```
- Standalone CLI, no Claude Code dependency
- CI/CD integration
- JSON/SARIF output
- Multi-framework support
- Persistent baseline for trend tracking

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

[MIT](LICENSE) — ReserveLab, 2026
