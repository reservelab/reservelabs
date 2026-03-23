# UX Patterns Reference

Accessibility (WCAG 2.2 mapped) and state coverage rules.

## Accessibility Rules

### UX-001: Image Alt Text
- **WCAG:** 1.1.1 Non-text Content (Level A)
- **Check:** All <img> and Image components have meaningful alt text
- **Why:** Screen readers cannot describe images without alt text
- **Good:** `<Image alt="Dashboard showing monthly revenue chart" />`
- **Bad:** `<Image />`, `<img alt="" />`, `<img alt="image" />`
- **Confidence:** HIGH
- **Auto-fix:** Yes — suggest alt based on context

### UX-002: Button/Link Accessible Name
- **WCAG:** 4.1.2 Name, Role, Value (Level A)
- **Check:** Buttons and links have text content or aria-label
- **Why:** Screen readers announce empty buttons as "button"
- **Good:** `<button aria-label="Close dialog">X</button>`
- **Bad:** `<button><Icon /></button>` (no text, no aria-label)
- **Confidence:** HIGH
- **Auto-fix:** Yes — suggest aria-label

### UX-003: Form Input Labels
- **WCAG:** 1.3.1 Info and Relationships (Level A)
- **Check:** Every input/select/textarea has associated label
- **Why:** Users cannot identify form fields without labels
- **Good:** `<label htmlFor="email">Email</label><input id="email" />`
- **Bad:** `<input placeholder="Email" />` (placeholder is not a label)
- **Confidence:** HIGH
- **Auto-fix:** Yes — suggest label element

### UX-004: Heading Hierarchy
- **WCAG:** 1.3.1 Info and Relationships (Level A)
- **Check:** Headings follow correct order (h1 → h2 → h3, no skipping)
- **Why:** Screen readers use headings for navigation
- **Good:** h1 → h2 → h3
- **Bad:** h1 → h3 (skipped h2), multiple h1 on page
- **Confidence:** HIGH
- **Auto-fix:** No — needs structural decision

### UX-005: Focus Visible
- **WCAG:** 2.4.7 Focus Visible (Level AA)
- **Check:** Interactive elements have visible focus indicator
- **Why:** Keyboard users cannot see where they are without focus styles
- **Good:** `focus-visible:ring-2 focus-visible:ring-primary`
- **Bad:** `outline-none` without replacement, `focus:outline-none` only
- **Confidence:** HIGH
- **Auto-fix:** Yes — suggest focus-visible:ring-2

### UX-006: Color Contrast
- **WCAG:** 1.4.3 Contrast Minimum (Level AA)
- **Check:** Text/background color pairs meet 4.5:1 ratio (3:1 for large text)
- **Why:** Low contrast text is unreadable for many users
- **Good:** text-slate-900 on bg-white, text-white on bg-slate-900
- **Bad:** text-gray-400 on bg-gray-200, text-slate-300 on bg-white
- **Confidence:** MEDIUM (LLM estimates, not calculated)
- **Auto-fix:** No — suggest checking with contrast tool

### UX-007: Touch Target Size
- **WCAG:** 2.5.8 Target Size Minimum (Level AA)
- **Check:** Clickable elements at least 44x44px
- **Why:** Small targets cause misclicks on mobile
- **Good:** `min-h-11 min-w-11` (44px)
- **Bad:** `h-6 w-6` icon button (24px)
- **Confidence:** HIGH on mobile, MEDIUM on desktop
- **Auto-fix:** Yes — suggest min-h-11 min-w-11

## State Coverage Rules

### UX-008: Loading State Missing
- **Check:** Components with async data have loading UI
- **Why:** No loading state = blank screen during fetch
- **Good:** Skeleton, Spinner, or Suspense fallback
- **Bad:** Component renders nothing until data arrives
- **Confidence:** MEDIUM
- **Auto-fix:** Yes — offer Skeleton template

### UX-009: Empty State Missing
- **Check:** Lists/tables handle zero-item case
- **Why:** Empty list with no message looks broken
- **Good:** "No items found" message, illustration, or CTA
- **Bad:** Empty container, blank white space
- **Confidence:** MEDIUM
- **Auto-fix:** Yes — offer empty state template

### UX-010: Error State Missing
- **Check:** API calls have error handling UI
- **Why:** Unhandled error = white screen or cryptic message
- **Good:** Error boundary, retry button, user-friendly message
- **Bad:** No catch block, no error UI, console.error only
- **Confidence:** HIGH
- **Auto-fix:** Yes — offer ErrorBoundary template
