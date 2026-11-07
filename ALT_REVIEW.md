# Alternative design: signature

Branch `alt-signature` in `/var/tmp/website/alt-signature`, based on live `master` at commit `7f1bc59`. Nothing has been pushed. The live design stays the default.

## The direction

The current minimal layout stays exactly as it is: same system fonts, the same one-column measure and accent blue, the same header, bio, News and Research. I added one visual identity element taken from the research, and used it again as the placeholder on the publications page.

**Hero figure: a multiple randomization design.** Under the name header, an inline SVG shows a grid of 5 buyers (rows) × 36 sellers (columns). Hugo generates it at build time in `layouts/_partials/mrd-grid.html`.
- Each buyer and each seller is assigned to treatment or control. The dots along the left and top edges show these assignments: filled for treatment, hollow for control.
- Each cell is coloured by its buyer–seller pair: darkest when both are treated, mid-tone when only the buyer is, light when only the seller is, and grey when neither is. This is the simple MRD structure from the JRSS-B paper, and it produces a recognisable plaid pattern.
- The figure uses only the site's own `--accent` and `--rule` colours, through CSS classes, so it follows light and dark mode automatically.
- Without JS, the build-time assignment is deterministic: a hash of a seed, giving 2 of 5 buyers and 18 of 36 sellers treated.
- With JS, a 707-byte inline script draws a fresh *complete* randomization on each load: exactly half of the buyers (rounded up) and half of the sellers are treated. It also reveals a small **Re-randomize** button. When you click it, the cells fade to the new assignment. The fade is turned off under `prefers-reduced-motion`.
- On phones (≤ 30 rem wide), the grid is shown at 175 % width inside a wrapper that clips it and fades out on the right. You see about 20 larger sellers instead of 36 tiny ones.
- A one-line caption explains the figure and links to the JRSS-B paper page.

**Publication thumbnails.** Every entry on `/publication/` now has a 3:2 thumbnail on its left, 120 × 80 CSS px (4.75 rem wide on phones).
- 13 papers have an arXiv PDF. For each, I cropped a figure from the PDF into `static/img/pubs/<slug>.webp`: 240 × 160 px (2×), 1.4–5.5 KB each, with white padding so the whole figure fits.
- The source page and alt text for each thumbnail are recorded in `data/thumbs.yaml`. The template reads that file, so adding or removing a thumbnail is a one-line change.
- The other 9 papers get the same buyers × sellers grid as a placeholder, at 5 × 8, seeded by the paper's slug. Each placeholder is a little different, and none is a broken image.
- In dark mode, the white figure thumbnails are dimmed slightly (`brightness(0.86)`) so they don't glare.
- I left the Research list on the homepage without thumbnails, to keep the homepage as minimal as it is now.

Files changed: `layouts/home.html`, `layouts/publication/list.html`, `layouts/_partials/mrd-grid.html` (new), `assets/css/main.css`, `data/thumbs.yaml` (new), `static/img/pubs/*.webp` (new). Content files (`content/`, `data/news.yaml`), `config.toml` and `netlify.toml` are unchanged.

## Thumbnail sources

All PDFs were downloaded from arXiv into `/var/tmp/website/alt-dl/`. Figures were extracted only with `pdftoppm` and cropped with Pillow; nothing from the downloads was executed. Every figure comes from the paper's own arXiv version, which is co-authored work. **No journal-only versions were used.**

| Publication (slug) | arXiv PDF | PDF page | Figure |
|---|---|---|---|
| agarwal-2024-bandits | 2405.18621 | 2 | network-interference diagram (Fig. 1) |
| bajari-2021-multiple | 2112.13495 (current version) | 20 | histograms of the SRD vs. MRD estimators |
| beraha-2026-online | 2505.19643 | 5 | cumulative new users over time |
| hut-2026-simulate | 2608.02345 | 6 | first two panels of the subsampling figure |
| imbens-2026-demonstration | 2603.06941 | 25 | multi-scale power curves, 10 arms |
| masoero-2021-bayesian | 2112.02032 | 4 | top row of the singletons / power figure |
| masoero-2022-cross | 2202.01910 | 17 | local replicability of k-means clusters |
| masoero-2022-more | 1912.05516 | 12 | first two TCGA panels (Glioblastoma, Ovarian) |
| masoero-2023-leveraging | 2305.01109 | 9 | DIM vs. LR estimates against imbalance |
| masoero-2024-multiple | **2401.01264v1** | 15 | estimator and variance-estimator histograms |
| missault-2024-switchback | 2506.12654 | 3 | SE of the ATE under three designs (Fig. 1) |
| nguyen-2020-independent | 2009.10780 | 23 | image-denoising quality panels (a)–(c) |
| sudijono-2026-regression | 2603.19480 | 5 | KDE of three direct-effect estimators |

Placeholders (no arXiv link on the site): bajari-2023-edm, camerlenghi-2022-scaled, gualavisy-2025-power, horel-21, masoero-2018-posterior, masoero-2024-marketplace, masoero-2025-bayesian, masoero-2026-mind, shen-2022-double.

## Screenshots

New, in `/var/tmp/website/alt-shots/signature/`:
- `home_desktop.png`, `home_mobile.png`, `home_desktop_dark.png`
- `pubs_desktop.png`, `pubs_mobile.png`, `pubs_desktop_dark.png` (extra)

Live, for comparison, in `/var/tmp/website/redesign-shots/`: `home_desktop.png`, `home_mobile.png`, `home_desktop_dark.png`, `pubs_desktop.png`, `pubs_mobile.png`. They appear to predate the last two content commits (`31c444b` News: RSS 2026 and `7f1bc59` Research order), so their News and Research sections differ slightly. That is a content difference, not a design one.

The screenshots of the new design were taken with JS on, so they show one random draw. I checked the no-JS grid from the built HTML: 2 of 5 buyers and 18 of 36 sellers treated.

## Verification

- **Build:** Hugo 0.169.0, `hugo --gc --minify`, 0 errors and 0 warnings.
- **Internal links:** a script checked every internal `href`/`src` and CSS `url()` in every HTML and CSS file in `public/`. **223 references, 0 broken.**
- **Live URLs:** I built live `master` (`7f1bc59`) in a temporary `git worktree`, then removed it with `git worktree remove --force`. All 58 files of the live build exist in this build, except the old fingerprinted stylesheet name (`css/main.min.bf50….css`, which is expected because the CSS changed). That covers `/`, `/publication/`, all 22 `/publication/<slug>/` pages, all 12 `cite.bib` files, `/files/*` and `/img/*`.
- **External requests at page load:** none. No fonts, CDNs or analytics. The only absolute URLs are the `rel=canonical` links, the same as live.
- **JS:** one inline script, 707 bytes, on the homepage only. The page is complete without it.
- **Accessibility:**
  - The hero SVG has `role="img"` and a descriptive `aria-label`. The placeholder SVGs are `aria-hidden`. Every thumbnail `<img>` has descriptive alt text from `data/thumbs.yaml`.
  - The new button has a visible `:focus-visible` outline, the same as the links.
  - The caption text uses `--muted`: #595959 on white is about 7:1, and #a6a6a6 on #15171a is about 8:1, both above WCAG AA.
  - The grid is decorative, so only its marker dots carry meaning. The cell tones are not needed to understand the page.
- **Page weight:**

| Page | Live | Signature |
|---|---|---|
| Home: HTML + CSS + portrait + favicon, raw | 19.5 KB | 36.5 KB |
| Home, with HTML/CSS gzipped | ≈ 13.6 KB | ≈ 15.7 KB |
| Publications, raw (incl. 13 thumbnails) | 16.1 KB | 101.0 KB |
| Publications, with HTML/CSS gzipped | ≈ 5.1 KB | ≈ 56.7 KB |

  Most of the homepage increase is the inline SVG of about 200 small `<rect>`s. It compresses to about 2 KB. On the publications page, about 50 KB is the 13 WebP thumbnails, which are lazy-loaded. The rest is the 9 inline placeholder grids, which compress very well.

## Worth your decision

1. **The arXiv link for the JRSS-B paper points at a withdrawn preprint.** `content/publication/masoero-2024-multiple/index.md` sets `url_preprint = "https://arxiv.org/abs/2401.01264"`. arXiv now says: "This paper has been withdrawn by Lorenzo Masoero. This work has been merged with and superseded by arXiv:2112.13495". I did not change any content. You may want to point the arXiv button at 2112.13495 instead.
2. **Its thumbnail comes from the withdrawn v1** (2401.01264v1, p. 15). This is your own preprint, but if you prefer a non-withdrawn source, the same kind of figure is in 2112.13495 around p. 19. Related: `bajari-2021-multiple` (arXiv 2112.13495) and the JRSS-B entry are now the same work on arXiv, so both list entries draw from it.
3. **Caption and button are new UI copy:** "A multiple randomization design: buyers (rows) × sellers (columns), each randomized to treatment or control. Read the paper · Re-randomize". Delete the `<figcaption>` in `layouts/home.html` if you want the figure without words.
4. **Random on every load vs. fixed.** If a different pattern on each visit feels too playful, delete the `<script>` block in `layouts/home.html`. The deterministic build-time grid then stays. Change `"seed" "masoero"` there to get a different fixed pattern.
5. **Thumbnails are small (120 × 80).** Figure details aren't legible at that size; they act as visual anchors. To make them larger, change `7.5rem` in `.pubs > li` in `assets/css/main.css` (the images are 2×, so up to about 120 px wide stays sharp).
6. **Crop choices:** some crops cut slightly into axis labels; for example, the "Item-randomization" label on the switchback thumbnail is tight. To redo a crop, render the page listed above with `pdftoppm`.
7. **No CSS entrance animation.** I first tried a staggered fade-in of the grid. In headless Chromium it sometimes left the cells invisible at screenshot time, so I removed it to make sure the page always looks complete. Only the fade on re-randomization remains, and it is turned off under reduced motion.

## Preview

```sh
cd /var/tmp/website/alt-signature && /var/tmp/hugo-new/hugo server --bind 127.0.0.1 --port 1313
```

To view it from a laptop, open an SSH tunnel to that port, e.g. `ssh -L 1313:127.0.0.1:1313 <cloud-desktop>`, then go to http://localhost:1313.

## Adopting it (optional)

The live design stays the default. If you choose this one, it can be published with:

```sh
cd /var/tmp/website/alt-signature && git push origin alt-signature:master
```

Lorenzo Masoero (masoerl), last updated 2026-11-07. Prepared with Claude.
