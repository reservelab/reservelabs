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
- [ ] Touch targets >=44px?

### Drift Prevention
- [ ] Similar existing components identified?
- [ ] Reuse vs. new component decision made?
- [ ] Consistent with existing patterns?

## Output Format
Present as checklist. For each unchecked item, explain why it matters
(1 sentence) and suggest a default if developer has no preference.
