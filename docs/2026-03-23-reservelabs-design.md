# ReserveLabs — Design Spec

> **Date:** 2026-03-23
> **Status:** Approved (reviewed + is-analizi completed)
> **Author:** ReserveLab

---

## 1. Problem Statement

AI-assisted development ("vibecoding") produces code that looks correct file-by-file but drifts into inconsistency across the full codebase. After 50+ prompts, a typical app suffers from:

- **Component duplication** — 3 different Card components doing the same thing
- **Style drift** — first half of the app looks minimal, second half looks material
- **Spacing entropy** — 12 different spacing values instead of a consistent grid
- **Dependency bloat** — 4 icon libraries, 3 animation libraries, all doing the same job
- **Color token sprawl** — 23 unique colors, 8 outside any palette
- **Zombie code** — AI-generated components/imports that are never used
- **Placeholder content** — "Lorem ipsum" and "Your title here" surviving to production

**No existing tool catches this.** Linters check syntax. A11y tools check attributes. Lighthouse checks performance. Nobody checks **design drift** — the gradual loss of visual and structural coherence as AI-generated code accumulates.

## 2. Solution

**ReserveLabs** is a stage-aware UIX quality checkpoint suite for AI-assisted development workflows. It integrates into Claude Code and OpenAI Codex as a skill/agent system.

### Core Value Proposition
"Every component looks right. After 50 prompts your app looks like Frankenstein. ReserveLabs catches that."

### Approach
Hybrid analysis: static baseline rules (zero config) + dynamic context reading (tailwind.config, theme files). Advisory-first: warns, suggests fixes, auto-fixes with developer approval. Never blocks.

### How v1 Analysis Works (Execution Model)
v1 is entirely **LLM-prompted reasoning over source code**. There is no AST parsing, no programmatic analysis, no deterministic scoring. The skill is a set of markdown prompts that instruct the LLM to read files, compare patterns, and report findings.

**Strengths:** Zero setup, flexible, understands intent and context (not just syntax).
**Limitations:** Non-deterministic — two runs on the same codebase may produce slightly different results. Scores are qualitative (good/needs-work/poor), not numerical percentages. Large codebases require chunking.

v2 (CLI tool) will add deterministic AST-based analysis for consistent, repeatable results.

## 3. Target Audience

Solo developers who vibecode with AI tools (Claude Code, Codex, Cursor, etc.). They ship fast but don't notice when design quality degrades across the codebase.

## 4. Supported Stacks

### v1
- React / Next.js
- Tailwind CSS
- shadcn/ui, aceternity, magicui, originui, preline, hyperui

### v2 (future)
- Vue / Nuxt
- Svelte / SvelteKit
- Vanilla CSS / CSS Modules

## 5. Architecture

### 5.1 Repository Structure

```
reservelabs/
├── skill.md                      # Main skill — stage router
├── agents/
│   └── design-reviewer.md        # Autonomous design review agent
├── checklists/
│   ├── planning.md               # Stage 1: Design gap analysis
│   ├── implementation.md         # Stage 2: Live drift detection
│   └── prerelease.md             # Stage 3: Full design QA
├── references/
│   ├── visual-rules.md           # Spacing + color + typography rules
│   ├── ux-patterns.md            # A11y + state coverage patterns
│   └── responsive.md             # Breakpoint + responsive rules
├── supported-stacks/
│   └── react-nextjs-tailwind.md  # Stack-specific detection rules
├── .reservelabs.example.yml      # Optional config example
├── README.md                     # Bilingual (TR + EN)
├── INSTALL.md                    # Claude Code + Codex setup
├── LICENSE                       # MIT
└── CHANGELOG.md
```

### 5.2 Stage-Aware Checkpoint System

#### Stage 1 — Planning (Manual trigger)

Activated when developer says "I'm going to build X" or invokes skill directly.

**Actions:**
- Design gap analysis for the planned component/page
- Checks: dark mode considered? Empty/error/loading states planned? Mobile responsive behavior defined?
- If `tailwind.config` exists: reads project tokens and reminds developer of available design system
- If no config: recommends sensible defaults (4px grid, WCAG AA)

**Output:** Checklist format — developer confirms or adds missing items.

#### Stage 2 — Implementation (Developer-invoked after edits)

Activated when developer says "check this" or "review my component" after writing/modifying code. **Note:** Claude Code skills cannot auto-trigger on file changes. The developer must explicitly invoke the check.

**P0 checks (v1 must-have):**
- Spacing grid adherence (4px/8px base)
- aria-label presence on interactive elements
- State coverage (loading/empty/error)
- **NEW: Drift detection** — compares new component against existing patterns

**P1 checks (v1 nice-to-have):**
- Color palette compliance (hardcoded hex detection)
- Typography consistency (font-size/weight count)
- Touch target minimums (44px)

**Output format:**
```
⚠️ 3 UIX issues found:

1. [DRIFT] This Card uses rounded-lg but 4 existing Cards use rounded-xl
   Auto-fix to rounded-xl? [Y/n]

2. [SPACING] gap-3 (12px) → gap-4 (16px) recommended (4px grid)
   Auto-fix? [Y/n]

3. [STATE] No error state — API failure shows blank screen
   Add error boundary template? [Y/n]
```

#### Stage 3 — Pre-Release (Manual trigger)

Activated when developer says "done" or before commit/PR.

**Actions:**
- Full codebase scan via design-reviewer agent
- Design consistency report with qualitative ratings (good / needs-work / poor per category)
- Drift detection across all components
- A11y final checklist
- Responsive test plan generation

### 5.3 Design Reviewer Agent

Autonomous agent (like mete but for UIX). Can be called independently or by Stage 3.

**7 Audit Layers:**

| # | Layer | Description | Version |
|---|-------|-------------|---------|
| 1 | Spacing Consistency | Same grid across all files? | v1 |
| 2 | Color Discipline | Off-palette colors? Hardcoded hex vs tokens? | v1 |
| 3 | Typography Order | How many distinct font-size/weight/family? | v1 |
| 4 | State Coverage | Which components lack loading/empty/error? | v1 |
| 5 | A11y Baseline | aria-label, alt text, focus visible, contrast | v1 |
| 6 | Component Deduplication | Similar components that should be merged | v1 |
| 7 | Dependency Bloat | Redundant libraries doing the same job | v1 |

**v2 additions:**
- Responsive gap analysis
- Cross-component style consistency scoring
- Placeholder content detection
- Zombie code detection

**Output:** Scored report with visual bars, grouped by severity (critical/warning/info), with auto-fix offers.

```
╔══════════════════════════════════════╗
║   ReserveLabs Design Review Report  ║
╠══════════════════════════════════════╣
║ Files Scanned: 23                   ║
║ Issues: 8 warning, 2 critical       ║
╚══════════════════════════════════════╝

🔴 CRITICAL (2)
1. [DEDUP] Card.tsx and ProductCard.tsx look very similar — likely merge candidates
2. [A11Y] 3 images without alt text

⚠️ WARNING (8)
3. [DRIFT] 2 distinct button styles detected across 15 files
4. [BLOAT] lucide-react + heroicons + phosphor-icons — pick one
...

📊 CONSISTENCY (qualitative — LLM-assessed, not calculated)
Spacing:     ████████░░ good
Color:       ██████░░░░ needs-work
Typography:  ███████░░░ needs-work
States:      █████░░░░░ poor
A11y:        ████████░░ good
Dedup:       ███████░░░ needs-work
```

**Note:** Scores are qualitative assessments by the LLM, not calculated metrics. They may vary slightly between runs. v2 CLI will provide deterministic scores.

### 5.4 Hybrid Analysis Model

**Layer 1 — Static Baseline (zero config):**
- 4px spacing grid
- WCAG AA contrast ratios
- 44px minimum touch targets
- Required states: loading, empty, error
- Max 6 font sizes, 3 font weights
- No hardcoded hex colors (when Tailwind detected)

**Layer 2 — Dynamic Context (auto-detected):**
- Reads `tailwind.config.ts` / `tailwind.config.js` for theme tokens
- Reads `globals.css` / `theme.ts` for custom properties
- Scans existing components to extract implicit design patterns
- Uses detected patterns as enforcement baseline

**Fallback:** If no config files found, Layer 1 defaults apply.

## 6. Configuration

Optional `.reservelabs.yml` — 12 lines, 30 seconds to understand:

```yaml
# .reservelabs.yml (all optional, smart defaults apply)

spacing_base: 4
a11y_level: AA          # AA | AAA
max_font_sizes: 6
allow_hardcoded_colors: false

exclude:
  - "node_modules"
  - "dist"
  - ".next"

critical:
  - a11y
  - states
warning:
  - spacing
  - color
  - typography
```

## 7. Advisory-First Model

ReserveLabs never blocks the developer. Three response levels:

1. **Warn** — "This spacing doesn't match your grid" (always)
2. **Suggest fix** — "Change gap-3 to gap-4" (always)
3. **Auto-fix with approval** — "Fix? [Y/n]" (when fix is deterministic)

If developer declines a fix, the issue is logged but development continues.

## 8. Language Strategy

- Skill content, checklists, references: **English**
- README: **Bilingual (TR + EN)** with flag markers
- Agent output messages: **English** (internationalization possible in v2)

## 9. Installation

### Claude Code
1. Clone repo
2. Copy `skill.md` to `.claude/skills/`
3. Copy `agents/`, `checklists/`, `references/`, `supported-stacks/` alongside
4. Optionally add `.reservelabs.yml` to project root

### OpenAI Codex
1. Clone repo
2. Place in `.codex/` directory
3. Configure Codex agents

## 10. Roadmap

### v1 (Phase 1 — Skill-based)
- [ ] 3-stage checkpoint system (planning, implementation, pre-release)
- [ ] React/Next.js/Tailwind support
- [ ] Zero-config + optional .reservelabs.yml
- [ ] Design reviewer agent
- [ ] Component deduplication detection
- [ ] Style drift detection
- [ ] Dependency bloat detection
- [ ] Bilingual README (TR + EN)

### v2 (Phase 2 — CLI tool)
- [ ] `npx reservelabs scan` CLI command
- [ ] AST-based analysis (deterministic results)
- [ ] CSS similarity algorithms
- [ ] Responsive gap analysis
- [ ] Placeholder content detection
- [ ] Zombie code detection
- [ ] Vue/Svelte support
- [ ] CI/CD integration (GitHub Actions)
- [ ] VS Code extension

## 11. Success Criteria

- GitHub stars > 100 in first month (concept resonance)
- At least 5 community contributions in first 3 months
- "Design drift" term gains traction in vibecoding discourse
- Phase 2 CLI justified by Phase 1 feedback

## 12. Risks

| # | Risk | Impact | Mitigation |
|---|------|--------|------------|
| R1 | **False positives erode trust** — flags valid design choices as drift | High | 3 confidence levels (HIGH/MEDIUM/INFO). INFO asks "intentional?" before flagging |
| R2 | **Context window limits** — full scan on 100+ files exceeds LLM context | High | Chunk strategy: file list first → group analysis → merge results |
| R3 | **LLM analysis inconsistent** — different results on same codebase | Medium | Qualitative ratings (good/needs-work/poor), not percentages. Deterministic checklists |
| R4 | **"Design drift" not a searchable term** | Medium | README uses existing keywords: "UI consistency", "design lint", "vibecoding quality" |
| R5 | **No persistence between runs** — can't track dismissed/fixed issues | Medium | v1 accepts this limitation. v2 CLI adds state file |
| R6 | **Tailwind v4 config format change** | Medium | Support both v3 (tailwind.config.ts) and v4 (CSS-based config) |
| R7 | **Codex compatibility uncertain** | Medium | Claude Code first, Codex best-effort |
| R8 | **Platform lock-in** — skill format changes | Low-Medium | Logic lives in checklists, only skill.md is platform-specific |
| R9 | **"Me too" perception vs Codex-Sentinel** | Low | Different domain (UIX vs security) + novel "drift" concept |
| R10 | **Low adoption** | Medium | Strong README + X launch + dogfooding examples |
| R11 | **Scope creep** | High | Strict v1/v2 boundary; ship v1 fast |
| R12 | **Data leakage in reports** — agent copies secrets/tokens/PII into output | High | Redaction rules: never copy API keys, tokens, passwords, emails, phone numbers. Show only file:line + "REDACTED" label |
| R13 | **KVKK/privacy risk** — scan reports may contain personal data | Medium | "Never report" list + redaction policy in agent prompt |

## 13. Non-Goals (What ReserveLabs Does NOT Do)

- NOT a linter — does not replace ESLint, Stylelint, or Biome
- NOT a CI/CD tool — does not run in pipelines (v1 is interactive only)
- NOT a design system — does not create or enforce a design system, only detects drift from implicit patterns
- NOT a testing framework — does not run visual regression tests
- NOT a security scanner — does not check for vulnerabilities (use Codex-Sentinel for that)
- NOT deterministic — v1 is LLM-based, results may vary between runs

## 14. Security & Data Handling

### Redaction Rules
The design-reviewer agent scans source code. It MUST never copy the following into its output:
- API keys, tokens, secrets, passwords
- Email addresses, phone numbers, national ID numbers
- Customer names combined with internal data
- Database connection strings
- Environment variable values

When detected, the agent must:
1. Report only the file path and line number
2. Label as `[REDACTED]`
3. Suggest remediation (e.g., "move to .env", "use environment variable")

### KVKK/Privacy Note
Even though ReserveLabs does not process personal data directly, its scan reports could inadvertently include personal data found in source code (hardcoded emails, test user data, etc.). The agent prompt must include explicit instructions to redact such data.

## 15. Accessibility Standard Mapping

A11y checks in v1 reference WCAG 2.2 success criteria. Mapping table:

| Check | WCAG 2.2 Criterion | Level |
|-------|-------------------|-------|
| Image alt text | 1.1.1 Non-text Content | A |
| Button/link accessible name | 4.1.2 Name, Role, Value | A |
| Form input labels | 1.3.1 Info and Relationships | A |
| Heading hierarchy | 1.3.1 Info and Relationships | A |
| Focus visible | 2.4.7 Focus Visible | AA |
| Color contrast | 1.4.3 Contrast (Minimum) | AA |
| Touch target size | 2.5.8 Target Size (Minimum) | AA |

This mapping enables traceable, verifiable findings. Each a11y finding in the report should reference the relevant WCAG criterion.

## 16. Supported Stack Details

v1 lists shadcn/ui, aceternity, magicui, originui, preline, hyperui as supported component libraries. In practice, the skill treats these as Tailwind-based component sets and does not have library-specific detection rules. "Support" means the skill understands their patterns and won't flag their standard usage as drift. Library-specific rules may be added in v2.

## 14. Codex Compatibility Note

OpenAI Codex installation instructions in Section 9 are intentionally brief. Codex agent format may differ from Claude Code skills. v1 prioritizes Claude Code; Codex support is best-effort and will be refined based on community feedback.
