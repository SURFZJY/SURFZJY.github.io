# Jiyuan Zhang

Personal academic homepage, built with [al-folio](https://github.com/alshedivat/al-folio) and published at [https://surfzjy.github.io](https://surfzjy.github.io).

The site source lives on `main`. GitHub Actions builds it and deploys the result to the `gh-pages` branch. In the repository settings, GitHub Pages should publish from the `gh-pages` branch (root), not from `main`.

```bash
bundle install
bundle exec jekyll serve
```

The local server uses the empty `baseurl` in `_config.yml`, so the site is at [http://localhost:4000/](http://localhost:4000/).
