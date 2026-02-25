---
name: web-audit
description: Comprehensive website quality audit — Lighthouse scores, accessibility (axe-core), cross-browser testing, performance budgets, and mobile responsiveness. Generates actionable reports with pass/fail per page. Use to audit any live website or as a QA gate before deployment.
---

This skill runs a comprehensive quality audit on any website. It tests every page across multiple dimensions and generates actionable reports with specific issues and fix recommendations.

The user provides: the URL of the site to audit, and optionally specific pages or audit categories to focus on.

## Audit Categories

### 1. Lighthouse Audit

Run Google Lighthouse on **every page**:

```bash
npx lighthouse {URL} --output=json --output=html --chrome-flags="--headless" --only-categories=performance,accessibility,best-practices,seo
```

**Target scores:**

| Category | Minimum | Ideal |
|----------|---------|-------|
| Performance | ≥ 90 | ≥ 95 |
| Accessibility | ≥ 95 | 100 |
| Best Practices | ≥ 90 | ≥ 95 |
| SEO | ≥ 95 | 100 |

Save HTML reports to `qa-reports/lighthouse/{page-slug}.html` for detailed review.

### 2. Accessibility Audit

Run axe-core on every page via Playwright:

```javascript
const { AxeBuilder } = require('@axe-core/playwright');
const results = await new AxeBuilder({ page }).analyze();
```

**Check for:**
- Missing alt text on images
- Insufficient color contrast (WCAG AA: 4.5:1 normal text, 3:1 large text)
- Missing form labels and ARIA attributes
- Heading hierarchy violations (skipped levels)
- Missing landmark regions (nav, main, footer)
- Keyboard traps (elements that can't be Tab-escaped)
- Focus indicator visibility
- Missing `lang` attribute on `<html>`
- ARIA roles used correctly

**Manual keyboard test** (via Playwright):
```
1. Tab through every interactive element on each page
2. Verify focus is visible on each element
3. Verify Enter/Space activates buttons and links
4. Verify Escape closes modals/dropdowns
5. Verify skip-to-content link exists and works
```

### 3. Cross-Browser Testing

Take screenshots of every page in **3 browsers × 2 viewports**:

| Browser | Mobile (390px) | Desktop (1280px) |
|---------|---------------|-------------------|
| Chromium | ✓ | ✓ |
| Firefox | ✓ | ✓ |
| WebKit (Safari) | ✓ | ✓ |

```javascript
// Playwright multi-browser
for (const browserType of ['chromium', 'firefox', 'webkit']) {
  const browser = await playwright[browserType].launch();
  // ... screenshot each page at each viewport
}
```

Save to `qa-reports/browsers/{browser}/{viewport}/{page-slug}.png`.

Flag any visual differences between browsers (layout shifts, font rendering, missing features).

### 4. Performance Budget

Measure Core Web Vitals on every page:

| Metric | Target | Method |
|--------|--------|--------|
| **LCP** (Largest Contentful Paint) | < 2.5s | Lighthouse / Performance Observer |
| **CLS** (Cumulative Layout Shift) | < 0.1 | Lighthouse / Performance Observer |
| **INP** (Interaction to Next Paint) | < 200ms | Chrome DevTools Protocol |
| **TTFB** (Time to First Byte) | < 800ms | Navigation Timing API |
| **FCP** (First Contentful Paint) | < 1.8s | Lighthouse |

**Additional checks:**
- Total JS bundle size (flag if > 200KB gzipped)
- Total page weight (flag if > 2MB)
- Number of HTTP requests (flag if > 50)
- Images not using modern formats (WebP/AVIF)
- Images without explicit width/height (causes CLS)
- Unminified CSS/JS
- Missing compression (gzip/brotli)
- Render-blocking resources

### 5. Mobile Responsiveness

For each page at **390×844 viewport** (iPhone 14):

- **No horizontal scrollbar**: page fits within viewport width
- **Touch targets**: all interactive elements ≥ 44×44px
- **Text readability**: no text smaller than 16px without zoom
- **Navigation**: hamburger/mobile menu opens and all links are accessible
- **Forms**: all inputs are usable, keyboard doesn't obscure fields
- **Images**: responsive, not overflowing container
- **Tables**: scrollable or responsive layout on narrow screens

## Report Output

### qa-summary.md (top-level overview)

```markdown
# QA Audit Summary — {site-name}
Date: {date}
Pages audited: {count}

## Overall Scores
| Page | Perf | A11y | BP | SEO | LCP | CLS | Mobile |
|------|------|------|----|-----|-----|-----|--------|
| / | 94 | 100 | 95 | 100 | 1.8s | 0.02 | ✅ |
| /about | 87 | 95 | 90 | 95 | 3.1s | 0.12 | ⚠️ |

## Critical Issues (must fix)
- /about: LCP 3.1s — hero image is 2.4MB unoptimized PNG
- /contact: form 'subject' input missing <label>

## Warnings (should fix)
- /blog: 3 images missing alt text
- /pricing: CLS 0.12 from font swap

## Passed
- All pages keyboard navigable
- All pages pass WCAG AA contrast
- No cross-browser rendering issues
```

### Per-page detailed reports in qa-reports/

```
qa-reports/
├── lighthouse/
│   ├── homepage.html
│   ├── about.html
│   └── ...
├── accessibility/
│   ├── homepage.json      (axe-core results)
│   └── ...
├── browsers/
│   ├── chromium/
│   │   ├── mobile/
│   │   └── desktop/
│   ├── firefox/
│   └── webkit/
├── performance/
│   └── metrics.json
└── mobile/
    ├── homepage-390.png
    └── ...
```

## Agent Team Integration

When used as the `qa-auditor` teammate in `website-refactor`:

- **Owns**: `qa-reports/`, `qa-summary.md`, `scripts/audit/`
- **Outputs**: `qa-summary.md` with pass/fail per page, detailed reports in `qa-reports/`
- **Communicates issues**: Via SendMessage to `designer` teammate with specific fixes needed
- **Signals completion**: By marking its task complete after all audits pass or issues are documented

## Standalone Usage

Invoke independently to audit any live site:
- **Pre-launch QA**: "Audit staging.example.com before we go live"
- **Competitor analysis**: "How does competitor.com score on performance and a11y?"
- **Sales tool**: "Audit prospect.com and show them what's broken — generate a proposal"
- **Periodic monitoring**: "Run monthly audits on our production site"

## Common Pitfalls

- **Lighthouse variance**: Scores vary between runs. Run 3 times and take the median.
- **Local vs production**: Always audit the deployed site, not localhost (different network conditions, CDN, compression).
- **Auth-protected pages**: Some pages may require login. Use Playwright's storage state to persist auth.
- **whileInView animations**: Framer Motion / GSAP animations that trigger on scroll cause blank areas in screenshots. Scroll the full page before capturing.
- **Third-party scripts**: Analytics, chat widgets, etc. can tank performance scores. Note which issues are from third-party vs first-party code.
