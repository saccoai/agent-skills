# saccoai/agent-skills

A collection of agent skills for Claude Code and other AI coding agents.

## Skills

### website-refactor

End-to-end website refactoring orchestrator using agent teams. Composes the three skills below as specialized teammates alongside a designer agent.

```bash
npx skills add saccoai/agent-skills@website-refactor
```

### content-extraction

Extract all content from an existing website into structured data files. Crawls pages, downloads images/assets, catalogs links, and outputs TypeScript data files and a content inventory.

```bash
npx skills add saccoai/agent-skills@content-extraction
```

### web-audit

Comprehensive website quality audit — Lighthouse, accessibility (axe-core), cross-browser testing, performance budgets, and mobile responsiveness. Generates actionable reports with pass/fail per page.

```bash
npx skills add saccoai/agent-skills@web-audit
```

### seo-migration

SEO preservation during website migrations — redirect mapping, canonical URLs, sitemap generation, structured data, meta tags, and validation.

```bash
npx skills add saccoai/agent-skills@seo-migration
```

## How They Work Together

```
website-refactor (orchestrator)
├── content-extraction  →  content-extractor teammate
├── web-audit           →  qa-auditor teammate
├── seo-migration       →  seo-manager teammate
└── frontend-design*    →  designer teammate

* frontend-design is a separate skill (not in this repo)
```

Each skill works standalone or as a teammate in the `website-refactor` agent team workflow.

## Installation

Install individual skills:

```bash
npx skills add saccoai/agent-skills@web-audit -g
```

Install all skills:

```bash
npx skills add saccoai/agent-skills --all -g
```

## License

MIT
