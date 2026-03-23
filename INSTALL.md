# Installation

## Claude Code (Primary)

ReserveLabs is a Claude Code skill. Install it by copying the directory into your project:

```bash
# 1. Clone the repository
git clone https://github.com/reservelab/reservelabs.git

# 2. Copy the entire directory into your project's skills folder
cp -r reservelabs/ your-project/.claude/skills/reservelabs/

# 3. (Optional) Copy the example config to your project root and customize
cp reservelabs/.reservelabs.example.yml your-project/.reservelabs.yml

# 4. Start using it — tell Claude Code:
#    "reservelabs check my component"
#    "reservelabs planning"
```

> **Important:** Directory names and structure must be preserved as-is.
> `skill.md` references files by relative path — `checklists/planning.md`,
> `agents/design-reviewer.md`, `references/`, `supported-stacks/`, etc.
> Renaming or restructuring these directories will break internal references.

## OpenAI Codex (Best-Effort)

Codex agent format may differ from Claude Code skills. Basic approach:

```
1. Clone the repository
2. Adapt skill.md to Codex agent format
3. Place in your project's .codex/ directory
```

Community contributions for Codex compatibility are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.
