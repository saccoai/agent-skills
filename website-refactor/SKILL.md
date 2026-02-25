---
name: website-refactor
description: End-to-end website refactoring workflow — from content extraction through redesign, implementation, QA, and deployment. Use when the user wants to rebuild, redesign, or migrate an existing website. Covers planning, content preservation, visual QA, mobile testing, accessibility, performance, and deployment.
---

This skill orchestrates a complete website refactor from an existing site to a modern stack. It ensures no content is lost, design quality is high, and the result is production-ready across all devices and browsers.

The user provides: the URL of the existing site, the target tech stack, and design direction (preserve brand, modernize, or fully redesign).

## Workflow Phases

Execute these phases in order. Each phase has defined inputs, outputs, and quality gates.

### Phase 1: Planning & Spec (Hybrid Approach)

Before writing any code, create a structured spec:

1. **Crawl the existing site** — Use browser automation (agent-browser or Playwright) to:
   - Map all pages and routes (build a sitemap)
   - Extract all text content, headings, metadata (title, description, OG tags)
   - Inventory all images, PDFs, and downloadable assets
   - Catalog all external links and their destinations
   - Note any forms, interactive elements, or dynamic features
   - Save the extracted content to structured data files

2. **Create the spec document** covering:
   - Page inventory with route mapping (old URL → new URL)
   - Content inventory per page (headings, paragraphs, lists, media)
   - Features list (forms, navigation, animations, SEO)
   - Design direction (preserve brand? modernize? full redesign?)
   - Technical constraints (framework, hosting, dependencies)
   - Redirects needed (old routes → new routes)

3. **Present the spec to the user** for review before proceeding. Ask clarifying questions about:
   - Any pages to add, remove, or merge
   - Design preferences (colors, fonts, layout style)
   - New features to add (contact form, analytics, etc.)
   - Deployment target and domain setup

**Quality gate**: User approves the spec before Phase 2 begins.

### Phase 2: Project Setup & Content Migration

1. **Scaffold the project** with the target stack
2. **Migrate all content** into structured data files or page components:
   - Text content → data files (TypeScript objects) or directly in page components
   - Images → public/images directory with proper optimization
   - PDFs/assets → public directory with organized structure
3. **Set up shared components**: layout, header, footer, navigation
4. **Configure metadata**: page titles, descriptions, OG tags for each route
5. **Set up redirects** for any changed routes

**Quality gate**: All original content is present in the codebase. Run automated content diff (see Phase 4).

### Phase 3: Design & Implementation

For each page, follow this cycle:

1. **Implement the page** with the agreed design direction
2. **Immediately test at mobile (390px) AND desktop (1280px+)** using Playwright screenshots
3. **Fix any responsive issues** before moving to the next page
4. **Verify content matches** the original page

Use the `frontend-design` skill for design quality if the user wants a distinctive aesthetic.

**Implementation order** (recommended):
1. Global layout (header, footer, globals.css)
2. Homepage
3. Content pages (simplest first)
4. Complex pages (forms, interactive elements)
5. Detail/sub-pages

**Per-page checklist**:
- [ ] All original content present
- [ ] Responsive at 390px mobile viewport
- [ ] Responsive at 768px tablet viewport
- [ ] Responsive at 1280px+ desktop viewport
- [ ] Images use next/image (or equivalent) with proper sizes/priority
- [ ] Links work (internal and external)
- [ ] Animations/interactions work
- [ ] Metadata (title, description) set correctly

### Phase 4: Automated Content Verification

Write and run verification scripts that:

1. **Crawl the original site** and extract:
   - All text content (headings, paragraphs)
   - All links (href values) with their anchor text
   - All image sources and alt text
   - All downloadable file URLs
   - Page titles and meta descriptions

2. **Crawl the new site** and extract the same data

3. **Diff the results** to catch:
   - Missing text content
   - Broken or missing links
   - Missing images or wrong alt text
   - Missing pages or wrong redirects
   - Changed metadata

4. **Generate a report** showing what matches and what's missing

**Quality gate**: Content diff shows 100% content parity (or user-approved exceptions).

### Phase 5: Quality Audits

Run ALL of the following on every page:

#### 5a. Lighthouse Audit
```bash
# Run Lighthouse CI on each page
npx lighthouse <url> --output=json --output=html --chrome-flags="--headless"
```
Target scores:
- Performance: ≥ 90
- Accessibility: ≥ 95
- Best Practices: ≥ 90
- SEO: ≥ 95

#### 5b. Accessibility Audit
- Run axe-core or similar on every page
- Test keyboard navigation (Tab, Enter, Escape)
- Verify screen reader compatibility (proper headings, ARIA labels, alt text)
- Check color contrast ratios (WCAG AA minimum)
- Test focus indicators visibility
- Verify form labels and error messages

#### 5c. Cross-Browser Testing
Test on at minimum:
- Chrome/Chromium (desktop + mobile)
- Safari (desktop + iOS)
- Firefox (desktop)

Use Playwright's browser contexts or agent-browser for automated screenshots across browsers.

#### 5d. Performance Budget
Check and enforce:
- **LCP** (Largest Contentful Paint): < 2.5s
- **CLS** (Cumulative Layout Shift): < 0.1
- **INP** (Interaction to Next Paint): < 200ms
- **Bundle size**: Monitor JS bundle size
- **Images**: All images properly sized and using modern formats (WebP/AVIF)

**Quality gate**: All audits pass minimum thresholds, or issues are documented and user-approved.

### Phase 6: Final Review & Deployment

1. **Run full mobile test suite** — screenshot every page at 390x844
2. **Test all interactive elements**:
   - Navigation (desktop + mobile hamburger menu)
   - Forms (submit, validation, error states, success states)
   - Links (internal navigation, external links open in new tab)
   - Animations (scroll-triggered, hover, page transitions)
3. **Verify environment variables** are documented (.env.example)
4. **Deploy to staging** and verify
5. **Deploy to production** with proper domain configuration

## Key Principles

- **Content is king**: Never lose original content. Automate verification.
- **Mobile-first QA**: Test mobile immediately after each page, not at the end.
- **Hybrid planning**: Spec the structure upfront, iterate on visual details after first pass.
- **Automate what you can**: Content diffs, Lighthouse, axe-core — don't rely on manual checks for things scripts can verify.
- **Progressive enhancement**: Ensure the site works without JavaScript for core content. Animations are enhancements, not requirements.

## Common Pitfalls

- **whileInView animations**: These cause blank areas in headless full-page screenshots. Always scroll the page to trigger them, or temporarily disable for screenshot QA.
- **Client components and metadata**: In Next.js App Router, `"use client"` pages can't export metadata. Use a sibling `layout.tsx` for metadata instead.
- **Image optimization**: Don't just copy images — ensure they're properly sized, use modern formats, and have correct `sizes` attributes.
- **Form setup**: Decide on email/backend provider during planning, not after implementation. Document all required environment variables early.
- **Redirects**: Set up redirects for ALL changed URLs to preserve SEO and existing bookmarks.
- **PDF/asset links**: Verify all downloadable file links work on the new site. These are easy to miss.
