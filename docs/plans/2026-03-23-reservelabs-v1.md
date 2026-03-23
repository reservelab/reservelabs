# ReserveLabs v1 Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a stage-aware UIX design drift detector skill suite for Claude Code, with a design reviewer agent, 3 stage checklists, and reference rules.

**Architecture:** Markdown-based skill system. `skill.md` is the stage router that detects intent via keywords, loads the right checklist, reads project context (tailwind.config, .reservelabs.yml), and produces advisory output. `design-reviewer.md` is an autonomous agent for full-codebase scans with chunk strategy for large projects.

**Tech Stack:** Pure markdown (Claude Code skill format). No runtime dependencies. No build step. No npm. Just .md files + 1 example .yml.

**Spec:** `docs/2026-03-23-reservelabs-design.md`
**Business Analysis:** `docs/is-analizi-reservelabs.md`

---

## File Structure

```
reservelabs/
├── skill.md                          # Stage router + context engine (CREATE)
├── agents/
│   └── design-reviewer.md            # Full-scan design review agent (CREATE)
├── checklists/
│   ├── planning.md                   # Stage 1 checklist (CREATE)
│   ├── implementation.md             # Stage 2 checklist (CREATE)
│   └── prerelease.md                 # Stage 3 checklist (CREATE)
├── references/
│   ├── visual-rules.md               # Spacing + color + typography (CREATE)
│   ├── ux-patterns.md                # A11y + state coverage (CREATE)
│   └── responsive.md                 # Breakpoints + responsive (CREATE)
├── supported-stacks/
│   └── react-nextjs-tailwind.md      # Stack-specific rules (CREATE)
├── .reservelabs.example.yml          # Config example (CREATE)
├── .gitignore                        # Git ignore rules (CREATE)
├── CONTRIBUTING.md                   # Contribution guide (CREATE)
├── README.md                         # Bilingual TR+EN (CREATE)
├── INSTALL.md                        # Setup guide (CREATE)
├── LICENSE                           # MIT (CREATE)
├── CHANGELOG.md                      # v1 release notes (CREATE)
└── docs/                             # Already exists
    ├── 2026-03-23-reservelabs-design.md
    ├── is-analizi-reservelabs.md
    └── plans/
        └── 2026-03-23-reservelabs-v1.md
```

**Responsibility map:**
- `skill.md` — Entry point. Detects stage, reads context, delegates to checklists. ~200 lines.
- `agents/design-reviewer.md` — Standalone agent prompt. Runs 7 audit layers with chunk strategy. ~250 lines.
- `checklists/*.md` — Pure checklist content, loaded by skill.md. ~80-120 lines each.
- `references/*.md` — Rule definitions. Referenced by checklists. ~60-100 lines each.
- `supported-stacks/*.md` — Stack-specific overrides. ~80 lines.

---

## Chunk 1: Foundation (Tasks 1-4)

### Task 1: LICENSE + CHANGELOG + .gitignore + .reservelabs.example.yml

**Files:**
- Create: `LICENSE`
- Create: `CHANGELOG.md`
- Create: `.gitignore`
- Create: `.reservelabs.example.yml`

- [ ] **Step 1: Create MIT LICENSE**

```
MIT License

Copyright (c) 2026 ReserveLab

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 2: Create CHANGELOG.md**

```markdown
# Changelog

## [0.1.0] - 2026-03-23

### Added
- Stage-aware UIX checkpoint system (planning, implementation, pre-release)
- Design reviewer agent with 7 audit layers
- React/Next.js/Tailwind support
- Zero-config with optional .reservelabs.yml
- Bilingual README (TR + EN)
- Reference rules: visual, UX patterns, responsive
```

- [ ] **Step 3: Create .gitignore**

```
.DS_Store
*.swp
*.swo
*~
.idea/
.vscode/
```

- [ ] **Step 4: Create .reservelabs.example.yml**

```yaml
# ReserveLabs Configuration (all optional — smart defaults apply)
# Copy to your project root as .reservelabs.yml

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

- [ ] **Step 5: Commit**

```bash
git add LICENSE CHANGELOG.md .gitignore .reservelabs.example.yml
git commit -m "feat: add LICENSE, CHANGELOG, .gitignore, and example config"
```

---

### Task 2: Reference Files (3 files)

**Files:**
- Create: `references/visual-rules.md`
- Create: `references/ux-patterns.md`
- Create: `references/responsive.md`

- [ ] **Step 1: Create references/visual-rules.md**

Content: Spacing system rules (4px/8px grid, common Tailwind spacing tokens, what counts as "off-grid"), color discipline rules (hardcoded hex detection, palette enforcement, token usage), typography rules (max font sizes, weight consistency, scale adherence).

Each rule must include:
- Rule ID (e.g., `VS-001`)
- What to check
- Why it matters (1 sentence)
- What "good" looks like (code example)
- What "bad" looks like (code example)
- Confidence level: HIGH (definite issue) / MEDIUM (likely issue) / INFO (might be intentional)
- Auto-fix: yes/no + fix template

- [ ] **Step 2: Create references/ux-patterns.md**

Content: A11y baseline rules (aria-label on interactive elements, alt text on images, focus-visible, color contrast, semantic HTML), state coverage rules (loading state required for async operations, empty state for lists/tables, error state for API calls, error boundaries).

Same rule format as visual-rules.md.

**WCAG 2.2 Mapping Required:** Each a11y rule MUST reference the corresponding WCAG 2.2 success criterion:
- Image alt text → WCAG 1.1.1 Non-text Content (A)
- Button/link accessible name → WCAG 4.1.2 Name, Role, Value (A)
- Form input labels → WCAG 1.3.1 Info and Relationships (A)
- Heading hierarchy → WCAG 1.3.1 Info and Relationships (A)
- Focus visible → WCAG 2.4.7 Focus Visible (AA)
- Color contrast → WCAG 1.4.3 Contrast Minimum (AA)
- Touch target size → WCAG 2.5.8 Target Size Minimum (AA)

- [ ] **Step 3: Create references/responsive.md**

Content: Breakpoint rules (mobile-first approach, standard breakpoints: 640/768/1024/1280/1536px), responsive patterns (no fixed widths in container elements, text truncation on small screens, touch target 44px minimum, horizontal scroll prevention).

Same rule format. Note: responsive checks are lighter in v1 (advisory only, not deep analysis).

- [ ] **Step 4: Commit**

```bash
git add references/
git commit -m "feat: add reference rules — visual, UX patterns, responsive"
```

---

### Task 3: Supported Stack — React/Next.js/Tailwind

**Files:**
- Create: `supported-stacks/react-nextjs-tailwind.md`

- [ ] **Step 1: Create react-nextjs-tailwind.md**

Content:
- How to detect this stack (presence of `next.config.*`, `tailwind.config.*`, `postcss.config.*`)
- How to read Tailwind config (theme.extend.colors, theme.extend.spacing, theme.extend.fontSize)
- Common component libraries and their patterns:
  - shadcn/ui: uses `cn()` utility and CSS variables
  - aceternity: uses Framer Motion for animations
  - magicui: uses Tailwind animations
  - originui: Tailwind-based, similar to shadcn patterns
  - preline: Tailwind utility-first components
  - hyperui: Pure Tailwind, no JS dependencies
- Stack-specific checks:
  - Next.js Image component vs `<img>` tag
  - `use client` / `use server` directive presence
  - Tailwind `@apply` overuse detection
  - className duplication across components (sign of copy-paste from AI)
- What NOT to flag (intentional patterns in these libraries)

- [ ] **Step 2: Commit**

```bash
git add supported-stacks/
git commit -m "feat: add React/Next.js/Tailwind stack support"
```

---

### Task 4: skill.md — Main Stage Router

> **NOTE:** skill.md defines the contract (stage detection, context engine, output format, confidence levels) that checklists and the agent must follow. It MUST be written before checklists.

**Files:**
- Create: `skill.md`

This task was previously Task 7 in Chunk 3. Moved here because skill.md defines the contract for all downstream files.

- [ ] **Step 1: Create skill.md**

Full content as defined in original Task 7 below in Chunk 3 (the complete skill.md with stage detection, context engine, output conventions). Write the FULL skill.md as specified there.

- [ ] **Step 2: Commit**

```bash
git add skill.md
git commit -m "feat: add main skill — stage router + context engine"
```

---

## Chunk 2: Checklists (Tasks 5-7)

### Task 5: Stage 1 — Planning Checklist

**Files:**
- Create: `checklists/planning.md`

- [ ] **Step 1: Create checklists/planning.md**

Structure:
```markdown
# Stage 1: Planning Checkpoint

## Trigger
Developer says: "I'm going to build...", "plan...", "design...", or invokes directly.

## Context Gathering
Before asking questions, check:
1. Does .reservelabs.yml exist? → Read overrides
2. Does tailwind.config exist? → Extract theme tokens
3. What components already exist? → List for drift comparison baseline

## Gap Analysis Questions
For the planned component/page, check each:

### Visual Design
- [ ] Color scheme decided? (using existing palette or new?)
- [ ] Typography choices made? (heading levels, body text)
- [ ] Spacing approach defined? (consistent with existing components?)
- [ ] Dark mode behavior planned?
- [ ] Icon library chosen? (same as existing or new?)

### UX States
- [ ] Loading state designed?
- [ ] Empty state designed?
- [ ] Error state designed?
- [ ] Success/confirmation feedback planned?
- [ ] Partial data state considered?

### Accessibility
- [ ] Keyboard navigation planned?
- [ ] Screen reader experience considered?
- [ ] Color contrast checked?
- [ ] Focus management for modals/overlays?

### Responsive
- [ ] Mobile layout planned?
- [ ] Tablet behavior defined?
- [ ] Touch targets ≥44px?

### Drift Prevention
- [ ] Similar existing components identified?
- [ ] Reuse vs. new component decision made?
- [ ] Consistent with existing patterns?

## Output Format
Present as checklist. For each unchecked item, explain why it matters
(1 sentence) and suggest a default if developer has no preference.
```

- [ ] **Step 2: Commit**

```bash
git add checklists/planning.md
git commit -m "feat: add Stage 1 planning checklist"
```

---

### Task 6: Stage 2 — Implementation Checklist

**Files:**
- Create: `checklists/implementation.md`

- [ ] **Step 1: Create checklists/implementation.md**

Structure:
```markdown
# Stage 2: Implementation Checkpoint

## Trigger
Developer says: "check this", "review my component", "implementation review",
or after writing/modifying .tsx/.jsx/.css files.

## Context Loading
1. Read .reservelabs.yml if exists → overrides
2. Read tailwind.config if exists → theme tokens
3. Scan src/components/ or src/app/ → existing patterns baseline
4. Read the specific file(s) developer is asking about

## Checks (in priority order)

### P0 — Critical (always run)

#### Drift Detection
- Compare this component's patterns against existing components:
  - Border radius: same as majority?
  - Shadow style: consistent?
  - Padding/margin patterns: matching?
  - Button/card/input styling: consistent with existing?
- Confidence: MEDIUM (might be intentional variation)
- If different: "This component uses rounded-lg but 4 existing components
  use rounded-xl. Intentional? [Y/n]"

#### Spacing Grid
- Check all spacing values against base grid (default 4px)
- Flag: arbitrary values like p-[13px], gap-[7px]
- Flag: inconsistent spacing within same component
- Confidence: HIGH for arbitrary values, MEDIUM for inconsistency
- Auto-fix: suggest nearest grid value

#### A11y Baseline
- Interactive elements: button, a, input, select, textarea → aria-label?
- Images: alt text present and meaningful (not "image", "photo")?
- Headings: correct hierarchy (no skipping h1→h3)?
- Confidence: HIGH (these are always issues)
- Auto-fix: suggest aria-label/alt text based on context

#### State Coverage
- Component has async data? → loading state exists?
- Component renders a list? → empty state exists?
- Component calls API? → error handling exists?
- Confidence: HIGH for missing states
- Auto-fix: offer template for missing state

### P1 — Important (run if not skipped)

#### Color Discipline
- Hardcoded hex/rgb values → should use Tailwind class or CSS variable
- Colors not in theme palette
- Confidence: MEDIUM
- Auto-fix: suggest nearest palette color

#### Typography Consistency
- Count distinct font sizes in component
- Compare against project-wide count (max from config, default 6)
- Confidence: INFO
- Auto-fix: no (needs design decision)

#### Touch Targets
- Clickable elements < 44px in either dimension
- Confidence: HIGH on mobile, MEDIUM on desktop
- Auto-fix: suggest min-h-11 min-w-11

## Output Format
Group by confidence level. Show file:line for each issue.
Offer auto-fix where available. Ask "intentional?" for MEDIUM/INFO items.
```

- [ ] **Step 2: Commit**

```bash
git add checklists/implementation.md
git commit -m "feat: add Stage 2 implementation checklist"
```

---

### Task 7: Stage 3 — Pre-Release Checklist

**Files:**
- Create: `checklists/prerelease.md`

- [ ] **Step 1: Create checklists/prerelease.md**

Structure:
```markdown
# Stage 3: Pre-Release Checkpoint

## Trigger
Developer says: "done", "ready for review", "pre-release check",
"final review", or before commit/PR.

## Execution Model
This stage invokes the design-reviewer agent for a full codebase scan.
If the project is small (<30 frontend files), run directly.
If large (30+), use chunk strategy (see design-reviewer.md).

## Pre-Review Quick Checks
Before full scan, quickly check:
1. Any TODO/FIXME/HACK comments left in code?
2. Any console.log/console.error left?
3. Any placeholder text ("Lorem ipsum", "TODO", "Your title here")?
4. Any commented-out code blocks?

## Full Review Delegation
Pass to design-reviewer agent with:
- Project root path
- .reservelabs.yml overrides (if any)
- Tailwind config tokens (if any)
- List of frontend files to scan

## Post-Review Actions
After receiving design-reviewer report:
1. Present summary to developer
2. Group by severity (critical first)
3. For each critical issue: offer auto-fix
4. Generate responsive test plan:
   "Test these pages at: 375px, 768px, 1024px, 1440px"
5. Generate a11y test suggestion:
   "Run axe-core or use browser a11y inspector on key pages"

## Output Format
Design reviewer report + actionable next steps.
```

- [ ] **Step 2: Commit**

```bash
git add checklists/prerelease.md
git commit -m "feat: add Stage 3 pre-release checklist"
```

> **NOTE:** Task 6 (pre-release) includes placeholder content checks (Lorem ipsum, TODO, commented-out code). This is a deliberate v1 inclusion of a v2 spec item — it's trivial to implement and high-value.

---

## skill.md Full Content (referenced by Task 4, Step 1)

The full content that Task 4 should produce:

```markdown
---
name: reservelabs
description: Design drift detector for AI-generated codebases.
  Stage-aware UIX quality checkpoints — catches visual inconsistency,
  component duplication, spacing entropy, and accessibility gaps.
  Supports React/Next.js/Tailwind. Zero config required.
---

# ReserveLabs — Design Drift Detector

You are a UIX quality advisor. Your job is to catch design drift in
AI-generated codebases — the gradual loss of visual and structural
coherence that happens after many AI prompts.

## Non-Goals
- NOT a linter (does not replace ESLint/Stylelint/Biome)
- NOT a CI/CD tool (v1 is interactive only)
- NOT a design system (does not create one, only detects drift)
- NOT a testing framework (no visual regression)
- NOT a security scanner (use Codex-Sentinel for that)
- NOT deterministic (LLM-based, results may vary)

## Core Principles
1. ADVISORY-FIRST: Never block the developer. Warn, suggest, fix with approval.
2. CONFIDENCE LEVELS: Tag every finding as HIGH / MEDIUM / INFO.
3. HONEST: You are LLM-based. Your assessments are qualitative, not calculated metrics.
4. DRIFT-FOCUSED: Individual file correctness is not your job (linters do that).
   Cross-file CONSISTENCY is your job.
5. REDACT SECRETS: NEVER copy API keys, tokens, passwords, emails, phone numbers,
   or PII into your output. Show only file:line + [REDACTED]. Suggest remediation.

## Stage Detection
Detect which stage based on developer's message:
- Keywords "build", "create", "plan", "design", "going to" → Stage 1 (Planning)
- Keywords "check", "review", "look at", "implementation" → Stage 2 (Implementation)
- Keywords "done", "ready", "final", "pre-release", "ship" → Stage 3 (Pre-Release)
- If unclear, ask: "Which stage? (1) Planning a new component,
  (2) Reviewing implementation, (3) Final pre-release check"

## Context Engine
Before running any stage, gather context:

### Step 1: Read config
- Check for .reservelabs.yml in project root
- If found: parse overrides (spacing_base, a11y_level, max_font_sizes, etc.)
- If not found: use defaults (4px grid, AA, 6 sizes, no hardcoded colors)

### Step 2: Read project context
- Check for tailwind.config.ts or tailwind.config.js
  - If found: extract theme.extend.colors, spacing, fontSize, borderRadius
  - Also check for Tailwind v4 CSS-based config (@theme in CSS files)
- Check for globals.css / theme.ts / CSS custom properties
  - If found: extract --primary, --secondary, etc.
- If nothing found: use static defaults only

### Step 3: Scan existing patterns
- List files in src/components/ and src/app/ (or pages/)
- Note dominant patterns: most common border-radius, spacing values,
  color tokens, button styles
- This becomes the "drift baseline" — new code is compared against this

### Step 4: Check exclude list
- Default excludes: node_modules, dist, .next, build, public
- Config excludes: add from .reservelabs.yml

## Stage Execution
- Stage 1 → Read and follow: checklists/planning.md
- Stage 2 → Read and follow: checklists/implementation.md
  - Also reference: references/visual-rules.md, references/ux-patterns.md
  - Also reference: supported-stacks/react-nextjs-tailwind.md (if stack matches)
- Stage 3 → Read and follow: checklists/prerelease.md
  - Dispatch: agents/design-reviewer.md for full scan

## Output Conventions
- Always show file path and line number for each finding
- Group findings by confidence: HIGH first, then MEDIUM, then INFO
- For HIGH: strongly recommend fix, offer auto-fix if possible
- For MEDIUM: note the issue, ask "Intentional? [Y/n]"
- For INFO: mention briefly, no action needed unless developer wants
- End with a 1-line summary: "X issues found (Y critical, Z warning)"
```

---

## Chunk 3: Agent + Validation (Tasks 8-9)

### Task 8: Design Reviewer Agent

**Files:**
- Create: `agents/design-reviewer.md`

- [ ] **Step 1: Create agents/design-reviewer.md**

Structure:

```markdown
---
name: reservelabs-design-reviewer
description: Autonomous design review agent. Scans entire frontend codebase
  for design drift, component duplication, spacing entropy, color sprawl,
  typography inconsistency, a11y gaps, and dependency bloat.
---

# ReserveLabs Design Reviewer

You are an autonomous design review agent. You scan an entire frontend
codebase and produce a comprehensive design consistency report.

## When Called
- By Stage 3 (pre-release) of the main ReserveLabs skill
- Directly by developer: "run design review" or "full design scan"

## Execution Strategy

### Small Project (<30 frontend files)
Scan all files in one pass.

### Large Project (30+ frontend files)
Use chunk strategy:
1. First pass: List ALL frontend files (.tsx, .jsx, .css, .module.css)
2. Group into chunks of ~15 files by directory
3. Analyze each chunk against the 7 audit layers
4. Merge findings across chunks
5. Produce unified report

## 7 Audit Layers

### Layer 1: Spacing Consistency
- Collect all spacing values across all files (p-*, m-*, gap-*, space-*)
- Count unique values
- Identify the dominant grid (e.g., most values divisible by 4)
- Flag values that break the grid
- Flag arbitrary values: p-[13px], m-[7px]
- Confidence: HIGH for arbitrary, MEDIUM for grid-breaking

### Layer 2: Color Discipline
- Collect all color references (text-*, bg-*, border-*, fill-*, stroke-*)
- Identify hardcoded values (#xxx, rgb(), hsl())
- Compare against Tailwind theme colors (if config available)
- Count unique colors — flag if >15 unique non-gray colors
- Confidence: HIGH for hardcoded, MEDIUM for palette sprawl

### Layer 3: Typography Order
- Collect all text-* and font-* classes
- Count distinct font-size values (text-xs through text-9xl + custom)
- Count distinct font-weight values
- Count distinct font-family values
- Flag if sizes > max_font_sizes config (default 6)
- Flag if weights > 3
- Confidence: INFO (typography diversity might be intentional)

### Layer 4: State Coverage
- For each component that fetches data or renders lists:
  - Loading state: Skeleton, Spinner, or loading prop handled?
  - Empty state: "No items" or empty UI handled?
  - Error state: Error boundary, error message, or catch block?
- Confidence: HIGH for missing error state, MEDIUM for others

### Layer 5: A11y Baseline
- Images without alt text
- Buttons/links without accessible name (aria-label or text content)
- Form inputs without labels
- Missing heading hierarchy
- Missing focus-visible styles
- Color contrast issues (estimate based on bg/text class pairs)
- Confidence: HIGH for all a11y issues

### Layer 6: Component Deduplication
- Compare component files by:
  - Similar prop interfaces
  - Similar JSX structure
  - Similar class names / styling
- Flag pairs that look very similar with a note:
  "ComponentA and ComponentB appear to serve similar purposes — consider merging"
- Confidence: MEDIUM (might be intentional variants)
- DO NOT output percentage similarity — say "very similar" or "somewhat similar"

### Layer 7: Dependency Bloat
- Check package.json for redundant packages:
  - Multiple icon libraries (lucide, heroicons, phosphor, etc.)
  - Multiple animation libraries (framer-motion, gsap, react-spring, etc.)
  - Multiple UI component libraries
  - Multiple CSS-in-JS solutions
- Flag each redundancy group
- Confidence: HIGH for same-category duplicates

## Report Format

Present findings in this format:

[Report header with files scanned count and issue count]

Group by severity:
- CRITICAL: a11y issues, missing error states
- WARNING: drift, spacing, color, duplication, bloat
- INFO: typography, style variations that might be intentional

For each finding:
- Category tag: [SPACING], [COLOR], [TYPO], [STATE], [A11Y], [DEDUP], [BLOAT]
- File path and line number
- What was found
- Suggested fix (if applicable)
- "Auto-fix? [Y/n]" for deterministic fixes
- "Intentional? [Y/n]" for MEDIUM/INFO items

End with per-category qualitative assessment:
- good: No significant issues found
- needs-work: Several issues that should be addressed
- poor: Systematic problems across the codebase

## Redaction Rules (CRITICAL)
When scanning source code, NEVER copy these into the report:
- API keys, tokens, secrets, passwords, connection strings
- Email addresses, phone numbers, national ID numbers
- Customer names + internal data combinations
- Environment variable VALUES (paths are ok, values are not)

If detected:
- Report ONLY: file path + line number + `[REDACTED]` label
- Suggest fix: "Move to .env" or "Use environment variable"

## Important Notes
- You are LLM-based. Your assessments are qualitative, not calculated.
- Two runs may produce slightly different results. This is expected.
- When uncertain, ask rather than assert.
- Never block the developer. Everything is advisory.
- If a file is too large to analyze effectively, note it and suggest splitting.
```

- [ ] **Step 2: Commit**

```bash
git add agents/
git commit -m "feat: add design reviewer agent with 7 audit layers"
```

---

### Task 9: Cross-Reference Validation

**Files:**
- No new files — validation only

- [ ] **Step 1: Verify all internal references**

Check that every file reference across all markdown files resolves correctly:
- `skill.md` → references to checklists/, references/, agents/, supported-stacks/
- `checklists/*.md` → references to references/ files
- `agents/design-reviewer.md` → references to output format, confidence levels
- Verify confidence level terminology is consistent (HIGH/MEDIUM/INFO everywhere)
- Verify output format is consistent across skill.md, checklists, and agent

- [ ] **Step 2: Fix any broken references or inconsistencies**

- [ ] **Step 3: Commit if fixes were needed**

```bash
git add -A
git commit -m "fix: resolve cross-reference inconsistencies"
```

---

## Chunk 4: Documentation (Tasks 10-12)

### Task 10: README.md — Bilingual

**Files:**
- Create: `README.md`

- [ ] **Step 1: Create README.md**

Must include:
- Project title + tagline (EN + TR)
- "What is this?" section (EN + TR)
- Problem statement: "50 prompts later, your app is Frankenstein"
- Visual diagram of 3 stages
- Quick start (Claude Code)
- What it checks (table)
- Example output (qualitative format: good/needs-work/poor, NOT percentages)
- Configuration (.reservelabs.yml)
- Non-Goals section: NOT a linter, NOT CI/CD, NOT a design system, NOT a security scanner
- Known Limitations section: non-deterministic (LLM-based), no persistence between runs, qualitative not quantitative
- Security note: agent redacts secrets/PII from scan reports
- Differentiation from Codex-Sentinel: "Codex-Sentinel = security checkpoints, ReserveLabs = design quality checkpoints. Different domains, complementary tools."
- Roadmap (v1 / v2)
- Contributing (link to CONTRIBUTING.md)
- License

Key SEO terms to include naturally: "design drift", "UI consistency", "design lint",
"vibecoding quality", "AI-generated code quality", "component duplication", "design system enforcement"

Keep the tone direct and confident. Show the problem, show the solution, show how to use it.

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "feat: add bilingual README (TR + EN)"
```

---

### Task 11: INSTALL.md

**Files:**
- Create: `INSTALL.md`

- [ ] **Step 1: Create INSTALL.md**

Two sections:

**Claude Code (primary):**
```
1. Clone: git clone https://github.com/reservelab/reservelabs.git
2. Copy entire reservelabs directory into your project's .claude/skills/:
   cp -r reservelabs/ your-project/.claude/skills/reservelabs/
3. (Optional) Copy config to project root:
   cp reservelabs/.reservelabs.example.yml your-project/.reservelabs.yml
4. Start using: tell Claude "reservelabs check my component" or "reservelabs planning"
```

**IMPORTANT:** Directory names must be preserved as-is. skill.md references
`checklists/planning.md`, `agents/design-reviewer.md`, etc. — renaming directories
will break these internal references.

**OpenAI Codex (best-effort):**
```
Codex agent format may differ. Basic approach:
1. Clone repo
2. Adapt skill.md to Codex agent format
3. Place in .codex/ directory
Community contributions for Codex compatibility are welcome.
```

- [ ] **Step 2: Commit**

```bash
git add INSTALL.md
git commit -m "feat: add installation guide for Claude Code and Codex"
```

---

### Task 12: CONTRIBUTING.md

**Files:**
- Create: `CONTRIBUTING.md`

- [ ] **Step 1: Create CONTRIBUTING.md**

Include:
- How to add a new supported stack (create file in `supported-stacks/`)
- How to add new rules to references
- How to improve checklists
- How to report false positives
- PR guidelines
- Code of conduct reference (Contributor Covenant)

- [ ] **Step 2: Commit**

```bash
git add CONTRIBUTING.md
git commit -m "feat: add contributing guide"
```

---

## Chunk 5: Dogfooding + Launch (Tasks 13-14)

### Task 13: Dogfooding Test

**Files:**
- No new files — testing on existing projects

- [ ] **Step 1: Pick a test project**

Use one of existing existing projects (e.g., DimTech frontend, Cevreci web, or any React/Next.js project on Spark) as test target.

- [ ] **Step 2: Install ReserveLabs skill into test project**

Copy skill files into the test project's `.claude/skills/` directory.

- [ ] **Step 3: Run Stage 1 (Planning)**

Tell Claude: "reservelabs — I'm going to build a dashboard component"
Verify: Does it produce a meaningful gap analysis?

- [ ] **Step 4: Run Stage 2 (Implementation)**

Point to an existing component. Tell Claude: "reservelabs check src/components/SomeComponent.tsx"
Verify: Does it find real issues? Are confidence levels appropriate? Any false positives?

- [ ] **Step 5: Run Stage 3 (Pre-Release)**

Tell Claude: "reservelabs final review"
Verify: Does design-reviewer agent produce a useful report? Does chunk strategy work for larger projects?

- [ ] **Step 6: Fix any issues found during testing**

Update skill/checklist/reference files based on real-world feedback.

- [ ] **Step 7: Commit fixes**

```bash
git add -A
git commit -m "fix: improvements from dogfooding test"
```

---

### Task 14: GitHub Push + X Announcement

**Files:**
- No new files

- [ ] **Step 1: Create GitHub repo**

```bash
gh repo create reservelab/reservelabs --public --description "Design drift detector for AI-generated codebases. Stage-aware UIX quality checkpoints for vibecoding workflows."
```

- [ ] **Step 2: Push to GitHub**

```bash
git remote add origin https://github.com/reservelab/reservelabs.git
git branch -M main
git push -u origin main
```

- [ ] **Step 3: Add GitHub topics**

Via GitHub UI or API, add topics:
`design-drift`, `uix`, `vibecoding`, `claude-code`, `codex`, `design-lint`,
`ui-consistency`, `accessibility`, `tailwind`, `react`, `nextjs`

- [ ] **Step 4: Draft X announcement**

Bilingual post (TR + EN) similar to Codex-Sentinel announcement:
- Problem statement (Frankenstein after 50 prompts)
- What ReserveLabs does (3 stages, 7 audit layers)
- Key capabilities list
- GitHub link
- Call to action (star, contribute, feedback)

- [ ] **Step 5: Owner reviews and posts**

Draft goes to Owner for review and posting on X.
