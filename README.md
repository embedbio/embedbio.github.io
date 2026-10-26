# embedbio.github.io

Website for Embed Bioinformatics LLC, published at https://embedbio.github.io by GitHub Pages.
The homepage is `index.html`; the blog is built by GitHub Pages' built-in Jekyll.

## Adding a blog post

1. Create `_posts/YYYY-MM-DD-short-title.md`:

   ```markdown
   ---
   layout: post
   title: "Your post title"
   date: 2026-10-26
   author: "Amit Indap"
   tags:
     - Nextflow
     - Genomics
   ---

   First paragraph. This becomes the excerpt on the blog index.

   The rest of the post, in GitHub-flavored Markdown.
   ```

2. Put any images in `assets/img/` and reference them as `![alt text](/assets/img/figure.png)`.
3. Commit and push to `main`. The post appears at `/blog/YYYY/MM/DD/short-title/` within a minute or two.

## Previewing locally (optional)

```bash
gem install bundler
echo 'source "https://rubygems.org"
gem "github-pages", group: :jekyll_plugins' > Gemfile
bundle install
bundle exec jekyll serve   # http://localhost:4000
```

`Gemfile`, `Gemfile.lock`, and `_site/` are only for local preview and don't need to be committed.

## Layout

- `index.html` — homepage (plain HTML, not processed by Jekyll)
- `assets/css/site.css` — styles shared by every page
- `_layouts/default.html` — header and footer for blog pages (keep its nav in sync with `index.html`)
- `_layouts/post.html` — a single post
- `blog/index.html` — the list of posts
