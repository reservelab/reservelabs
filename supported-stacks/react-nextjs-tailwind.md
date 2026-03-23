# Supported Stack: React / Next.js / Tailwind CSS

This file tells ReserveLabs how to detect, read, and analyze projects
built with React, Next.js, and Tailwind CSS. It also documents common
component library patterns so the skill avoids false positives.

---

## Stack Detection

Identify this stack by checking for the following files in the project root
(or nearby directories). Any combination confirms the stack:

| Signal | File Pattern | Confirms |
|--------|-------------|----------|
| Next.js | `next.config.js`, `next.config.mjs`, `next.config.ts` | Next.js framework |
| Tailwind v3 | `tailwind.config.js`, `tailwind.config.ts`, `tailwind.config.mjs` | Tailwind CSS v3 |
| Tailwind v4 | `@theme` directive inside any `.css` file (typically `app/globals.css`) | Tailwind CSS v4 |
| PostCSS | `postcss.config.js`, `postcss.config.mjs`, `postcss.config.cjs` | Tailwind (via PostCSS) |
| React | `package.json` containing `"react"` or `"next"` in `dependencies` | React / Next.js |
| App Router | `app/` directory with `layout.tsx` or `page.tsx` | Next.js App Router |
| Pages Router | `pages/` directory with `_app.tsx` or `index.tsx` | Next.js Pages Router |

**Detection order:**
1. Check `package.json` → confirms React/Next.js
2. Check for `next.config.*` → confirms Next.js
3. Check for `tailwind.config.*` → confirms Tailwind v3
4. If no Tailwind config file: search CSS files for `@theme` → confirms Tailwind v4
5. Check for `postcss.config.*` → secondary Tailwind confirmation

If only React is detected (no Next.js), the stack still applies — skip
Next.js-specific checks (Image component, directives, next/font).

---

## Reading Tailwind Config

### Tailwind v3 (tailwind.config.js / .ts / .mjs)

Read the config file and extract these theme tokens:

| Config Path | What It Provides | How ReserveLabs Uses It |
|-------------|-----------------|------------------------|
| `theme.extend.colors` | Project color palette (e.g., `primary`, `accent`, `brand`) | Enforce palette compliance — flag hex/rgb not in this palette |
| `theme.extend.spacing` | Custom spacing tokens (e.g., `18: '4.5rem'`) | Add to valid spacing grid — don't flag these as off-grid |
| `theme.extend.fontSize` | Custom type scale (e.g., `'2xs': '0.625rem'`) | Include in valid font size count — adjust max_font_sizes threshold |
| `theme.extend.borderRadius` | Custom radii (e.g., `'4xl': '2rem'`) | Include in drift comparison — these are valid radius values |
| `theme.extend.fontFamily` | Custom font stacks | Include in valid font family count |
| `theme.extend.screens` | Custom breakpoints | Use for responsive checks instead of defaults |
| `theme.extend.animation` | Custom animations | Don't flag these keyframe classes as unknown |
| `theme.extend.keyframes` | Custom keyframe definitions | Companion to animation — validates animation class usage |
| `content` | File glob patterns | Tells us which files Tailwind scans — match our scan scope |

Also check `theme` (without `.extend`) — this **replaces** defaults entirely
rather than extending them. If `theme.colors` exists (not `theme.extend.colors`),
the default Tailwind palette is NOT available.

### Tailwind v4 (CSS-based config)

Tailwind v4 moves configuration into CSS using the `@theme` directive.
Look for this in `globals.css`, `app.css`, or any root CSS file:

```css
@theme {
  --color-primary: #3b82f6;
  --color-secondary: #64748b;
  --spacing-18: 4.5rem;
  --font-size-2xs: 0.625rem;
  --radius-4xl: 2rem;
}
```

**Extraction rules for v4:**
- `--color-*` → project color palette
- `--spacing-*` → custom spacing tokens
- `--font-size-*` → custom type scale
- `--radius-*` → custom border radii
- `--font-*` (non-size) → custom font families
- `--breakpoint-*` → custom breakpoints
- `--animate-*` → custom animations

Also check for `@theme inline` (values available in CSS only, not as
utility classes) — these are still valid tokens for ReserveLabs purposes.

### CSS Custom Properties (Both Versions)

Regardless of Tailwind version, scan `globals.css` or root CSS files for
CSS custom properties used as design tokens:

```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --primary: 222.2 47.4% 11.2%;
  --secondary: 210 40% 96.1%;
  --muted: 210 40% 96.1%;
  --accent: 210 40% 96.1%;
  --destructive: 0 84.2% 60.2%;
  --border: 214.3 31.8% 91.4%;
  --ring: 222.2 84% 4.9%;
  --radius: 0.5rem;
}
```

These are valid design tokens — especially common with shadcn/ui.
Colors referenced via `hsl(var(--primary))` or `bg-primary` (Tailwind v4)
are palette-compliant and must NOT be flagged.

---

## Component Libraries

Each library below has distinct patterns in code. ReserveLabs must
recognize these patterns and NOT flag them as drift, duplication, or
inconsistency unless they genuinely conflict with the project's baseline.

### shadcn/ui

**Detection:** `components/ui/` directory + `lib/utils.ts` containing `cn()` function.
Also: `@radix-ui/*` packages in `package.json`.

**Patterns — do NOT flag:**
- `cn()` utility for conditional className merging (uses `clsx` + `tailwind-merge`)
- CSS custom properties for theming: `--primary`, `--secondary`, `--muted`, `--accent`, `--destructive`, `--border`, `--ring`, `--radius`
- HSL color format: `hsl(var(--primary))` — this is NOT a hardcoded color
- Radix UI primitives as base: `@radix-ui/react-dialog`, `@radix-ui/react-popover`, etc.
- `data-[state=open]` and `data-[state=closed]` attribute selectors
- `forwardRef` pattern on all components
- Variant patterns using `class-variance-authority` (cva): `variants`, `defaultVariants`
- `components.json` config file at project root
- Component files in `components/ui/` that look similar to each other (they are meant to be independent primitives)

**Intentional duplication:** shadcn components are copy-pasted into the project by design.
Two projects may have identical `Button.tsx` files — this is NOT component duplication.
Only flag duplication when the developer creates a SECOND custom button outside `components/ui/`.

### aceternity (Aceternity UI)

**Detection:** Framer Motion heavy usage + gradient/glow effects + `"aceternity"` or
`@aceternity` in package.json or import paths.

**Patterns — do NOT flag:**
- Heavy `framer-motion` usage: `motion.div`, `AnimatePresence`, `useMotionValue`, `useTransform`
- Complex multi-line className strings (these components have many conditional styles)
- Gradient backgrounds: `bg-gradient-to-*`, linear-gradient in style prop
- Glow/blur effects: `blur-xl`, `blur-3xl`, filter properties
- `useRef` + `useEffect` for scroll/mouse tracking animations
- Inline styles for dynamic animation values (`style={{ transform, opacity }}`)
- Canvas elements for particle/beam effects
- SVG animation with motion components
- Large className strings (50+ characters) — this is normal for aceternity components

**Do NOT flag as dependency bloat:** `framer-motion` is required — it is not redundant
even if other animation approaches exist in the project.

### magicui (Magic UI)

**Detection:** Tailwind animation classes + custom keyframes + `"magicui"` or
`@magicui` in imports/package.json.

**Patterns — do NOT flag:**
- Custom Tailwind animation utility classes: `animate-*` with project-specific keyframes
- Multiple `@keyframes` definitions in CSS (magicui is keyframe-heavy)
- `animation-delay` and `animation-duration` inline styles
- Complex `animate-[custom]` arbitrary value syntax in className
- Intersection Observer usage for scroll-triggered animations
- `tailwindcss-animate` plugin in Tailwind config
- Staggered animation patterns (multiple similar elements with offset delays)

### originui (Origin UI)

**Detection:** Tailwind-based components similar to shadcn patterns. Check imports
from `@/components/ui/` or `originui` package references.

**Patterns — do NOT flag:**
- shadcn-similar patterns (cn utility, CSS variables, Radix primitives)
- Slightly different variant implementations compared to shadcn — these are
  intentional design choices, not drift
- Both shadcn AND originui components in same project (developer may use both)
- Component files that look similar to shadcn but have different default styles

### preline (Preline UI)

**Detection:** `data-hs-*` attributes on elements + `preline` in package.json
or script imports.

**Patterns — do NOT flag:**
- `data-hs-overlay`, `data-hs-collapse`, `data-hs-dropdown`, `data-hs-tab`,
  `data-hs-accordion`, `data-hs-tooltip` attributes
- Preline's JavaScript plugin initialization (`HSStaticMethods.autoInit()`)
- Tailwind utility-first approach with verbose className strings
- `hs-*` CSS classes (preline's custom classes)
- Plugin in `tailwind.config.js`: `require('preline/plugin')`
- Inline event handlers for component initialization

**Important:** Preline uses its own JS for interactivity (not Radix). Having both
Preline JS and Radix JS is NOT dependency bloat — they serve different components.

### hyperui (HyperUI)

**Detection:** Pure Tailwind components with NO JS dependencies. No special
package in package.json — hyperui components are copy-pasted HTML+Tailwind.

**Patterns — do NOT flag:**
- Very long className strings (hyperui components are verbose by design)
- No React state management in component (pure presentational)
- Repeated Tailwind utility patterns across components (copy-paste is the distribution model)
- Group hover/focus patterns: `group`, `group-hover:*`, `group-focus:*`
- Peer patterns: `peer`, `peer-checked:*`, `peer-focus:*`
- Complex responsive utility chains: `sm:* md:* lg:* xl:*` on single elements
- No `cn()` or className merging — classes are static strings

**Important:** HyperUI components look like "copy-paste from AI" because they ARE
copy-paste components. Do not flag className duplication across hyperui components
unless the developer has clearly duplicated a full component (same structure, same
purpose, different file name).

---

## Stack-Specific Checks

These checks apply specifically to React/Next.js/Tailwind projects.

### Next.js Image Component

**Rule ID:** `STACK-NJS-001`
**Check:** Raw `<img>` tags should use Next.js `<Image>` component instead.
**Why:** Next.js Image provides automatic optimization, lazy loading, and responsive sizing.
**Confidence:** MEDIUM — some cases legitimately need `<img>` (SVG inline, external
images from untrusted domains, email templates).

**Flag when:**
- `<img src=` appears in `.tsx`/`.jsx` files inside `app/` or `pages/` directories
- Image source is a local file (`/images/`, `./assets/`, import from file)

**Do NOT flag when:**
- Image is inside an email template or generated HTML string
- Image is an inline SVG (`<img>` wrapping SVG data URI)
- Component is explicitly marked `"use client"` and uses dynamic src from user input
- Image is inside `public/` referenced by absolute path AND there is a documented
  reason (check for nearby comment)

**Suggested fix:** `Replace <img> with next/image Image component for automatic optimization`

### Client/Server Directives

**Rule ID:** `STACK-NJS-002`
**Check:** Presence and correctness of `"use client"` / `"use server"` directives.
**Why:** App Router requires explicit boundaries between server and client components.
**Confidence:** INFO — directive strategy is an architectural decision.

**Flag when:**
- A component uses hooks (`useState`, `useEffect`, `useRef`, `useContext`) but has
  no `"use client"` directive → will fail at runtime
- A component has `"use client"` but uses NO client-side features (hooks, event handlers,
  browser APIs) → unnecessary client bundle size
- `"use server"` appears on a function that does not perform server-side operations

**Do NOT flag when:**
- `"use client"` is at the top of a file that imports from a client-only library
  (framer-motion, aceternity components, etc.)
- The file is inside a directory that has a parent `"use client"` boundary
- Pages Router project (directives don't apply)

### Tailwind @apply Overuse

**Rule ID:** `STACK-TW-001`
**Check:** Excessive use of `@apply` directive in CSS files.
**Why:** `@apply` defeats Tailwind's utility-first approach. Heavy use indicates
the developer is fighting the framework — classes should be in JSX, not extracted
to CSS unless creating a true design token.
**Confidence:** MEDIUM for >5 per file, INFO for 3-5 per file.

**Threshold:**
- 0-2 `@apply` rules per file → acceptable (base layer, resets)
- 3-5 per file → INFO: "Consider using utility classes directly in components"
- >5 per file → WARNING: "Heavy @apply usage — this file is fighting Tailwind's
  utility-first model. Consider moving styles to component className props."

**Do NOT flag when:**
- `@apply` is in a `base` layer (`@layer base { }`) for global resets
- `@apply` is creating animation keyframe utilities
- The file is a dedicated design token stylesheet (e.g., `tokens.css`, `typography.css`)

### className Duplication Across Components

**Rule ID:** `STACK-TW-002`
**Check:** Identical or near-identical long className strings appearing in multiple
component files.
**Why:** This is the #1 indicator of copy-paste from AI. The developer asked AI
to create similar components separately instead of extracting a shared component
or using a shared className constant.
**Confidence:** MEDIUM — some duplication is acceptable (simple utility combos like
`flex items-center justify-between`).

**Flag when:**
- A className string of 8+ utilities appears identically in 3+ different files
- Two component files have >70% similar className patterns on their root element
  and immediate children (assessed qualitatively — "very similar" not a percentage)

**Do NOT flag when:**
- Short utility combos (<5 classes): `flex items-center gap-2` appearing everywhere is fine
- Components from different component libraries (shadcn Button vs aceternity Button)
- Layout utilities that naturally repeat: `container mx-auto px-4`
- Components in `components/ui/` directory (these are primitives, repetition is expected)

### next/font Usage

**Rule ID:** `STACK-NJS-003`
**Check:** Font loading strategy — `next/font` vs manual font imports.
**Why:** `next/font` provides automatic optimization, zero layout shift, and
built-in privacy (self-hosted, no external requests to Google Fonts).
**Confidence:** INFO — manual font loading works fine, next/font is a best practice.

**Flag when:**
- `<link>` tags loading Google Fonts in `layout.tsx` or `_document.tsx`
- `@import url('fonts.googleapis.com/...')` in CSS files
- Manual `@font-face` declarations for common Google Fonts that next/font supports

**Do NOT flag when:**
- Custom/proprietary fonts not available via next/font
- Font loaded from project's own `/public/fonts/` directory with `@font-face`
- The project uses Pages Router with a custom `_document.tsx` font strategy

**Suggested fix:** `Replace Google Fonts link with next/font/google for automatic optimization and zero layout shift`

---

## Intentional Patterns (Do NOT Flag)

These patterns look like drift, duplication, or bad practice but are
intentional and correct in React/Next.js/Tailwind projects:

### Tailwind Patterns
- **Long className strings:** Tailwind's utility-first approach naturally produces
  long class lists. A 15-class string is normal, not a problem.
- **Responsive prefix chains:** `text-sm md:text-base lg:text-lg xl:text-xl` on a
  single element is correct responsive design, not class bloat.
- **Dark mode variants:** `bg-white dark:bg-gray-900 text-black dark:text-white` doubles
  the class count — this is expected, not duplication.
- **State variants:** `hover:bg-blue-600 focus:ring-2 focus:ring-blue-500 active:bg-blue-700`
  adds multiple classes for one element — all intentional.
- **Arbitrary values:** `w-[calc(100%-2rem)]` or `grid-cols-[1fr_2fr_1fr]` are valid
  Tailwind syntax for values not in the default scale. Only flag if the value
  could be replaced by a standard token (e.g., `p-[16px]` should be `p-4`).
- **`!important` modifier:** `!text-red-500` — Tailwind's important modifier. May
  indicate specificity issues but is a valid pattern.

### React Patterns
- **Multiple similar components in `components/ui/`:** These are design system primitives.
  `Button`, `IconButton`, `LinkButton` are intentionally separate.
- **Wrapper components:** A thin wrapper around a library component (e.g., wrapping
  shadcn's Button with project-specific defaults) is good practice, not duplication.
- **Context providers stacked in layout.tsx:** Multiple `<Provider>` wrapping is
  standard React composition.
- **Re-exporting from barrel files:** `components/ui/index.ts` re-exporting all
  primitives is organizational, not bloat.

### Next.js Patterns
- **`loading.tsx` + `error.tsx` + `not-found.tsx` in same directory:** These are
  Next.js conventions for route-level states. Their presence is GOOD — flag their
  ABSENCE instead.
- **`layout.tsx` at multiple directory levels:** Nested layouts are a core Next.js
  App Router feature.
- **Server Actions in `"use server"` files:** Functions that look like they should
  be API routes but are in `.ts` files with the `"use server"` directive.
- **Parallel routes (`@folder`):** Directories starting with `@` are parallel route
  segments, not naming errors.
- **Intercepting routes (`(.)`, `(..)`):** Parenthesized folder names are route
  groups and intercepting routes.
- **`generateStaticParams` + `generateMetadata` exports:** These are Next.js
  conventions, not unused exports.

### Component Library Co-existence
- **shadcn + aceternity in same project:** Common pattern. shadcn for form/data
  components, aceternity for landing page effects. NOT library bloat.
- **Multiple Radix packages:** shadcn uses many `@radix-ui/*` packages. Each is
  a separate primitive — 10+ Radix packages is normal, not bloat.
- **framer-motion + CSS animations:** framer-motion for complex orchestrated
  animations, CSS/Tailwind animations for simple transitions. Both in same
  project is fine.
- **tailwindcss-animate plugin:** Required by shadcn/ui and magicui. Its presence
  alongside framer-motion is NOT redundant.
