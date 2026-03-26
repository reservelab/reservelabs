# AGENTS.md

## Overview

ReserveLabs is a **documentation-only Claude Code skill suite** — there is no executable code, no runtime, no build step, and no server to start. The entire repo consists of Markdown (`.md`) files and one YAML config example (`.reservelabs.example.yml`).

The "application" is used by copying the repo into a target project's `.claude/skills/reservelabs/` directory, where Claude Code reads the Markdown prompts as instructions.

## Cursor Cloud specific instructions

### Linting

- **Markdown lint:** `markdownlint '**/*.md' --ignore node_modules` (requires `markdownlint-cli` installed globally via npm).
- **YAML lint:** `python3 -m yamllint .reservelabs.example.yml`.
- There is no TypeScript, ESLint, or Prettier config — the repo has no executable code.

### Testing

- There is no automated test suite. Validation consists of:
  1. Markdown linting for style consistency.
  2. YAML validation for the example config.
  3. Verifying all internal file references in `skill.md` resolve to existing files (checklists, agents, references, supported-stacks directories).
- To test the skill end-to-end, install it into a real React/Next.js/Tailwind project per `INSTALL.md` and invoke it through Claude Code.

### Building / Running

- There is no build step. The skill is "deployed" by copying the directory: `cp -r reservelabs/ your-project/.claude/skills/reservelabs/`.
- The example config is optionally copied: `cp .reservelabs.example.yml your-project/.reservelabs.yml`.

### Key file structure

- `skill.md` — main entry point read by Claude Code
- `checklists/` — stage-specific checklists (planning, implementation, prerelease)
- `agents/design-reviewer.md` — autonomous full-scan agent prompt
- `references/` — rule references (visual-rules, ux-patterns, responsive)
- `supported-stacks/` — stack-specific detection and analysis rules
- `.reservelabs.example.yml` — example configuration file
