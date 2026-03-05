# AGENTS.md

## Cursor Cloud specific instructions

This is a **Jekyll static site** (Picaro.dev — personal blog with neon/hacker theme). There is no backend, no database, no tests, and no linting configured.

### Running the dev server

```bash
jekyll serve --host 0.0.0.0 --port 4000
```

The site is then available at `http://localhost:4000/`.

### Caveats

- **Config filename**: The Jekyll config file is named `_config,yml` (comma, not dot). Jekyll's `--config` flag treats commas as separators, so the config is **not loaded** by any standard Jekyll command. The site builds with Jekyll defaults. Do not attempt to pass `--config "_config,yml"` as it will be misinterpreted as two files.
- **Post filename**: The test post at `_posts/test-post.md` does not follow Jekyll's required `YYYY-MM-DD-title.md` naming convention, so it is not included in `site.posts`. New posts must follow the standard naming convention to appear.
- **No Gemfile**: There is no `Gemfile` in the repo. Jekyll and Bundler are installed globally via `gem install`.
- **No automated tests or linting**: The project has no test suite or linter configuration.
