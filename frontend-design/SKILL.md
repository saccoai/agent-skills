---
name: frontend-design
description: Create distinctive, production-grade frontend interfaces with adaptive design direction. Guides aesthetic selection via a decision tree based on client industry/audience, enforces WCAG AA, mobile-first responsive, performance budgets, and dark mode. Uses shadcn/ui + Tailwind CSS v4. Every project is fully bespoke.
---

This skill creates production-grade frontend interfaces that are distinctive, accessible, and performant. Every project gets a bespoke aesthetic direction — no two designs should look alike.

The user provides: the client context (industry, audience, brand), the tech stack, and any design constraints.

## Non-Negotiable Quality Standards

Every project MUST meet these. No exceptions, no shortcuts:

### 1. Mobile-First Responsive
- Design at **390px first**, then scale up
- Test at 390px, 768px, 1280px+ breakpoints after implementing each component
- No horizontal scrolling at any breakpoint
- Touch targets ≥ 44×44px on mobile
- Stack layouts vertically on mobile, expand to grid on desktop

### 2. WCAG AA Accessibility
- **Color contrast**: 4.5:1 for normal text, 3:1 for large text (18px+ bold or 24px+)
- **Keyboard navigation**: every interactive element reachable via Tab, activatable via Enter/Space
- **Focus indicators**: visible focus ring on all interactive elements (never `outline: none` without replacement)
- **ARIA**: proper roles, labels, and descriptions. Headings in logical order (no skipped levels)
- **Alt text**: every `<img>` has meaningful alt text (or `alt=""` for decorative images)
- **Form labels**: every input has a visible `<label>`. Error messages linked via `aria-describedby`
- **Screen readers**: test with VoiceOver (macOS) — content must be logical when read linearly
- **Reduced motion**: respect `prefers-reduced-motion` — disable animations for users who opt out

### 3. Performance Budget
- No single component adds more than **50KB** of JavaScript
- All images: use `next/image` with proper `sizes` and `priority` attributes
- Images served in modern formats (WebP/AVIF) via Next.js automatic optimization
- Total page JS bundle: flag if exceeding **200KB gzipped**
- Animations: use `LazyMotion` + `m` from Framer Motion (tree-shakeable), never import `motion` directly
- Fonts: load via `next/font` with `display: swap`. Maximum 2 font families per project
- Lighthouse Performance score target: **≥ 90**

### 4. Dark Mode Support
- Every project ships with working dark mode
- Use CSS variables in Tailwind v4 `@theme` for all colors
- Use `dark:` variants in Tailwind for component-level overrides
- Toggle mechanism: respect `prefers-color-scheme` system preference, with optional manual toggle
- Test both modes visually — dark mode is not an afterthought

```css
/* Example: globals.css */
@import "tailwindcss";

@theme {
  --color-background: #f8f7f5;
  --color-foreground: #1c1917;
  --color-primary: #8e375c;
  --color-muted: #a8a29e;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-background: #1c1917;
    --color-foreground: #f5f5f4;
    --color-primary: #c76b94;
    --color-muted: #78716c;
  }
}
```

## Aesthetic Direction — Decision Tree

Before writing any code, run this decision tree to select the right aesthetic for the client. Ask these questions (use AskUserQuestion or discuss with the user):

### Q1: What is the client's industry?

| Industry | Lean toward |
|----------|------------|
| Finance, law, consulting | **Refined Professional** |
| Fashion, art, architecture | **Editorial Luxury** |
| Tech, SaaS, startup | **Clean Technical** |
| Food, hospitality, wellness | **Warm Organic** |
| Education, non-profit, government | **Trustworthy Minimal** |
| Creative agency, entertainment | **Bold Expressive** |

### Q2: Who is the primary audience?

| Audience | Adjust toward |
|----------|--------------|
| Corporate / B2B | More restraint, more whitespace, conventional layouts |
| Consumer / B2C | More personality, bolder colors, more engaging interactions |
| Elderly / accessibility-first | Larger text, higher contrast, simpler layouts, less motion |
| Young / digital-native | More dynamic, more motion, more experimental layouts |

### Q3: What is the brand personality?

| Personality | Design expression |
|-------------|------------------|
| Authoritative / trustworthy | Serif headings, muted palette, symmetric layouts |
| Innovative / forward-thinking | Geometric sans-serif, asymmetric, accent gradients |
| Warm / approachable | Rounded elements, warm colors, friendly typography |
| Premium / exclusive | High contrast, editorial spacing, restrained palette |
| Playful / energetic | Bold colors, large type, dynamic layouts, more animation |

### Result: Select one of these directions

#### 1. Refined Professional
- **Typography**: Serif display (e.g., Playfair Display, DM Serif Display, Lora) + clean sans body (e.g., Inter, DM Sans, Source Sans 3)
- **Palette**: Muted, warm neutrals. One sophisticated accent (navy, burgundy, forest green). No bright colors.
- **Layout**: Symmetric, generous whitespace, max-w-5xl content, clear hierarchy
- **Motion**: Subtle — fade-in reveals on scroll, gentle hover states
- **Spacing**: py-20 md:py-28 section padding, relaxed line-height (1.7+)

#### 2. Editorial Luxury
- **Typography**: Elegant serif (e.g., Cormorant Garamond, Libre Baskerville, Noto Serif Display) + refined sans (e.g., Outfit, Satoshi, General Sans)
- **Palette**: Near-monochrome with one rich accent. Black/cream base. Gold, plum, or deep teal accent.
- **Layout**: Magazine-inspired. Asymmetric grids, overlapping elements, dramatic hero images
- **Motion**: Moderate — parallax on heroes, staggered list reveals, text character animations
- **Spacing**: Very generous. Content floats in space. Negative space is a design element.

#### 3. Clean Technical
- **Typography**: Modern geometric sans (e.g., Plus Jakarta Sans, Geist, Manrope) for both display and body
- **Palette**: Cool neutrals (slate, zinc). One vibrant accent (blue, violet, emerald). Clean whites.
- **Layout**: Grid-based, card-heavy, data-friendly. Dashboard-like precision.
- **Motion**: Functional — hover lifts on cards, smooth page transitions, loading states
- **Spacing**: Compact but not cramped. Efficient use of space. gap-6 between cards.

#### 4. Warm Organic
- **Typography**: Rounded or humanist sans (e.g., Nunito, Quicksand, Rubik) or soft serif (e.g., Merriweather, Bitter)
- **Palette**: Earth tones — terracotta, sage, cream, warm brown. Soft pastels. No harsh contrasts.
- **Layout**: Rounded corners, soft shadows, flowing sections. Organic shapes (SVG blobs, curves).
- **Motion**: Gentle — slow fades, easing that feels natural, subtle breathing animations
- **Spacing**: Generous and flowing. Sections merge softly. Rounded containers.

#### 5. Trustworthy Minimal
- **Typography**: Neutral, highly legible sans (e.g., IBM Plex Sans, Atkinson Hyperlegible, Open Sans) for everything
- **Palette**: High-contrast. Dark text on white. One institutional accent (blue, teal). No gradients.
- **Layout**: Simple, predictable, content-first. Single-column where possible. Large text.
- **Motion**: Minimal — content appears immediately. Maybe a single page-load fade.
- **Spacing**: Generous but functional. Focus on readability over aesthetics.

#### 6. Bold Expressive
- **Typography**: Statement display fonts (e.g., Space Grotesk, Syne, Cabinet Grotesk, Clash Display) + contrasting body
- **Palette**: High saturation. Unexpected combinations. Dark backgrounds with neon accents. Or pure white with a single bold color.
- **Layout**: Breaking the grid. Oversized headings. Diagonal flow. Overlapping elements. Scroll-triggered reveals.
- **Motion**: Rich — page transitions, scroll-linked animations, hover transformations, parallax
- **Spacing**: Dramatic contrasts — tight in some areas, extremely open in others.

## Font Selection Criteria

Since every project gets custom fonts, follow these rules:

1. **Never reuse fonts across projects** — each client gets a unique typographic identity
2. **Always pair a display font with a body font** — same font for both is lazy
3. **Test at multiple sizes** — display fonts must work at h1 AND h3 sizes
4. **Check language support** — verify the font covers all characters the client needs (accents, special chars)
5. **Load via `next/font`** — always. No external font CDN links.
6. **Maximum 2 families** — one display, one body. More than 2 hurts performance.
7. **Avoid overused fonts** — do NOT use: Inter (alone), Roboto, Open Sans, Montserrat, Poppins, Lato, or Raleway as display fonts. These are fine for body text paired with a distinctive display font.

**Where to find distinctive fonts:**
- Google Fonts (filter by popularity: avoid top 20)
- Fontsource (local hosting for any Google Font)
- Variable fonts preferred (one file, multiple weights)

## Component Library: shadcn/ui

Always use **shadcn/ui** as the component base:

```bash
npx shadcn@latest add button input textarea label sheet
```

**Customization approach:**
- Override via CSS variables in `globals.css` (colors, radius, fonts)
- Override via Tailwind classes on the component (spacing, sizing)
- Never edit files in `components/ui/` directly — customize via wrapper components if needed

**Essential components to install per project:**
- `button`, `input`, `textarea`, `label` — forms
- `sheet` — mobile navigation
- `dialog` — modals
- `separator` — visual dividers
- `skeleton` — loading states
- Add others as needed per project

## Animation Tiers

Select the animation tier based on the aesthetic direction:

### Tier 1: Subtle (Trustworthy Minimal, Refined Professional)
```typescript
// Fade in on scroll, once
<m.div
  initial={{ opacity: 0, y: 16 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true }}
  transition={{ duration: 0.5 }}
/>
```

### Tier 2: Moderate (Editorial Luxury, Warm Organic, Clean Technical)
```typescript
// Staggered reveals + hover effects
<m.div
  initial={{ opacity: 0, y: 20 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, margin: "-100px" }}
  transition={{ duration: 0.6, delay: index * 0.08 }}
/>

// Hover lift on cards
<m.div whileHover={{ y: -4, transition: { duration: 0.2 } }} />
```

### Tier 3: Rich (Bold Expressive)
```typescript
// Page transitions, parallax, character animations
// Use AnimatePresence for page transitions
// Use useScroll + useTransform for parallax
// Use staggerChildren for text reveals
```

**Always respect `prefers-reduced-motion`:**
```typescript
const prefersReducedMotion = usePrefersReducedMotion();
const animation = prefersReducedMotion
  ? { opacity: 1, y: 0 }
  : { opacity: 0, y: 20 };
```

## Implementation Workflow

For each component/page:

1. **Select direction** — run decision tree if not done yet
2. **Choose fonts** — pick a unique pairing, test at multiple sizes
3. **Define palette** — set CSS variables in globals.css, include dark mode
4. **Build mobile-first** — start at 390px, add responsive breakpoints going up
5. **Add motion** — select animation tier, implement with LazyMotion + m
6. **Test accessibility** — keyboard nav, contrast, screen reader, focus indicators
7. **Screenshot at 390px + 1280px** — verify responsive layout
8. **Check dark mode** — toggle and verify visually
9. **Check performance** — no unnecessary JS, images optimized

## Anti-Patterns

- **Cookie-cutter designs** — if two client projects could swap stylesheets and look the same, you failed
- **Desktop-first** — designing at 1280px and "making it fit" on mobile always produces bad mobile UX
- **Ignoring dark mode** — adding it later is 5x harder. Build it from the start with CSS variables
- **Animation overload** — more animation ≠ better design. Match the tier to the aesthetic direction
- **Generic font choices** — Inter + no display font = looks like every other site built with AI
- **Decorating instead of designing** — gradients, shadows, and borders don't make a design good. Typography, spacing, and hierarchy do
- **Skipping the decision tree** — jumping to code without choosing a direction produces incoherent designs
