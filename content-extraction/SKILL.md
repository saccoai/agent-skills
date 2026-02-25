---
name: content-extraction
description: Extract all content from an existing website into structured data files. Crawls pages, downloads images/assets, catalogs links, and outputs TypeScript data files and a content inventory. Use when migrating a site, auditing content, or building a content map.
---

This skill extracts ALL content from an existing website and outputs it as structured, reusable data files. It crawls every page, downloads every asset, and produces a complete content inventory.

The user provides: the URL of the site to extract from, and optionally the target format (TypeScript, JSON, Markdown).

## What Gets Extracted

For **each page** on the site:

| Content type | Output |
|-------------|--------|
| Text | Headings, paragraphs, lists, quotes — preserved with hierarchy |
| Metadata | `<title>`, `<meta description>`, OG tags, canonical URL, `lang` |
| Images | Downloaded to `public/images/` with original filenames. Alt text cataloged |
| Links | Internal + external, with anchor text and destination URL |
| PDFs & assets | Downloaded to `public/assets/`. Filenames and original URLs cataloged |
| Forms | Field names, types, labels, validation rules, action URLs |
| Navigation | Menu structure, link hierarchy, active states |
| Structured data | JSON-LD, microdata, schema.org markup |

## Extraction Process

### Step 1: Discover all pages

Use browser automation (agent-browser or Playwright) to:

```
1. Start at the site root
2. Extract all internal links from navigation, footer, sitemap.xml
3. Follow every internal link recursively
4. Build a complete page list with URLs
5. Detect and note any client-side routing (SPA)
```

### Step 2: Extract content per page

For each discovered page:

```
1. Navigate to the page
2. Wait for full load (networkidle)
3. Extract the DOM structure:
   - All headings (h1-h6) with hierarchy
   - All paragraphs and text blocks
   - All lists (ordered/unordered)
   - All images (src, alt, dimensions)
   - All links (href, text, target)
   - All forms (fields, labels, actions)
4. Extract <head> metadata
5. Take a full-page screenshot for reference
6. Save structured data to output files
```

### Step 3: Download assets

```
1. Download all images to public/images/{page-slug}/
2. Download all PDFs to public/assets/
3. Download favicons, OG images, other static assets
4. Preserve original filenames where possible
5. Note any broken/404 asset URLs
```

### Step 4: Generate output files

#### content-inventory.md
Human-readable summary of everything extracted:

```markdown
# Content Inventory — {site-name}

## Pages ({count})

### / (Homepage)
- Title: "Site Name — Tagline"
- Description: "Meta description here"
- H1: "Main Heading"
- Sections: Hero, Features (3), CTA, Testimonials (4)
- Images: 8 (hero.jpg, feature-1.png, ...)
- Links: 12 internal, 3 external
- Forms: none

### /about
...
```

#### src/data/*.ts (TypeScript format)
Structured data files ready for import:

```typescript
// src/data/pages.ts
export interface Page {
  slug: string;
  url: string;
  title: string;
  description: string;
  headings: { level: number; text: string }[];
  sections: Section[];
}

// src/data/navigation.ts
export interface NavItem {
  label: string;
  href: string;
  children?: NavItem[];
}

// src/data/images.ts
export interface ImageAsset {
  originalUrl: string;
  localPath: string;
  alt: string;
  width?: number;
  height?: number;
  page: string;
}
```

#### screenshots/
Full-page screenshots of every page for visual reference.

## Agent Team Integration

When used as a teammate in `website-refactor`, this skill runs as the `content-extractor` agent:

- **Owns**: `src/data/`, `scripts/extract/`, `public/images/`, `public/assets/`, `content-inventory.md`, `screenshots/`
- **Outputs**: Structured data files that `designer` and `content-verifier` teammates consume
- **Signals completion**: By marking its task complete and confirming `content-inventory.md` is written

## Standalone Usage

Can be invoked independently for:
- **Content audits**: "How much content does this site have?"
- **Migration planning**: "Extract everything from the old site before we rebuild"
- **Competitive analysis**: "What content does competitor X have?"
- **Archival**: "Save a complete copy of this site's content"

## Output Formats

The default output is TypeScript (`.ts` files). Pass format preference as an argument:

- `typescript` (default) — `src/data/*.ts` with interfaces and typed exports
- `json` — `content/*.json` files
- `markdown` — `content/*.md` files with frontmatter

## Common Pitfalls

- **SPAs and client-side routing**: Some sites don't render content without JavaScript. Always use a real browser (Playwright/agent-browser), not HTTP fetches.
- **Lazy-loaded content**: Scroll the full page before extracting to trigger lazy-loaded images and infinite scroll sections.
- **Authentication walls**: Some content may be behind login. Note these pages as "requires auth" in the inventory.
- **Rate limiting**: Add delays between page fetches to avoid being blocked. 1-2 seconds between requests is safe.
- **Duplicate content**: The same content may appear on multiple pages (shared sections, footers). Deduplicate in the data files.
- **Relative URLs**: Always resolve to absolute URLs before cataloging or downloading.
