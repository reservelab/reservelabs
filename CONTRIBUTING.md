# Contributing to ReserveLabs

Welcome! ReserveLabs is a design drift detector for AI-generated codebases, and contributions help it cover more stacks, catch more drift patterns, and reduce false positives. Whether you're adding a new stack, improving a checklist, or reporting a false positive — thank you.

## How to Contribute

### Add a New Supported Stack

1. Create a new file in `supported-stacks/` following the existing format. See `supported-stacks/react-nextjs-tailwind.md` as a reference.
2. Your file should include:
   - **Stack Detection** — file patterns and signals that identify the stack
   - **Reading Config** — how to extract theme tokens and design decisions
   - **Component Libraries** — common libraries for that stack, their patterns, and what NOT to flag
   - **Stack-Specific Checks** — rules unique to this stack (each with a Rule ID)
   - **Intentional Patterns** — things that look like drift but are correct for this stack
3. Open a PR with the new file.

### Add New Rules to References

Rules in `references/` use the following ID format:

```
XX-NNN
```

- `XX` — category prefix (e.g., `VR` for visual rules, `UX` for UX patterns, `STACK` for stack-specific)
- `NNN` — three-digit sequential number within that category

When adding a new rule:
1. Find the next available number in the relevant category.
2. Include: Rule ID, what to check, why it matters, confidence level (HIGH / MEDIUM / INFO), when to flag, when NOT to flag, and a suggested fix.
3. Follow the structure of existing rules.

### Improve Checklists

Checklists live in `checklists/` (planning, implementation, prerelease). To improve them:

1. Add new checks that catch real drift patterns you've encountered.
2. Assign an appropriate confidence level:
   - **HIGH** — almost certainly a problem, strong recommendation to fix
   - **MEDIUM** — likely a problem, ask the developer if intentional
   - **INFO** — worth mentioning, no action needed unless developer wants
3. Include "do NOT flag when" exceptions to avoid false positives.
4. If modifying an existing check, explain why in the PR description.

### Report False Positives

If ReserveLabs flagged something that was intentional in your project, open a GitHub issue with:

- **What was flagged** — the rule ID and finding message
- **Why it's intentional** — explain the design decision or pattern
- **What project/stack** — the framework, component libraries, and any relevant config

This helps us add exceptions and improve accuracy.

## PR Guidelines

- **One feature per PR.** Don't mix a new stack with checklist changes.
- **Describe what and why.** The title says what changed; the description explains the reasoning.
- **Test in a real project.** Run your changes against an actual codebase before submitting. Mention which project/stack you tested against.

## Code of Conduct

This project follows the [Contributor Covenant v2.1](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). Be respectful, constructive, and inclusive.
