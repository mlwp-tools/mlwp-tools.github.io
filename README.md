# mlwp-tools.github.io

Source for the mlwp-tools website, https://mlwp-tools.github.io. The site is built with [Quarto](https://quarto.org) from the markdown files in this repository, and deployed to GitHub Pages by the `publish` workflow on every push to `main`.

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

which serves the site locally and re-renders on every change. `quarto render` builds the full site into `_site/`.
