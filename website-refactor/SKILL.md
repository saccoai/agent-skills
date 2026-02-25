---
name: website-refactor
description: End-to-end website refactoring workflow using agent teams — from content extraction through redesign, implementation, QA, and deployment. Use when the user wants to rebuild, redesign, or migrate an existing website. Spawns specialized teammates for parallel content extraction, design, QA, and deployment.
---

This skill orchestrates a complete website refactor using **Claude Code agent teams**. It spawns specialized teammates that work in parallel — extracting content, implementing design, running QA audits, and verifying content — to refactor a website faster and more thoroughly than a single agent.

The user provides: the URL of the existing site, the target tech stack, and design direction (preserve brand, modernize, or fully redesign).

## Prerequisites

Agent teams must be enabled:

```json
// ~/.claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## Agent Team Roles

Spawn these teammates. Each owns specific files to **avoid edit conflicts**:

### Lead (you)
- **Role**: Orchestrator — creates the team, defines tasks, coordinates handoffs, synthesizes results
- **Does NOT**: Implement pages or run audits directly (use delegate mode: `Shift+Tab`)
- **Tools**: TeamCreate, TaskCreate, TaskUpdate, TaskList, SendMessage

### content-extractor
- **Focus**: Crawling the original site and extracting all content
- **Owns**: `src/data/`, `scripts/extract/`, `content-inventory.md`
- **Tools**: Browser automation (agent-browser or Playwright), Read, Write, Bash
- **Prompt template**: See Phase 1 below

### designer
- **Focus**: Implementing page designs with the agreed aesthetic direction
- **Owns**: `src/app/*/page.tsx`, `src/components/`, `src/app/globals.css`
- **Tools**: Read, Write, Edit, Bash (dev server), Playwright (screenshots)
- **Prompt template**: See Phase 3 below

### qa-auditor
- **Focus**: Lighthouse, accessibility, cross-browser, performance, mobile responsiveness
- **Owns**: `qa-reports/`, `scripts/audit/`
- **Tools**: Bash (lighthouse, axe-core), Playwright (screenshots at multiple viewports/browsers)
- **Prompt template**: See Phase 5 below

### content-verifier
- **Focus**: Automated diffing of original vs new site content
- **Owns**: `scripts/verify/`, `content-diff-report.md`
- **Tools**: Browser automation, Bash, Read, Write
- **Prompt template**: See Phase 4 below

## Workflow Phases

### Phase 1: Planning & Content Extraction

**Lead** creates the team and spawns `content-extractor`:

```
Create an agent team "website-refactor".

Spawn teammate "content-extractor" with prompt:
"Crawl {ORIGINAL_URL} and extract ALL content. For each page:
1. Map the route/URL
2. Extract all text (headings, paragraphs, lists)
3. Extract all metadata (title, description, OG tags)
4. Inventory all images with alt text and download to public/images/
5. Catalog all links (internal + external) with anchor text
6. Note all PDFs/downloadable assets and download to public/
7. Identify forms, interactive elements, dynamic features

Save results to:
- content-inventory.md (human-readable summary)
- src/data/*.ts (structured TypeScript data files)
- public/images/ (all images)
- public/ (PDFs and assets)

The target stack is {TECH_STACK}. Structure data files accordingly."
```

**While content-extractor works**, the Lead:
1. Creates a spec document from initial page discovery
2. Asks the user clarifying questions (pages to add/remove, design preferences, new features)
3. Documents redirects (old URL → new URL)

**Quality gate**: content-extractor completes. Lead reviews `content-inventory.md` and confirms completeness with the user.

### Phase 2: Project Setup

**Lead** scaffolds the project (or the designer teammate does this):
1. Initialize the project with the target stack
2. Set up shared components: layout, header, footer, navigation
3. Configure metadata for each route
4. Set up redirects for changed routes
5. Document environment variables in `.env.example`

This phase is sequential — must complete before design begins.

### Phase 3: Design & Implementation

**Lead** spawns the `designer` teammate with page-by-page tasks:

```
Spawn teammate "designer" with prompt:
"Implement all pages for {SITE_NAME} using {TECH_STACK}.
Design direction: {DESIGN_DIRECTION}
Brand: {COLORS, FONTS, CONSTRAINTS}

Content is already extracted in src/data/ and content-inventory.md. Read these first.

CRITICAL WORKFLOW — for EACH page:
1. Implement the page using the extracted content
2. Take a Playwright screenshot at 390px width (mobile)
3. Take a Playwright screenshot at 1280px width (desktop)
4. Fix any responsive issues before moving to the next page
5. Mark the page task as complete

Implementation order:
1. Global layout (header, footer, globals.css)
2. Homepage
3. Content pages (simplest first)
4. Complex pages (forms, interactive elements)
5. Detail/sub-pages

Per-page checklist:
- All original content present (compare with content-inventory.md)
- Responsive at 390px, 768px, 1280px+
- Images optimized with proper sizes/priority attributes
- Internal and external links work
- Animations/interactions work
- Page metadata (title, description) set correctly"
```

**Lead** creates individual TaskCreate entries for each page so progress is trackable.

### Phase 4: Automated Content Verification

**After the designer completes**, Lead spawns `content-verifier`:

```
Spawn teammate "content-verifier" with prompt:
"Verify that the new site at {NEW_URL} has 100% content parity with {ORIGINAL_URL}.

Write and run scripts that:
1. Crawl {ORIGINAL_URL} — extract all headings, paragraphs, links (href + text), images (src + alt), downloadable files, page titles, meta descriptions
2. Crawl {NEW_URL} — extract the same data
3. Diff the results and generate content-diff-report.md showing:
   - ✅ Content that matches
   - ❌ Missing text content
   - ❌ Broken or missing links
   - ❌ Missing images or wrong alt text
   - ❌ Missing pages or wrong redirects
   - ❌ Changed metadata

Save scripts to scripts/verify/ for future re-runs.
Target: 100% content parity (or document approved exceptions)."
```

**Quality gate**: content-diff-report.md shows full parity. Lead reviews any discrepancies with the user.

### Phase 5: Quality Audits (Parallel)

**Lead** spawns `qa-auditor` to run ALL audits in parallel:

```
Spawn teammate "qa-auditor" with prompt:
"Run comprehensive quality audits on {NEW_URL}. Test EVERY page.

1. LIGHTHOUSE AUDIT
   Run on each page. Target scores:
   - Performance: ≥ 90
   - Accessibility: ≥ 95
   - Best Practices: ≥ 90
   - SEO: ≥ 95

2. ACCESSIBILITY AUDIT
   - Run axe-core on every page
   - Test keyboard navigation (Tab, Enter, Escape)
   - Verify screen reader compatibility (headings, ARIA, alt text)
   - Check color contrast (WCAG AA minimum)
   - Test focus indicators
   - Verify form labels and error messages

3. CROSS-BROWSER TESTING
   Take screenshots on each page at mobile (390px) and desktop (1280px+) in:
   - Chromium
   - Firefox
   - WebKit (Safari)
   Use Playwright's browser contexts.

4. PERFORMANCE BUDGET
   - LCP < 2.5s
   - CLS < 0.1
   - INP < 200ms
   - Check JS bundle size
   - Verify images use modern formats (WebP/AVIF)

5. MOBILE RESPONSIVENESS
   For each page, screenshot at 390x844 viewport and verify:
   - No horizontal scroll
   - Touch targets ≥ 44px
   - Text readable without zooming
   - Navigation (hamburger menu) works

Save all results to qa-reports/ with per-page breakdowns.
Generate qa-summary.md with pass/fail for each check."
```

**If qa-auditor finds issues**, Lead sends them to `designer` via SendMessage:

```
SendMessage to "designer":
"QA found these issues — please fix:
- /biografia: LCP 3.2s (hero image not optimized)
- /contatto: missing form label on 'subject' field
- /hobbies: CLS 0.15 (image shift on load)
See qa-reports/ for details."
```

### Phase 6: Final Review & Deployment

**Lead** coordinates the final steps:

1. Review qa-summary.md — all checks pass
2. Review content-diff-report.md — 100% parity
3. Test interactive elements manually or via Playwright:
   - Navigation (desktop + mobile hamburger)
   - Forms (submit, validation, error/success states)
   - External links open in new tab
4. Verify `.env.example` documents all required variables
5. Deploy to staging, verify
6. Deploy to production

## Task Dependencies

Structure tasks with proper dependencies so teammates don't start too early:

```
TaskCreate: "Extract all content"          → no dependencies (starts immediately)
TaskCreate: "Scaffold project"             → no dependencies (starts immediately)
TaskCreate: "Implement homepage"           → depends on: scaffold, extract
TaskCreate: "Implement content pages"      → depends on: scaffold, extract
TaskCreate: "Implement complex pages"      → depends on: scaffold, extract
TaskCreate: "Verify content parity"        → depends on: all implementation tasks
TaskCreate: "Run quality audits"           → depends on: all implementation tasks
TaskCreate: "Fix audit issues"             → depends on: quality audits
TaskCreate: "Deploy"                       → depends on: verify, fix audit issues
```

## File Ownership (Avoid Conflicts)

| Teammate | Owns (read/write) | Can read |
|----------|-------------------|----------|
| content-extractor | `src/data/`, `scripts/extract/`, `public/images/`, `content-inventory.md` | Original site |
| designer | `src/app/`, `src/components/`, `src/app/globals.css`, `tailwind.config.*` | `src/data/`, `content-inventory.md` |
| content-verifier | `scripts/verify/`, `content-diff-report.md` | Original site, new site |
| qa-auditor | `qa-reports/`, `qa-summary.md`, `scripts/audit/` | New site (all pages) |

## Key Principles

- **Delegate, don't implement**: Lead orchestrates. Teammates do the work. Enable delegate mode (`Shift+Tab`) early.
- **Content is king**: Never lose original content. Automate verification with content-verifier.
- **Mobile-first QA**: Designer tests mobile + desktop after EACH page, not at the end.
- **Parallel where possible**: content-extractor + scaffold run in parallel. qa-auditor + content-verifier run in parallel after implementation.
- **Communicate via SendMessage**: When qa-auditor finds issues, send them to designer. Don't have the Lead fix things.
- **3-5 teammates max**: More than 5 adds coordination overhead without proportional benefit.

## Common Pitfalls

- **File conflicts**: Two teammates editing the same file causes overwrites. Enforce file ownership strictly.
- **whileInView animations**: Cause blank areas in headless screenshots. Designer should scroll pages before taking screenshots, or temporarily disable animations for QA.
- **Client components and metadata**: In Next.js App Router, `"use client"` pages can't export metadata. Use a sibling `layout.tsx`.
- **Lead doing implementation**: Resist the urge. Use delegate mode and let teammates handle the work.
- **Missing task dependencies**: If designer starts before content-extractor finishes, they'll have no content to implement. Set dependencies explicitly.
- **Form/SMTP setup**: Decide on email provider during Phase 1 planning. Document env vars in `.env.example` during Phase 2.
- **Redirects**: Set up redirects for ALL changed URLs during Phase 2 to preserve SEO.
