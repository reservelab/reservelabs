# Visual Rules Reference

Rules for spacing, color, and typography consistency.

## Spacing Rules

### VS-001: Spacing Grid Adherence
- **Check:** All spacing values (p-*, m-*, gap-*, space-*) divisible by base grid (default 4px)
- **Why:** Consistent spacing creates visual rhythm
- **Good:** `p-4` (16px), `gap-2` (8px), `mt-6` (24px)
- **Bad:** `p-[13px]`, `gap-[7px]`, `mt-[15px]`
- **Confidence:** HIGH
- **Auto-fix:** Yes — suggest nearest grid value

### VS-002: Arbitrary Spacing Values
- **Check:** Tailwind arbitrary values in spacing: p-[Npx], m-[Npx]
- **Why:** Arbitrary values bypass design system, signal AI copy-paste
- **Good:** `p-4`, `mx-auto`, `gap-8`
- **Bad:** `p-[13px]`, `mt-[47px]`, `gap-[9px]`
- **Confidence:** HIGH
- **Auto-fix:** Yes — suggest nearest Tailwind class

## Color Rules

### VS-003: Hardcoded Hex Colors
- **Check:** Inline hex (#xxx), rgb(), hsl() in className or style
- **Why:** Hardcoded colors break theme consistency and dark mode
- **Good:** `text-primary`, `bg-slate-900`, `border-zinc-200`
- **Bad:** `text-[#ff0000]`, `bg-[rgb(51,51,51)]`, `style={{color: '#333'}}`
- **Confidence:** HIGH
- **Auto-fix:** Yes — suggest nearest theme color

### VS-004: Color Outside Theme Palette
- **Check:** Color classes not in the project palette — `theme.extend.colors`
  (Tailwind v3) or the `--color-*` variables in the reachable `@theme` blocks
  (Tailwind v4)
- **Why:** Off-palette colors create visual inconsistency
- **Good:** Colors from project theme
- **Bad:** Random Tailwind colors not in theme
- **Confidence:** MEDIUM
- **Auto-fix:** No — needs design decision
- **Normalize before comparing:** v4 ships an OKLCH palette while AI-written code
  keeps emitting hex. Convert both sides to one color space first — never report
  "off-palette" on a notation difference alone (see STACK-TW-006)

## Typography Rules

### VS-005: Too Many Font Sizes
- **Check:** Count distinct text-* sizes across project. Flag if > max_font_sizes (default 6)
- **Why:** Typography chaos — AI generates random sizes per prompt
- **Good:** Consistent scale: text-sm, text-base, text-lg, text-xl, text-2xl, text-3xl
- **Bad:** 12 different sizes scattered across components
- **Confidence:** INFO
- **Auto-fix:** No — needs design decision

### VS-006: Too Many Font Weights
- **Check:** Count distinct font-* weights. Flag if > 3
- **Why:** Too many weights reduce hierarchy clarity
- **Good:** font-normal, font-medium, font-bold
- **Bad:** font-thin + font-light + font-normal + font-medium + font-semibold + font-bold
- **Confidence:** INFO
- **Auto-fix:** No — needs design decision

### VS-007: Inconsistent Border Radius
- **Check:** Compare border radius across similar components
- **Why:** Drift — each AI prompt generates different radius
- **Good:** All cards use rounded-xl, all buttons use rounded-lg
- **Bad:** Card A: rounded-lg, Card B: rounded-xl, Card C: rounded-2xl
- **Confidence:** MEDIUM
- **Auto-fix:** Yes — suggest majority pattern
