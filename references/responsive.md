# Responsive Rules Reference

Breakpoint and responsive design patterns. v1 is advisory-only.

## Rules

### RS-001: Mobile-First Approach
- **Check:** Base styles target mobile, larger screens use sm:/md:/lg: prefixes
- **Why:** Mobile-first ensures all devices get a usable layout
- **Good:** `text-sm md:text-base lg:text-lg`
- **Bad:** `text-lg sm:text-sm` (desktop-first, mobile as afterthought)
- **Confidence:** INFO
- **Auto-fix:** No — architectural decision

### RS-002: Fixed Width Containers
- **Check:** Container/wrapper elements use responsive widths, not fixed px
- **Why:** Fixed widths break on different screen sizes
- **Good:** `max-w-7xl mx-auto`, `w-full md:w-1/2`
- **Bad:** `w-[1200px]`, `width: 960px`
- **Confidence:** MEDIUM
- **Auto-fix:** Yes — suggest max-w-* + mx-auto

### RS-003: Horizontal Scroll Prevention
- **Check:** No elements wider than viewport causing horizontal scroll
- **Why:** Horizontal scroll on mobile = broken layout
- **Good:** `overflow-x-hidden` on container, responsive widths
- **Bad:** Fixed-width tables, images without max-w-full
- **Confidence:** MEDIUM
- **Auto-fix:** Yes — suggest overflow-x-auto or max-w-full

### RS-004: Mobile Touch Targets
- **Check:** Clickable elements >= 44px on mobile viewports
- **Why:** Small touch targets cause user frustration
- **Good:** `min-h-11 p-3` on buttons/links
- **Bad:** Tiny icon buttons, cramped navigation links
- **Confidence:** HIGH
- **Auto-fix:** Yes — suggest min-h-11

### RS-005: Text Truncation
- **Check:** Long text handled with truncation or wrapping on small screens
- **Why:** Overflowing text breaks layouts on mobile
- **Good:** `truncate`, `line-clamp-2`, `break-words`
- **Bad:** Long titles/names overflowing container
- **Confidence:** INFO
- **Auto-fix:** Yes — suggest truncate or line-clamp

## Standard Breakpoints (Tailwind)
| Prefix | Min Width | Device |
|--------|-----------|--------|
| sm: | 640px | Large phone |
| md: | 768px | Tablet |
| lg: | 1024px | Laptop |
| xl: | 1280px | Desktop |
| 2xl: | 1536px | Large desktop |
