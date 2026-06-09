---
name: lovstudio-repo2docs
description: >
  Generate a professional, polished documentation website for a product from its
  GitHub or local code repository using Fumadocs (Next.js), then deploy it to
  https://{product-id}.lovstudio.ai/docs. Reads the source code, README, examples
  and comments to author real MDX docs (overview, getting started, guides, API
  reference), scaffolds a Fumadocs site under the /docs base path, and ships it to
  Vercel with the subdomain bound. Trigger when the user wants to "generate docs",
  "build a docs site", "make a documentation website", "fumadocs", "为项目生成文档",
  "做文档站", "生成文档网站", or points at a repo and asks for a docs site at
  *.lovstudio.ai/docs.
license: MIT
compatibility: >
  Portable Agent Skills format. Requires Node.js 18+, npx, git, and the Vercel
  CLI for deployment. The deploy step delegates to the lovstudio-deploy-to-vercel
  skill (Cloudflare DNS auto-config needs CLOUDFLARE_API_KEY). The {product-id}
  subdomain and lovstudio.ai base domain are user-configurable.
depends_on:
  - lovstudio-deploy-to-vercel
metadata:
  author: lovstudio
  version: "0.1.0"
  tags: docs fumadocs documentation nextjs vercel codebase
---

# repo2docs — Codebase → Polished Docs Site

Turn a product's code repository into a professional Fumadocs documentation
website and deploy it to `https://{product-id}.lovstudio.ai/docs`.

The skill does two jobs:
1. **Author** — read the source repo (code, README, examples, comments) and write
   real MDX pages, not boilerplate.
2. **Ship** — scaffold a Fumadocs (Next.js) app served under `/docs`, then deploy
   to Vercel with the subdomain bound.

## User Configuration

Defaults are portable. The base domain is `lovstudio.ai`; the subdomain is the
product id. Override via CLI/answer when the product lives on another domain.
Deploy delegates to `lovstudio-deploy-to-vercel`, which reads `CLOUDFLARE_API_KEY`
for DNS. See `references/user-config.md`.

## When to Use

- "用 fumadocs 给这个项目生成文档站并部署到 xxx.lovstudio.ai/docs"
- User points at a GitHub URL or local repo and wants a documentation website
- A product needs polished docs (overview / quickstart / guides / API reference)

## Workflow (MANDATORY — follow in order)

### Step 0: Resolve skill root

```bash
export SKILL_DIR="${SKILL_DIR:-$(pwd)}"   # or the installed lovstudio-repo2docs dir
```

If any network command times out in the sandbox, export the proxy once:

```bash
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7891
```

### Step 1: Collect inputs with AskUserQuestion

**IMPORTANT: Use `AskUserQuestion` BEFORE doing anything**, unless the user
already gave all of these:

- **Source**: GitHub URL, local path, or current directory?
- **Product id**: the subdomain → `{product-id}.lovstudio.ai`. Propose a slug
  from the repo name; confirm.
- **Title**: human-facing product name for the docs.
- **Deploy now?**: deploy to Vercel + bind subdomain immediately, or generate only.

### Step 2: Resolve the source repo

| Source | Action |
|--------|--------|
| GitHub URL | `git clone --depth 1 <url> <tmp>/src` |
| Local path | use as-is (read-only) |
| Current dir | use `$(pwd)` |

Never write into the source repo. The docs site is a **separate** directory.

### Step 3: Understand the product (this is the real work)

Read enough to write accurate docs. Prioritize:

1. `README*`, `docs/`, `CHANGELOG*`, `examples/`, `LICENSE`
2. `package.json` / `pyproject.toml` / `Cargo.toml` / `go.mod` — name, scripts,
   entry points, deps, bin
3. Public API surface — exported functions, CLI commands, routes, config schema
4. Inline doc comments / docstrings

Build a mental outline. A good default information architecture:

```
Introduction        (what it is, who it's for, key features)
Getting Started     (install, prerequisites, first run, hello-world)
Guides/             (task-oriented how-tos for the main use cases)
API Reference/      (functions / CLI / endpoints / config — derived from code)
FAQ / Troubleshooting
```

Adapt to the product: a CLI tool, a library, and a web service each need a
different shape. Do NOT invent features that aren't in the code.

For large repos, consider delegating Step 3 to a subagent (Explore /
general-purpose) to map the codebase and return a docs outline, so the main
context stays focused on authoring.

### Step 4: Scaffold the Fumadocs site

```bash
python3 "$SKILL_DIR/scripts/scaffold_docs.py" \
  --product-id "<id>" \
  --out "<docs-out-dir>" \
  --title "<Title>" \
  --pm pnpm
```

This runs `create-fumadocs-app`, sets `basePath: '/docs'` in `next.config.mjs`,
and resets `content/docs/` to a single placeholder. See
`references/fumadocs.md` for the directory layout and config details.

### Step 5: Author the MDX pages

Replace the placeholder. Write into `content/docs/` (or `src/content/docs/` with
`--src`). For each page:

- Front-matter: `title`, `description`.
- Real content derived from Step 3 — never lorem ipsum.
- Use Fumadocs components (`<Cards>`, `<Tabs>`, `<Steps>`, `<Callout>`,
  `<CodeBlock>`) for a polished feel. See `references/fumadocs.md`.
- Maintain `meta.json` in each folder to control sidebar order.

Quality bar: this is a **professional, polished** site. Coherent IA, working
internal links, real code samples copied/adapted from the repo, an inviting
landing page.

### Step 6: Verify the build locally

```bash
cd "<docs-out-dir>"
pnpm build        # must succeed — fix MDX/link errors before deploying
```

### Step 7: Deploy (if requested)

Delegate to the `lovstudio-deploy-to-vercel` skill — do NOT reimplement Vercel
or DNS logic here.

```bash
cd "<docs-out-dir>"
# Ensure package.json "name" is a valid lowercase slug, then:
# invoke lovstudio-deploy-to-vercel with domain {product-id}.lovstudio.ai
```

The site serves at `https://{product-id}.lovstudio.ai/docs` because of the
`/docs` basePath. After deploy, **verify the live URL returns 200** (curl), not
just that the alias was set.

## CLI Reference

`scripts/scaffold_docs.py`:

| Argument | Default | Description |
|----------|---------|-------------|
| `--product-id` | (required) | Slug → `{id}.lovstudio.ai` |
| `--out` | (required) | Output dir for the docs site (separate from source) |
| `--title` | `=product-id` | Human-facing product title |
| `--pm` | `pnpm` | Package manager (pnpm/npm/yarn/bun) |

## Dependencies

- Node.js 18+, `npx`, `git`
- Vercel CLI (`npm i -g vercel`) for deploy
- `lovstudio-deploy-to-vercel` skill (handles Vercel + Cloudflare DNS)

For Fumadocs structure, components, and config, see `references/fumadocs.md`.
