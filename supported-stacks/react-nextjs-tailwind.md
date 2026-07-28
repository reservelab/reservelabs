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
| Tailwind version | `tailwindcss` entry in `package.json` / lockfile | **Authoritative** — v3 vs v4 |
| Tailwind v4 (CSS) | `@import "tailwindcss"` in the entry CSS; `@theme`, `@utility`, `@source` | Tailwind CSS v4 |
| Tailwind v4 (build) | `@tailwindcss/postcss` or `@tailwindcss/vite` in `package.json` | Tailwind CSS v4 |
| Tailwind v3 (CSS) | `@tailwind base;` / `@tailwind components;` / `@tailwind utilities;` | Tailwind CSS v3 |
| Tailwind v3 (build) | `tailwindcss` listed directly in `postcss.config.*` plugins | Tailwind CSS v3 |
| JS config | `tailwind.config.js`, `tailwind.config.ts`, `tailwind.config.mjs` | **Not a version signal** — see below |
| React | `package.json` containing `"react"` or `"next"` in `dependencies` | React / Next.js |
| App Router | `app/` directory with `layout.tsx` or `page.tsx` | Next.js App Router |
| Pages Router | `pages/` directory with `_app.tsx` or `index.tsx` | Next.js Pages Router |

**Detection order:**
1. Check `package.json` → confirms React/Next.js
2. Check for `next.config.*` → confirms Next.js
3. Read the `tailwindcss` version from `package.json` (and the lockfile if the range
   is ambiguous) → this is the **primary** version evidence
4. If the version is unreadable, fall back to the CSS entry point:
   `@import "tailwindcss"` → v4; `@tailwind base/components/utilities` → v3
5. Check the build plugin: `@tailwindcss/postcss` / `@tailwindcss/vite` → v4;
   plain `tailwindcss` in `postcss.config.*` → v3

**Do not infer the version from `tailwind.config.*`.** A v4 project can still load
that file through `@config "../tailwind.config.js"` — a *hybrid v4* setup. When both
exist, CSS `@theme` definitions win over the JS config on conflict. Treat a project as
v3 only when the version evidence in step 3–5 says so.

**Do not treat `postcss.config.*` alone as proof of Tailwind** — it is used by many
other tools. Likewise, a `@theme` block in a CSS file that is never reached from the
entry point (not imported anywhere) is a dead theme file, not the active config.

**Monorepos:** resolve the version per workspace, not per repository. A genuine v3
package and a v4 package can coexist in one repo. A single build pipeline, however,
runs exactly one major.

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
@import "tailwindcss";

@theme {
  --color-primary: oklch(0.72 0.11 178);
  --color-secondary: #64748b;
  --spacing-18: 4.5rem;
  --text-2xs: 0.625rem;
  --radius-4xl: 2rem;
}
```

**The source of truth is a graph, not a file.** In v3 you read one JS object.
In v4 you must start from the CSS entry point (the file containing
`@import "tailwindcss"`) and follow its `@import` chain, collecting **every**
`@theme` block along the way. Resolve them in cascade order: a later block
overrides an earlier one for the same variable. A `@theme` block in a CSS file
that nothing imports is dead code — ignore it, and optionally report it.

**Theme namespaces in v4** — each one generates a family of utilities:

| Namespace | Generates |
|-----------|-----------|
| `--color-*` | `bg-*`, `text-*`, `border-*`, … |
| `--font-*` | font family — `font-sans` |
| `--text-*` | **font size** — `text-xl` |
| `--font-weight-*` | `font-bold` |
| `--tracking-*` | `tracking-wide` |
| `--leading-*` | `leading-tight` |
| `--spacing` / `--spacing-*` | `p-4`, `w-16`, `gap-2`, … |
| `--breakpoint-*` | `sm:`, `md:` variants |
| `--container-*` | `max-w-md`, `@sm:` container queries |
| `--radius-*` | `rounded-*` |
| `--shadow-*`, `--inset-shadow-*`, `--drop-shadow-*` | shadow utilities |
| `--blur-*`, `--perspective-*`, `--aspect-*` | filter / 3D / ratio utilities |
| `--ease-*` | `ease-out` |
| `--animate-*` | `animate-spin` |

**There is no `--font-size-*` namespace in v4.** Font sizes live under `--text-*`.
A project writing `--font-size-lg` inside `@theme` has a dead variable that
generates no utility — flag it (see `STACK-TW-003`).

**Namespace resets:** `--color-*: initial` removes every default color, and
`--*: initial` clears the entire default theme. If a reset is present, the default
Tailwind palette/scale is NOT available — treat only the explicitly redefined
tokens as valid, exactly like `theme.colors` (without `.extend`) in v3.

**`@theme inline`** does not disable utility generation. It changes how the value is
resolved: the generated utility carries the *resolved* value instead of a `var()`
reference. Use it as the bridge signal — `@theme inline { --color-primary: var(--primary) }`
is what makes a runtime `:root` variable reachable as `bg-primary`.
`@theme static` emits all variables even when unused.

**Scan scope:** v4 has no `content` array. Files are found by automatic source
detection (the project root, minus `.gitignore`d paths), adjusted by
`@import "tailwindcss" source("…")`, extra `@source "…"` rules, and
`source(none)` which disables automatic detection entirely. Match ReserveLabs'
scan scope to that, not to a `content` glob.

### CSS Custom Properties (Both Versions)

Regardless of Tailwind version, scan `globals.css` or root CSS files for
CSS custom properties used as design tokens:

```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --primary: 222.2 47.4% 11.2%;
  --radius: 0.5rem;
}
```

Sort every custom property you find into one of three buckets — the bucket
decides whether it is a token, a bug, or noise:

| Bucket | Where it lives | Meaning |
|--------|---------------|---------|
| **Registered token** | inside `@theme` / `@theme inline` (v4), or the JS config (v3) | Real design token. Generates utilities. Palette-compliant. |
| **Runtime token** | `:root`, `.dark`, `[data-theme="…"]` — and bridged into `@theme inline` | Real design token, themeable at runtime. Palette-compliant. |
| **Rogue variable** | `:root` (or a component file) with no bridge into `@theme` | Looks like a token, generates nothing. Usual cause of drift. |

**In v4, `:root { --primary: … }` alone does NOT produce `bg-primary`.** It needs
`@theme inline { --color-primary: var(--primary) }`. Projects migrated from v3 by
hand often keep the `:root` block and lose the bridge — the class silently stops
existing. This is `STACK-TW-004`, and it is one of the highest-signal v4 findings.

Colors referenced via `hsl(var(--primary))` (v3-era shadcn) or `bg-primary`
(bridged v4 token) are palette-compliant and must NOT be flagged.

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
- `tailwindcss-animate` plugin in Tailwind config (v3) or `tw-animate-css` imported
  in CSS (v4) — same role, different era
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

### Dead Theme Variable (Tailwind v4)

**Rule ID:** `STACK-TW-003`
**Check:** A variable inside `@theme` that does not match any v4 theme namespace.
**Why:** v4 generates utilities from namespaces. `--font-size-lg`, `--colors-brand`,
or `--spacing18` (missing dash) produce no class at all. The developer — or the AI —
writes the token, uses `text-lg`/`bg-brand` in JSX, and gets the default value or
nothing. Silent failure.
**Confidence:** HIGH — this is mechanically verifiable against the namespace table.
**Applies to:** v4 only.

**Flag when:**
- A `@theme` variable's prefix is not one of the namespaces listed above
- Most common: `--font-size-*` (should be `--text-*`), plural forms (`--colors-*`),
  and missing separator dashes

**Do NOT flag when:**
- The variable is inside `@theme inline` purely as a bridge target — it still has to
  match a namespace, but the *value* being a `var()` is normal
- The file is a plain `:root` block (that is a runtime token or rogue variable —
  a different rule, `STACK-TW-004`)

**Suggested fix:** `Rename to the correct v4 namespace (e.g. --font-size-2xs → --text-2xs)`

### Broken Theme Bridge (Tailwind v4)

**Rule ID:** `STACK-TW-004`
**Check:** A `:root` variable that looks like a design token but is never bridged
into `@theme`, while utility classes referencing it appear in components.
**Why:** The most common failure of a hand-done v3 → v4 migration. `--primary` stays
in `:root`, `@theme inline { --color-primary: var(--primary) }` is never written,
and every `bg-primary` in the codebase silently resolves to nothing.
**Confidence:** HIGH when the class is actually used; MEDIUM when the variable is
merely orphaned.
**Applies to:** v4 only.

**Flag when:**
- `bg-<name>` / `text-<name>` / `border-<name>` appears in components AND
  `--color-<name>` is absent from every reachable `@theme` block
- A `:root` token set (shadcn-style: `--primary`, `--muted`, `--destructive`, …)
  exists with no corresponding `@theme inline` bridge

**Do NOT flag when:**
- The variable is consumed only through `var(--primary)` in hand-written CSS —
  no utility is expected
- The name collides with a default Tailwind color (`--primary` vs `bg-blue-500`)

**Suggested fix:** `Bridge the runtime variable into the theme: @theme inline { --color-primary: var(--primary) }`

### Split Source of Truth

**Rule ID:** `STACK-TW-005`
**Check:** The same design role defined in two places with two different values.
**Why:** After a migration or a long AI session, `--primary` (in `:root`),
`--color-primary` (in `@theme`), and `theme.extend.colors.primary` (in a still-loaded
`tailwind.config.js`) can all exist and disagree. Whichever wins, the other two are
lies that future prompts will read and propagate.
**Confidence:** HIGH — differing values for the same role is unambiguous.

**Flag when:**
- A hybrid v4 setup (`@config`) has a token defined in both CSS and the JS config
  with different values
- The same token name is redefined across two `@theme` blocks in the import chain
  with different values
- A `var()` alias chain is circular or points at an undefined variable

**Do NOT flag when:**
- The redefinition is a deliberate theme override (`.dark { --primary: … }`) —
  same role, different context, by design
- The values are equal after normalization (`#3b82f6` vs `oklch(…)` of the same color)

### Color Format Drift

**Rule ID:** `STACK-TW-006`
**Check:** One palette family expressed in mixed color formats.
**Why:** v4 ships an OKLCH default palette, shadcn v4 emits OKLCH, and AI-generated
code keeps writing hex. `--color-brand: #3b82f6` next to `--color-brand-dark: oklch(…)`
makes the palette impossible to reason about — and impossible to interpolate cleanly.
**Confidence:** INFO — this is a consistency finding, not a bug.

**Flag when:**
- Tokens in the same namespace family use 3+ different notations
  (hex / rgb / hsl / oklch)

**Do NOT flag when:**
- A single format is used consistently, whichever it is
- Only the format differs while the resolved color is identical — **normalize colors
  before comparing.** Never report "different color" on a format difference alone.

### Custom Utility Duplication (Tailwind v4)

**Rule ID:** `STACK-TW-007`
**Check:** The same abstraction expressed as `@utility`, as an `@apply` class, and as
hand-written CSS — or an `@utility` body containing literal values instead of tokens.
**Why:** v4 added `@utility` for first-class custom utilities. A drifting codebase
accumulates all three forms of the same idea, and the newest AI-written one hardcodes
`#3b82f6` instead of `var(--color-primary)`.
**Confidence:** MEDIUM.
**Applies to:** v4 only.

**Flag when:**
- Two or more of `@utility`, an `@apply` rule, and raw CSS define visually equivalent
  output under different names
- An `@utility` body uses literal colors/spacing that exist as theme tokens

**Do NOT flag when:**
- `@utility` and `@apply` merely coexist — they do different jobs. `@utility` *defines*
  a new utility; `@apply` *consumes* existing ones. Coexistence is not drift.

### Arbitrary Value Escape

**Rule ID:** `STACK-TW-008`
**Check:** Arbitrary values that bypass an existing token, especially when repeated.
**Why:** `p-[17px]` in one file is a rounding accident. `bg-[#3b82f6]` in six files
when `--color-primary` holds that exact value is the token system being routed around.
**Confidence:** MEDIUM when repeated 3+ times; INFO for a single occurrence.

**Flag when:**
- An arbitrary value equals — or is within ~1px / one step of — an existing theme token
- The same arbitrary value appears in 3+ files
- v4's CSS-variable shorthand `bg-(--foo)` points at a rogue variable
  (see `STACK-TW-004`)

**Do NOT flag when:**
- The value is genuinely one-off and computed: `w-[calc(100%-2rem)]`,
  `grid-cols-[1fr_2fr_1fr]`, `max-h-[calc(100vh-4rem)]`
- The project is v4 and the class is a **dynamic spacing utility** — see below

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

### Tailwind v4 Patterns
- **Off-scale numeric utilities:** `p-17`, `w-29`, `mt-13` are **valid in v4**. The
  spacing scale is derived dynamically — `p-<n>` compiles to
  `padding: calc(var(--spacing) * <n>)`. Do NOT flag these as off-grid against a fixed
  token list the way you would in v3. Only flag them if the project's rhythm is a clear
  multiple (4/8) and the value breaks it.
- **`@utility` alongside `@apply`:** Different jobs. `@utility` defines a new utility,
  `@apply` consumes existing ones. Both in one stylesheet is normal.
- **`@config "../tailwind.config.js"`:** A hybrid v4 setup, not a failed migration.
  JS configs still work in v4, they are just no longer auto-detected. Note that
  `corePlugins`, `safelist`, and `separator` are dropped in v4 — a config still
  carrying them has genuinely dead options (safelisting moved to `@source inline(…)`).
- **OKLCH color values:** `oklch(0.72 0.11 178)` is the v4 default palette format and
  what shadcn/ui emits. It is not an exotic hand-written value.
- **`@source` / `source(…)` rules:** These replace v3's `content` array. Their presence
  is configuration, not clutter.

### shadcn/ui on Tailwind v4
- **`tw-animate-css` instead of `tailwindcss-animate`:** shadcn deprecated the old
  plugin. Seeing `tw-animate-css` imported in CSS — with no plugin entry in any JS
  config — is correct for v4, not a missing dependency.
- **`:root` + `.dark` blocks outside `@layer base`, bridged by `@theme inline`:** This
  is the documented shadcn v4 layout. The `:root` variables are runtime tokens and the
  `@theme inline` block is the bridge — do NOT report the pair as a split source of
  truth (`STACK-TW-005`).
- **OKLCH values where older shadcn used `hsl(var(--x))`:** Expected. shadcn converted
  its palette to OKLCH. Both forms are palette-compliant; only mixing three or more
  notations in one family is a finding (`STACK-TW-006`).

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
- **tailwindcss-animate plugin (v3) / tw-animate-css (v4):** Required by shadcn/ui
  and magicui. Its presence alongside framer-motion is NOT redundant. Only flag when
  BOTH are installed at once — that is a leftover from an incomplete v4 migration.
