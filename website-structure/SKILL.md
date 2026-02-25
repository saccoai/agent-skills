---
name: website-structure
description: Analyze and map the complete structure of any website by recursively traversing all links. Discovers all pages, routes, navigation patterns, link relationships, and content hierarchy. Use before any migration, redesign, or audit to ensure nothing is missed.
---

This skill recursively traverses an entire website and produces a complete structural map — every page, every link, every route, every asset. It's the first thing to run before any migration or redesign to ensure no content is missed.

The user provides: the root URL of the site to analyze.

## Why This Skill Exists

The most common mistake in website migrations is **missing pages**. Sites have:
- Gallery sub-pages linked from cards
- PDF downloads buried in article pages
- Pagination on blog/article listings
- Accordion/tab content that hides links
- Footer links to pages not in the main navigation
- Sitemap.xml entries for pages with no visible links
- Old pages still indexed by Google but removed from navigation

This skill finds ALL of them by traversing every link on every page.

## Analysis Process

### Step 1: Discover Entry Points

Before crawling, gather all known URLs from multiple sources:

```
1. Parse sitemap.xml (and sitemap index files)
2. Parse robots.txt for referenced sitemaps
3. Extract all links from the homepage
4. Extract all links from the main navigation
5. Extract all links from the footer
6. Check common paths: /about, /contact, /blog, /404, /privacy, /terms
```

### Step 2: Recursive Crawl

For each discovered URL, use browser automation (not HTTP fetches — SPAs need real browsers):

```
For each page:
1. Navigate to the URL
2. Wait for full load (networkidle)
3. Scroll the FULL page to trigger lazy-loaded content
4. Click any expandable elements (accordions, tabs, "show more" buttons)
5. Extract ALL links:
   - <a href="..."> links
   - JavaScript navigation (onClick handlers that change routes)
   - Links inside dynamically loaded content
   - Image gallery links / lightbox triggers
   - PDF and asset download links
6. Classify each link:
   - Internal page (same domain, HTML)
   - Internal asset (same domain, PDF/image/file)
   - External link (different domain)
   - Anchor link (same page, #fragment)
   - mailto: / tel: link
7. Add new internal links to the crawl queue
8. Record the page's metadata (title, description, H1)
```

**Crawl rules:**
- Stay within the same domain (don't follow external links)
- Handle trailing slashes consistently (normalize URLs)
- Detect and skip infinite pagination / infinite scroll
- Respect a maximum depth limit (default: 10 levels)
- Detect redirect chains and note the final destination
- Handle URL parameters (ignore tracking params like utm_*)
- Rate limit: 1-2 seconds between page fetches

### Step 3: Classify Pages

Group discovered pages into categories:

| Category | Detection |
|----------|----------|
| **Navigation pages** | Linked from main nav or header |
| **Content pages** | Individual articles, blog posts, case studies |
| **Gallery/collection pages** | Pages that primarily display a grid of images or items |
| **Detail pages** | Sub-pages of a parent (e.g., /gallery/fotografie/) |
| **Utility pages** | 404, privacy policy, terms, sitemap, search |
| **Asset pages** | PDFs, downloads, media files |
| **Orphan pages** | In sitemap but not linked from any other page |
| **Redirect pages** | Return 301/302 to another URL |

### Step 4: Build the Site Map

Generate a hierarchical tree showing the complete structure:

```
/ (Homepage)
├── /biografia/
├── /consulenze/
├── /articoli-conferenze/
├── /pubblicazioni/
│   ├── /la-ferrovia-del-gottardo/
│   └── /die-gotthardbahn/
├── /hobbies/
│   ├── /gallery/fotografie/        ← LINKED FROM HOBBY CARD
│   ├── /gallery/viaggi-in-bici/    ← LINKED FROM HOBBY CARD
│   └── /gallery/dipinti-e-disegni/ ← LINKED FROM HOBBY CARD
├── /contatto/
└── /invia/  → REDIRECT → /contatto/
```

### Step 5: Detect Issues

Flag potential problems:

- **Orphan pages**: In sitemap but no internal links point to them
- **Dead ends**: Pages with no outbound internal links
- **Broken links**: Links pointing to 404 pages
- **Redirect chains**: A → B → C (should be A → C)
- **Duplicate content**: Multiple URLs serving the same content
- **Missing from navigation**: Pages that exist but aren't in any nav menu
- **Deep pages**: Pages more than 3 clicks from the homepage
- **Mixed HTTP/HTTPS**: Links using HTTP when the site is HTTPS

## Output Files

### site-structure.md (Primary deliverable)

```markdown
# Site Structure — {domain}

**Crawled**: {date}
**Total pages**: {count}
**Total assets**: {count}
**External links**: {count}

## Site Tree

/ (Homepage) — "Site Title"
├── /about/ — "About Us"
│   ├── /about/team/ — "Our Team"
│   └── /about/history/ — "Our History"
├── /blog/ — "Blog" (12 posts)
│   ├── /blog/post-1/ — "First Post"
│   ├── /blog/post-2/ — "Second Post"
│   └── ... (10 more)
├── /gallery/ — "Gallery"
│   ├── /gallery/photos/ — "Photos" (24 images)
│   └── /gallery/videos/ — "Videos" (6 videos)
├── /contact/ — "Contact"
└── /privacy/ — "Privacy Policy"

## Navigation Structure

### Main Nav
- Home → /
- About → /about/
- Blog → /blog/
- Gallery → /gallery/
- Contact → /contact/

### Footer Nav
- Privacy → /privacy/
- Terms → /terms/

## Page Inventory

| URL | Title | Type | Links In | Links Out | Assets |
|-----|-------|------|----------|-----------|--------|
| / | Homepage | nav | 0 | 12 | 5 images |
| /about/ | About | nav | 3 | 8 | 2 images |
| /gallery/photos/ | Photos | detail | 1 | 24 | 24 images |
| ... | ... | ... | ... | ... | ... |

## Assets Inventory

| URL | Type | Size | Linked From |
|-----|------|------|------------|
| /uploads/photo1.jpg | image | 245KB | /gallery/photos/ |
| /uploads/report.pdf | PDF | 1.2MB | /about/history/ |

## Issues Found

### ⚠️ Orphan Pages (in sitemap, no internal links)
- /old-event/ — not linked from anywhere

### ❌ Broken Links
- /blog/post-3/ links to /author/john/ → 404

### 🔄 Redirect Chains
- /old-contact/ → /invia/ → /contatto/ (should be /old-contact/ → /contatto/)

### 📊 Deep Pages (>3 clicks from home)
- /blog/2023/archive/post-47/ — 4 clicks deep
```

### site-structure.json (Machine-readable)

```json
{
  "domain": "example.com",
  "crawledAt": "2026-02-25",
  "pages": [
    {
      "url": "/",
      "title": "Homepage",
      "description": "...",
      "h1": "Welcome",
      "type": "navigation",
      "linksIn": [],
      "linksOut": ["/about/", "/blog/", "/contact/"],
      "assets": ["/images/hero.jpg"],
      "metadata": { "title": "...", "description": "..." }
    }
  ],
  "assets": [...],
  "redirects": [...],
  "issues": { "orphans": [...], "broken": [...], "chains": [...] }
}
```

### site-tree.txt (Quick reference)

Plain text tree view for easy scanning:

```
cavadini.ch
├── / (Homepage)
├── /biografia/
├── /consulenze/
├── /articoli-conferenze/
├── /pubblicazioni/
│   ├── /la-ferrovia-del-gottardo/
│   └── /die-gotthardbahn/
├── /hobbies/
│   ├── /gallery/fotografie/ (18 images)
│   ├── /gallery/viaggi-in-bici/ (12 images)
│   └── /gallery/dipinti-e-disegni/ (15 images)
├── /contatto/
└── /invia/ → 301 → /contatto/
```

## Agent Team Integration

This skill should run **before** `content-extraction`:

```
Phase 0: Structure Analysis (website-structure)
    ↓ outputs site-structure.md, site-tree.txt
Phase 1: Content Extraction (content-extraction)
    ↓ uses site-structure.md as the crawl map — no pages missed
Phase 2+: Design, Audit, etc.
```

In the `website-refactor` agent team, this runs as the first task of `content-extractor` or as a separate lightweight pre-scan by the Lead.

## Standalone Usage

- **Pre-migration**: "Map the entire site before we start rebuilding"
- **Discovery**: "How many pages does this site actually have?"
- **Link audit**: "Find all broken links on our production site"
- **Competitor analysis**: "Map the structure of competitor.com"
- **QA**: "Verify our new site has the same structure as the old one"

## Common Pitfalls

- **Missing gallery/detail pages**: Cards or thumbnails link to sub-pages. Always click into every card, not just scrape the listing page.
- **JavaScript navigation**: SPAs use `onClick` handlers or `router.push()` instead of `<a href>`. Use a real browser and detect route changes.
- **Accordion/tab content**: Content hidden behind UI elements often contains links. Click every expandable element before extracting links.
- **Infinite scroll**: Some pages load more content on scroll. Scroll to the bottom (multiple times if needed) before extracting.
- **URL normalization**: `/about`, `/about/`, and `/about/index.html` may all be the same page. Normalize before deduplicating.
- **Query parameters**: `/blog?page=2` is a different page from `/blog`. Include pagination but exclude tracking params.
- **HTTP fetches miss SPAs**: Never use `curl` or `fetch()` alone. Always use Playwright or agent-browser to render JavaScript.
