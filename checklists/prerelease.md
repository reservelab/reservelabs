# Stage 3: Pre-Release Checkpoint

## Trigger
Developer says: "done", "ready for review", "pre-release check",
"final review", or before commit/PR.

## Execution Model
This stage invokes the design-reviewer agent for a full codebase scan.
If the project is small (<30 frontend files), run directly.
If large (30+), use chunk strategy (see agents/design-reviewer.md).

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
