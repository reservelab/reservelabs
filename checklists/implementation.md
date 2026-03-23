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
- Headings: correct hierarchy (no skipping h1 to h3)?
- Confidence: HIGH (these are always issues)
- Auto-fix: suggest aria-label/alt text based on context
- Reference: WCAG 2.2 criteria (see references/ux-patterns.md)

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

## Redaction
If source code contains API keys, tokens, passwords, emails, or other PII:
- Do NOT copy them into findings
- Report only file:line + [REDACTED]
- Suggest: "Move to .env" or "Use environment variable"

## Output Format
Group by confidence level. Show file:line for each issue.
Offer auto-fix where available. Ask "intentional?" for MEDIUM/INFO items.
