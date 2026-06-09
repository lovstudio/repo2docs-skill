# lovstudio-repo2docs

![Version](https://img.shields.io/badge/version-0.1.0-CC785C)

Generate a professional **Fumadocs** documentation website from a product's
GitHub or local code repository, and deploy it to
`https://{product-id}.lovstudio.ai/docs`.

Part of [lovstudio general skills](https://github.com/lovstudio/general-skills) — by [lovstudio.ai](https://lovstudio.ai)

## What it does

```
 source repo  ──▶  read code/README/examples  ──▶  author MDX
 (URL | local)                                        │
                                                      ▼
                                          scaffold Fumadocs (Next.js)
                                          basePath = /docs
                                                      │
                                                      ▼
                                  deploy → {product-id}.lovstudio.ai/docs
                                  (via lovstudio-deploy-to-vercel)
```

Unlike boilerplate generators, the agent actually reads the source — README,
examples, public API, doc comments — and writes real pages (overview, getting
started, guides, API reference), not placeholder text.

## Install

```bash
git clone https://github.com/lovstudio/repo2docs-skill "${LOVSTUDIO_SKILLS_INSTALL_DIR:?Set LOVSTUDIO_SKILLS_INSTALL_DIR}/lovstudio-repo2docs"
```

Requires:
- Node.js 18+, `npx`, `git`
- Vercel CLI (`npm i -g vercel`) for deployment
- The [`lovstudio-deploy-to-vercel`](https://github.com/lovstudio/deploy-to-vercel-skill)
  skill (handles Vercel + Cloudflare DNS for the subdomain)

## Usage

Trigger it in any Claude Code session:

> 用 fumadocs 给 github.com/acme/widget 生成文档站，部署到 widget.lovstudio.ai/docs

The skill will ask for the source, product id, title, and whether to deploy, then
scaffold + author + (optionally) ship.

### Scaffold step directly

```bash
SKILL_DIR="${LOVSTUDIO_SKILLS_INSTALL_DIR:?}/lovstudio-repo2docs"
python3 "$SKILL_DIR/scripts/scaffold_docs.py" \
  --product-id widget \
  --out ./widget-docs \
  --title "Widget" \
  --pm pnpm
```

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `--product-id` | (required) | Slug → `{id}.lovstudio.ai` |
| `--out` | (required) | Output dir for the docs site (kept separate from source) |
| `--title` | `=product-id` | Human-facing product title |
| `--pm` | `pnpm` | Package manager (pnpm / npm / yarn / bun) |

## How the `/docs` path works

The site is served under `/docs` via Next.js `basePath: '/docs'`, which the
scaffold script sets automatically. See
[`references/fumadocs.md`](references/fumadocs.md) for the routing details and the
common double-prefix gotcha.

## License

MIT
