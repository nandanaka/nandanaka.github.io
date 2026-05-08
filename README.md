# nandanaka.github.io

Personal site for Nandana K A — built with [Astro](https://astro.build/), deployed to GitHub Pages.

## Develop

```bash
npm install
npm run dev      # http://localhost:4321
npm run build    # static output to ./dist
npm run preview  # serve the built site
```

## Structure

- `src/pages/index.astro` — home page (resume content, intro, project cards)
- `src/pages/projects/[...slug].astro` — project deep-dive layout
- `src/content/projects/*.md` — one Markdown file per deep-dive
- `src/content.config.ts` — content-collection schema for projects
- `src/layouts/BaseLayout.astro` — HTML shell, meta tags, global styles
- `src/styles/global.css` — single stylesheet (system fonts, light/dark)
- `public/` — static assets served at the root (favicon, `resume.pdf`)

## Deploy

`.github/workflows/deploy.yml` builds and deploys on push to `main` via
[`withastro/action`](https://github.com/withastro/action) +
[`actions/deploy-pages`](https://github.com/actions/deploy-pages).

After creating the GitHub repo `nandanaka/nandanaka.github.io`:

1. Enable Pages: **Settings → Pages → Source: GitHub Actions**.
2. Push to `main` — the workflow handles build + deploy.

## Adding a project deep-dive

Drop a new `.md` file in `src/content/projects/`. Frontmatter schema is in
`src/content.config.ts`. Set `draft: true` to hide a page from the home page and
URL routes; set `order` (lower = earlier) to control card sorting.

## Resume PDF

Place the latest compiled PDF at `public/resume.pdf`. The home page links to
`/resume.pdf`. Keep this in sync with the LaTeX source in the sibling `resume/` repo.
