# saccoai/agent-skills

Agent skills for AI-powered web development and consultancy workflows.

## Skills

### Orchestrator

| Skill | Description |
|-------|-------------|
| **website-refactor** | End-to-end website refactoring using agent teams. Composes website-analysis, web-audit, and seo-migration as specialized teammates |

```bash
npx skills add saccoai/agent-skills@website-refactor
```

### Core Skills (used standalone or as teammates)

| Skill | Description |
|-------|-------------|
| **website-analysis** | Single-pass crawl that produces both the structural map and full content extraction. Discovers pages, navigation, multilingual variants, and issues while extracting all text, images, and assets. Replaces the old website-structure + content-extraction combo |
| **web-audit** | Lighthouse, accessibility, cross-browser, performance budgets, mobile responsiveness |
| **seo-migration** | Redirect mapping, canonical URLs, sitemap, structured data, meta tags |

```bash
npx skills add saccoai/agent-skills@website-analysis
npx skills add saccoai/agent-skills@web-audit
npx skills add saccoai/agent-skills@seo-migration
```

### Client Delivery

| Skill | Description |
|-------|-------------|
| **client-proposal** | Generate a professional project proposal from a website audit |
| **project-handoff** | Generate complete client handoff documentation |

```bash
npx skills add saccoai/agent-skills@client-proposal
npx skills add saccoai/agent-skills@project-handoff
```

### Design

| Skill | Description |
|-------|-------------|
| **frontend-design** | Adaptive design direction via decision tree, WCAG AA, mobile-first, dark mode, performance budgets. Replaces the community frontend-design with agency-specific standards |

```bash
npx skills add saccoai/agent-skills@frontend-design
```

### Tech Stack

| Skill | Description |
|-------|-------------|
| **nextjs-fullstack** | Opinionated Next.js patterns — App Router, Tailwind v4, shadcn/ui, Better Auth, Drizzle ORM |

```bash
npx skills add saccoai/agent-skills@nextjs-fullstack
```

## How They Work Together

```
website-refactor (orchestrator — agent team)
├── website-analysis   →  single-pass crawl: structure + content (Phase 1)
├── web-audit          →  qa-auditor teammate
├── seo-migration      →  seo-manager teammate
├── frontend-design    →  designer teammate (aesthetic direction + standards)
├── nextjs-fullstack   →  architecture reference for all teammates
├── client-proposal    →  pre-engagement (sales/scoping)
└── project-handoff    →  post-delivery (final deliverable)

```

## Install All

```bash
npx skills add saccoai/agent-skills --all -g
```

## License

MIT
