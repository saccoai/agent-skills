# saccoai/agent-skills

A collection of agent skills for Claude Code and other AI coding agents.

## Skills

### website-refactor

End-to-end website refactoring workflow — from content extraction through redesign, implementation, QA, and deployment.

**Install:**

```bash
npx skills add saccoai/agent-skills@website-refactor
```

Covers 6 phases:

1. **Planning & Spec** — Crawl original site, build content inventory, hybrid spec
2. **Content Migration** — Scaffold project, migrate content, set up redirects
3. **Design & Implementation** — Page-by-page with immediate mobile + desktop QA
4. **Automated Content Verification** — Script-based diff of old vs new site
5. **Quality Audits** — Lighthouse, accessibility, cross-browser, performance budget
6. **Final Review & Deploy** — Full test suite, staging, production

## Installation

Install a specific skill globally:

```bash
npx skills add saccoai/agent-skills@website-refactor -g
```

Or install all skills:

```bash
npx skills add saccoai/agent-skills --all -g
```

## License

MIT
