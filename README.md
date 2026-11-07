# lorenzomasoero.com

Personal academic website, built with [Hugo](https://gohugo.io) and deployed by Netlify from `master`
(`netlify.toml` pins the Hugo version). No theme, no JavaScript, no external fonts or CDNs.

## Layout

| Path | What it is |
|---|---|
| `config.toml` | Site title, name, role line, contact links, header menu |
| `content/home/bio.md` | Homepage biography (Markdown) |
| `content/home/research.md` | Homepage research groups (`### Heading` + link list each) |
| `data/news.yaml` | Homepage News list, newest first |
| `content/publication/<slug>/index.md` | One folder per paper; optional `cite.bib` next to it |
| `layouts/` | Page templates (`baseof.html`, `home.html`, `publication/`, `_partials/`) |
| `assets/css/main.css` | The only stylesheet |
| `static/files/`, `static/img/` | CV PDFs, posters, portrait, favicon (served as-is) |

## Editing

**News.** Edit `data/news.yaml`. Put the newest item first; `date` is shown as written, `text` is Markdown:

```yaml
- date: "Dec 2026"
  text: "[Paper title](/publication/some-slug/) accepted at Some Venue."
```

**Add a paper.** Create `content/publication/<slug>/index.md`:

```toml
+++
title = "Paper title"
date = 2026-12-01            # the year groups the publications page
authors = ["First Author", "Lorenzo Masoero"]
publication = "Full venue name"
publication_short = "Venue 2027"   # shown in the list
url_pdf = "https://..."            # optional; the title links here if set
url_preprint = "https://arxiv.org/abs/..."   # optional; labelled arXiv for arxiv.org links
abstract = ""                      # optional
+++
```

Drop a `cite.bib` in the same folder to get a BibTeX button.

## Preview

```sh
hugo server --bind 127.0.0.1
```

## License

Site content © Lorenzo Masoero. The repository started from Academic Kickstart (MIT, see `LICENSE.md`).
