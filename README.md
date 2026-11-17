# mlwp-tools.github.io

Source for the mlwp-tools website, https://mlwp-tools.github.io. The site is built with [Quarto](https://quarto.org) and deployed to GitHub Pages by the `publish` workflow on every push to `main`.

## Pages

| File | Page |
|---|---|
| `index.qmd` | Landing page: what mlwp-tools is, the three-layer design, the packages and how to get involved. Uses `page-layout: custom` with styled sections (see below). |
| `get-started.qmd` | Install and a step-by-step example: validate, load, align, compare, and the config-driven pipeline. |
| `roadmap.md` | Release plan and open decisions for the core packages. |

## Look and feel

- `theme-light.scss` and `theme-dark.scss` hold the colour palette (taken from the logo in `img/logo.png`) as Sass defaults and `--mt-*` CSS custom properties.
- `theme.scss` holds the shared rules: fonts (DM Sans, JetBrains Mono, loaded in `fonts.html`), the landing page sections, cards, code windows and buttons.
- The landing page is built from Pandoc fenced divs with these classes, so it stays markdown:
  - `.hero`, `.band` / `.band-tint` (full-width sections) and `.band-inner` (centred content),
  - `.eyebrow` (the bracketed `[ LABEL ]` above a heading), `.lead-text`, `.accent` (blue gradient text),
  - `.split` (two columns on wide screens; `.split-center`, `.split-hero`, `.split-wide-left` variants),
  - `.card-grid` (`.card-grid-3`, `.card-grid-5`) with `.mt-card` children, `.card-icon`, `.layer-num`, `.card-links`, `.badge-status` / `.badge-released`,
  - `.terminal` with a `.terminal-bar` title, wrapping a code block,
  - `.pull-quote`, `.traits-table`, `.flow-wrap` (around the mermaid diagram),
  - `.btn-cta` and `.btn-ghost` on links, e.g. `[Get started](get-started.qmd){.btn-cta}`.
- Icons are [Bootstrap Icons](https://icons.getbootstrap.com/), which Quarto bundles: `<i class="bi bi-github"></i>`.
- Quarto's Bootstrap build has no flex `.row`/`.col-*` grid, so use `.split` or `.card-grid` for columns.

## Adding or editing a page

1. Create a branch and add a markdown file at the repository root, e.g. `my-page.md`, starting with a front matter block that sets the page title:

   ```markdown
   ---
   title: "My page"
   ---

   Content goes here.
   ```

2. Link the page from the navbar by adding it under `website.navbar.left` in `_quarto.yml`.
3. Open a pull request. The `publish` workflow renders the site on the PR to check it builds, and deploys it once the PR is merged into `main`.

Small edits can also be made straight from the site through the "Edit this page" link on each page.

## Previewing locally

Install the [Quarto CLI](https://quarto.org/docs/get-started/) (or use `uvx --from quarto-cli quarto`), then run

```bash
quarto preview
```

which serves the site locally and re-renders on every change. `quarto render` builds the full site into `_site/`. Check the landing page at phone width and in dark mode (the toggle in the navbar) when changing the theme.
