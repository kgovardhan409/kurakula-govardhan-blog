# Govardhan Dev Blog

Source for [kurakulagovardhan.xyz](https://kurakulagovardhan.xyz/), a personal blog built with [Hugo](https://gohugo.io/) using the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.

## Running locally

This repo uses PaperMod as a git submodule, so clone with `--recurse-submodules`:

```bash
git clone --recurse-submodules https://github.com/kgovardhan409/kurakula-govardhan-blog.git
cd kurakula-govardhan-blog
hugo server -D
```

If you already cloned without submodules, fetch them with:

```bash
git submodule update --init --recursive
```

The site will be available at `http://localhost:1313/`.

## Deployment

Pushes to `main` trigger the [`hugo.yml`](.github/workflows/hugo.yml) GitHub Actions workflow, which builds the site with Hugo and deploys it to GitHub Pages.
