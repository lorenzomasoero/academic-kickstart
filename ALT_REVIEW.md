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

## Rotating figures

Added on branch `alt-signature` in commits `65d627b` and `7fc9ad3` (nothing pushed). Each homepage load now shows one schematic figure under the name header, chosen at random from eight, one per line of research. The MRD grid is still the figure shown without JS. Bio, News, Research and the publications pages are unchanged.

### The figures

The captions read "<description> — <title link> (<venue>)". All are labelled as illustrations, and none draws numbers from a paper.

| id | Figure | Caption | Link |
|---|---|---|---|
| `mrd` | buyers × sellers grid (unchanged; re-randomized each time it is shown) | Illustration: buyers (rows) × sellers (columns), each randomized to treatment or control — Multiple randomization designs (JRSS-B) | `/publication/masoero-2024-multiple/` |
| `switchback` | 5 units × 36 periods; each unit alternates in blocks of 6 with its own phase | Illustration: units (rows) switch between treatment and control in blocks over time, each on its own randomized schedule — Multiple-unit switchback experiments (preprint) | `/publication/missault-2024-switchback/` |
| `more-for-less` | discovery curve: solid observed part, dashed extrapolation in a widening band; axis labels "new variants" and "samples" only | Illustration: new genomic variants found as the sample grows, extrapolated past the samples at hand with widening uncertainty — More for less (Biometrika) | `/publication/masoero-2022-more/` |
| `ibp` | 10 users × 66 days, left-ordered staircase; first visit solid, repeats lighter | Illustration: users (rows) by days (columns); each row starts on a user's first active day, later marks are repeat visits — Online activity prediction (Annals of Applied Statistics) | `/publication/beraha-2026-online/` |
| `triagent` | loop: tickets → triage agent → answer → gap flagged → docs updated → agent, with a moving packet | Illustration: an agent answers support tickets, flags gaps in the docs, and the updated docs improve the next answer — Mind the Gap (IAAI 2027) | `/publication/masoero-2026-mind/` |
| `agents-ab` | agent dots in control (hollow) and treatment (filled), an outcome bar for each, and the simulated vs. real effect as two intervals | Illustration: agents split into control and treatment, and the simulated effect checked against the real one — Can AI agents simulate A/B test outcomes? (WAB@COLM, CODE@MIT 2026) | `/publication/hut-2026-simulate/` |
| `bandits` | 34-node network; nodes solid / light / hollow by arm, edges for interference | Illustration: units on a network, each shaded by the arm chosen for it; edges link units that affect each other — Multi-armed bandits with network interference (NeurIPS 2024) | `/publication/agarwal-2024-bandits/` |
| `early-stop` | posterior-probability path crossing a dashed threshold; the rest of the planned run is shaded "time saved" | Illustration: the probability that treatment is better, tracked over time; the test stops once it crosses the threshold — Bayesian early termination (CODE@MIT 2025) | `/publication/masoero-2025-bayesian/` |

### How it works

- `data/hero.yaml` lists the figures: `id`, `page`, `title`, `venue`, `caption`, and an optional `clip`. `layouts/home.html` renders every figure into the HTML, each as its own `<figure>`. All but the first have `hidden`.
- Each figure is one partial, `layouts/_partials/hero/<id>.html`. Each is an inline SVG with the grid's 662 × 104 viewBox, so the space it takes never changes. It is coloured only through shared CSS classes in `assets/css/main.css` ("Rotating hero figures").
- The `mrd`, `switchback` and `ibp` grids are generated by Hugo at build time. Switchback and IBP draw their cells as one `<path>` per tone, which keeps them small. The curves and the network use fixed coordinates, computed once with a throwaway script.
- An inline script (1,133 bytes, replacing the 707-byte one) does three things:
  - picks a random figure, or the one named by `?fig=<id>`;
  - moves the **Show another** button (formerly Re-randomize) into the visible caption, keeping its focus;
  - cycles to a different figure on each click. Showing `mrd` draws a fresh complete randomization, as before.
- I checked 30 consecutive clicks in headless Chromium: never the same figure twice in a row, and always exactly one figure visible.
- Captions reserve two lines on desktop and four on phones. The bio starts at the same height for every figure (432 px at 1440 wide, 475 px at 390 wide).
- **Motion:** only the `triagent` packet moves, using a CSS motion path. It is hidden in browsers without `offset-path` support. Under `prefers-reduced-motion` it stays still, on the agent node. The MRD fade is turned off as before.
- **Phones (≤ 30 rem):**
  - Figures with `clip: true` (`mrd`, `switchback`, `ibp`, `bandits`) use the grid's 175 % slice that fades out on the right.
  - The diagrams (`more-for-less`, `triagent`, `agents-ab`, `early-stop`) are shown whole, with labels at about 11 px and thicker lines, in a box as tall as the grid slice.

### Adding a figure

1. Create `layouts/_partials/hero/<id>.html` containing `<svg class="hfig" viewBox="0 0 662 104" role="img" aria-label="…">`. Use the existing classes: `.a`, `.a2`, `.on`, `.off`, `.rep`, `.ring`, `.ln`, `.dash`, `.dot`, `.ax`, `.thr`, `.band`, `.edge`, `.track`, `.chev`, plus plain `<text>`. Keep labels at least ~110 units apart, so they still fit when phones enlarge them.
2. Add an entry to `data/hero.yaml`. Add `clip: true` if the figure is a texture that can be cropped on phones.
3. Optional: add new classes to `assets/css/main.css`. The script needs no change.

The first entry in `data/hero.yaml` is the figure shown without JS. A missing `page` stops the build with an error.

### Screenshots

All are in `/var/tmp/website/alt-shots/signature-rotating/`, taken through `?fig=<id>`:
- `<id>_desktop.png`: all eight figures, 1440 wide, light mode
- `<id>_mobile.png`: all eight at 390 wide (the brief asked for two)
- `<id>_desktop_dark.png`: all eight in dark mode (the brief asked for two)
- `nojs_desktop.png`: the page with the script removed, showing the build-time MRD grid

### Verification and page weight

- Hugo 0.169.0, `--gc --minify`: 0 errors and 0 warnings.
- A script checked every internal `href`/`src`/`url()` in `public/`: 230 references, 0 broken. All eight caption links resolve.
- No external requests and no libraries.

| Homepage | Signature (`4b711f0`) | With rotating figures |
|---|---|---|
| HTML + CSS + inline JS, raw | 26.3 KB | 44.3 KB |
| the same, gzipped | 6.0 KB | 10.2 KB |
| plus portrait and favicon (as in the table above) | 15.3 KB* | 19.5 KB |

\*The earlier table reports 15.7 KB for the signature design; this re-measurement of commit `4b711f0` with `gzip -9` gives 15.3 KB. The increase is about +4.2 KB gzipped, mostly from the IBP matrix and the network. The JS is 1.1 KB.

The live preview was rebuilt with `hugo --quiet -d /var/tmp/website/preview/signature`, and http://127.0.0.1:1313 is serving it.

### Dropped or unsure

- **Nothing dropped.** All eight read at this size. The weakest are:
  - `ibp`: the staircase is clear, but the repeat visits make it busy.
  - `bandits`: "interference" is carried by the caption more than by the drawing.
- **Colours:** labels and axes use `--muted` and `--rule`, and marks use `--accent`. The brief said `--accent`/`--rule` only, but the original grid already uses `--muted` for its hollow dots. Grey text needed it.
- **Venues:** I followed the brief. Switchback shows "preprint", because the brief gives no venue. The beraha-2026 page itself says "AOAS (to appear)". The short titles are my wording.
- **Phone labels** are about 11 px, the practical limit inside a 104-unit-tall strip. The diagrams leave some empty space above and below on phones, because the box is kept at the grid's height so nothing jumps.
- **At 360 px and narrower**, the `agents-ab` caption wraps to a fifth line, so the page below moves by one line for that figure only.
- **Older review items:** item 3 in "Worth your decision" is now superseded. The caption copy and the button name ("Show another") come from `data/hero.yaml` and `layouts/home.html`. To return to the single grid, keep only the `mrd` entry in `data/hero.yaml`.

## More snapshots

Added on branch `alt-signature` in commits `b8fb845`, `bd2d38c`, `3398fb5`, `de82186` and `e7e199a` (nothing pushed). The homepage now picks from 21 snapshots instead of 8 (20 from this round, plus `cpd-data`, added afterwards; see the last section), so every paper has one except `bajari-2021-multiple`. Each new figure is one partial in `layouts/_partials/hero/<id>.html` and one entry in `data/hero.yaml`, in the same 662 × 104 viewBox, palette, stroke weights and 12 px labels as the first eight. The script is unchanged, and the MRD grid is still the figure shown without JS.

### Figures added

The header shows "<title link> (<venue>)". Captions are listed as written in `data/hero.yaml`. All are schematic: the curves are smooth functions and the scatters are fixed pseudo-random draws. Nothing comes from a paper's numbers.

| id | Figure | Caption | Header: title (venue) → link |
|---|---|---|---|
| `edm` | bipartite graph: 5 buyers (left) and 5 sellers (right), filled = treated, hollow = control; grey transaction curves, with the treated-buyer → control-seller curves in accent under an "interference" label | Illustration: buyers and sellers, each treated or control, joined by transactions; transactions between treated buyers and control sellers carry interference | Experimental design in marketplaces (Statistical Science) → `/publication/bajari-2023-edm/` |
| `spillover` | 7 clusters of 7 units in thin rings; 2 treated clusters (solid) sit in soft radial halos that reach their neighbours (light); one cluster far from both stays hollow. Labels: "direct", "indirect", "no spillover" | Illustration: treated clusters of units feel a direct effect, and the effect spills over to neighbouring clusters as an indirect one | Measuring direct and indirect impacts (TSMO@KDD, CODE@MIT 2024) → `/publication/masoero-2024-marketplace/` |
| `double-rand-reg` | scatter of outcome against covariate, treated (filled) and control (hollow), with two parallel fitted lines; on the right, two I-beam intervals for the effect, "before" (wide) and "after" (narrow) | Illustration: outcomes against a covariate for treated and control units; adjusting for the covariate narrows the interval for the effect | Regression adjustments for double randomization (arXiv 2026) → `/publication/sudijono-2026-regression/` |
| `cuped` | two equal-area normal curves on one axis: wide and dashed ("unadjusted"), narrow and solid ("adjusted"), with a dashed "effect" line through both centres | Illustration: the spread of an effect estimate with and without covariate adjustment; both are centred on the same effect | Covariate adjustment at scale (KDD 2023 workshop) → `/publication/masoero-2023-leveraging/` |
| `power` | power curve rising with effect size (solid), prior over effect sizes (dashed), and the prior × power area shaded as "expected power" | Illustration: power rises with the effect size; weighting it by a prior belief about effect sizes gives the expected power | Statistical power calculations revisited (CODE@MIT 2025) → `/publication/gualavisy-2025-power/` |
| `demo` | 100 % stacked bands for three arms over time. They start even and shift to the bottom arm, up to a dashed threshold with a check mark and "demonstrated" | Illustration: the share of units given to each of three arms over time, shifting towards the arm that works until it is demonstrated | Demonstration experiments (arXiv 2026) → `/publication/imbens-2026-demonstration/` |
| `scaled-priors` | two small panels with the same observed curve and the same later counts (hollow dots). "standard prior": narrow band, falls short. "scaled prior": wider band that covers the dots | Illustration: predicting new genetic variants; a standard prior is too confident and misses, a scaled prior's band covers what comes next | Scaled process priors (JASA) → `/publication/camerlenghi-2022-scaled/` |
| `power-rare` | a hump-shaped power curve across the budget split, with the optimum marked; small glyphs show "breadth" (many thin bars) and "depth" (few thick bars) at the two ends | Illustration: a fixed sequencing budget spent on many samples at low depth or few samples at high depth; power peaks in between | Power maximization for rare variants (LMRL@NeurIPS 2021) → `/publication/masoero-2021-bayesian/` |
| `two-pops` | two discovery curves, "population 1" (solid) and "population 2" (lighter), plus a shaded "shared" area below them, with labels at the ends of the curves | Illustration: new variants found as samples grow in two populations, and the part of them the two populations share | Double trouble (LMRL@NeurIPS 2022) → `/publication/shen-2022-double/` |
| `replicability` | "study 1" and "study 2" side by side, each with clusters (solid, light, hollow) linked by dotted lines to their match. The study-1 cluster with no counterpart is faded grey and labelled "no match" | Illustration: clusters found in two studies; matching clusters are linked, and one that does not replicate is faded | Cross-study replicability in cluster analysis (Statistical Science) → `/publication/masoero-2022-cross/` |
| `finite-approx` | two bar rows: "infinite", 58 geometrically decreasing weights that trail off, and "finite", 22 close but not identical weights ending at a dashed "truncated here" line | Illustration: the weights of infinitely many atoms (top), and a finite approximation that keeps a fixed number of them (bottom) | Independent finite approximations (Bayesian Analysis) → `/publication/nguyen-2020-independent/` |
| `trait-alloc` | a dot matrix: a pool row on top (dot size = how common the trait is), then three groups of two samples each; filled dots = traits a sample has; shaded columns = traits all three groups use. Cropped on phones like the grids | Illustration: three groups of samples (row pairs) draw traits (columns) from a shared pool on top; shaded columns are traits all groups share | Hierarchical CRMs in trait allocation (BNP@NeurIPS 2018) → `/publication/masoero-2018-posterior/` |

Each figure has its own visual idea: a bipartite graph, halos, a scatter with error bars, overlaid densities, a curve times a prior, stacked bands, small multiples, a hump with glyphs, labelled curve ends, linked clusters, bar rows and a dot matrix. None of them reuses the square-cell grid of `mrd`, `switchback` and `ibp`.

### Dropped, skipped, and unsure

- **Nothing dropped.** All twelve read at 1440 and at 390 wide. The weakest at phone size are:
  - `replicability`: the clusters become small dot groups, and the faded one is faint by design.
  - `scaled-priors`: two panels in one strip are small; the narrow band in the "standard prior" panel is thin.
- **Skipped, as the brief asked:** `bajari-2021-multiple`, the same work as the existing `mrd` snapshot.
- **Venues and titles:**
  - The short titles are my wording, e.g. "Statistical power calculations revisited" for gualavisy-2025 (the brief called it "Power with effect-size priors"), "Covariate adjustment at scale" and "Double trouble".
  - Both 2026 preprints show "arXiv 2026", as the brief gives for `demo`. Their pages say arXiv 2603.19480 and 2603.06941. The older `switchback` entry still shows "preprint"; make them match if you prefer one style.
  - Workshop venues are abbreviated as "LMRL@NeurIPS 2021/2022", "BNP@NeurIPS 2018" and "KDD 2023 workshop".
- **Small layout change on phones:** with 20 papers, some titles wrap to two lines at 390 px. The paper line under "Random research snapshot" now reserves two lines on phones (`.hf-paper { min-height: 2.8em }`), so short titles leave one empty line. In exchange, the bio starts at the same height for all 20 figures. I measured it in headless Chromium: 483 px at 1440 wide and 545 px at 390 wide, with every caption inside its reserved 2 (desktop) or 4 (phone) lines.
- **New CSS** (all in the "Rotating hero figures" block of `assets/css/main.css`; colours only through `--accent`, `--rule`, `--muted` and `--bg`):
  - `.hl` accent edges, `.ln2` lighter lines, `.link` dotted links, `.bar` glyph bars, `.fade` faded markers, `.ck` the check mark (in `--bg`), and `text.ta` accent labels;
  - `.g0`/`.g1` gradient stops for the `spillover` halos;
  - `.dt`, compact dots: one `<path>` of zero-length round-capped strokes instead of one `<circle>` per dot. This took `trait-alloc` from 10.2 KB to 3.9 KB;
  - a 13 px label size for cropped (`clip: true`) figures on phones. They are already drawn 1.75× larger there, so labels in the visible part come out at about 11 px.
- **Motion:** none added. Only the `triagent` packet moves, as before.
- The coordinates were computed once with a throwaway script, which is not in the repo, as for the first figures. Only `ibp` (Hugo template) and `mrd` are generated at build time.

### `ibp` and `bandits`

- **`ibp`** (was too busy): the matrix is now 8 users × 54 days instead of 10 × 66, with larger cells (9 units on a 12 grid, was 8 on 10). Repeat visits are sparser (16–30 % of days, was 25–60 %). A thin accent line traces the staircase of first visits, so the "new dishes" structure is the first thing you see. New caption: "Illustration: users (rows) by days (columns); each row starts on a user's first active day, which traces a staircase; later marks are repeat visits".
- **`bandits`** (interference lived in the caption): the network is the same, minus one node, but one unit is now highlighted. It has a soft halo and a ring, its edges to its four neighbours are drawn in accent, and an accent "interference" label sits above it. All of this is in the left third, so it stays visible in the phone crop. New caption: "Illustration: units on a network, each shaded by the arm chosen for it; one unit is highlighted with the links through which its neighbours' arms affect it".
- These two captions replace the ones in the "Rotating figures" table above.

### Verification and page weight

- Hugo 0.169.0, `--gc --minify`, into a fresh folder: 0 errors and 0 warnings.
- The link script checked every internal `href`/`src`/`url()`: 242 references, 0 broken. All 20 header links resolve.
- `?fig=<id>` works for all 20 ids. For each one, headless Chromium's `--dump-dom` showed exactly that figure visible and the other 19 `hidden`.
- No external requests, no libraries, and no change to the script (1.1 KB).

| Homepage (KiB) | Brief's figure | `c119fd0` (re-measured) | With 20 snapshots (`e7e199a`) |
|---|---|---|---|
| HTML + CSS + inline JS, raw | — | 46.2 | 72.4 |
| the same, gzipped | — | 10.6 | 17.7 |
| plus portrait and favicon | 19.5 | 19.9 | 27.0 |

The brief's 19.5 KB was measured before the snapshot header (`c119fd0`). The new figures add about 7 KiB gzipped. The HTML alone is 64.4 KiB raw, well under the ~120 KB point where lazy loading would pay off. So all figures stay inline, with `hidden`, and there are still no extra requests and no `<template>` indirection.

### Screenshots

All are in `/var/tmp/website/alt-shots/signature-rotating-2/`, taken through `?fig=<id>`:
- `<id>_desktop.png`: the 12 new figures plus the changed `ibp` and `bandits`, 1440 wide, light mode
- `spillover_mobile.png`, `demo_mobile.png`, `bandits_mobile.png`: 390 wide
- `cuped_desktop_dark.png`, `power_desktop_dark.png`, `ibp_desktop_dark.png`: dark mode

I also checked every new figure at 390 wide and six in dark mode while working. Those working copies are not kept.

The live preview was rebuilt with `hugo --quiet -d /var/tmp/website/preview/signature`, and http://127.0.0.1:1313 is serving it.

### Chicago Police Department dataset (`cpd-data`)

Added at your request in commit `ace6c6b` on branch `alt-signature` (nothing pushed). This replaces the earlier note on skipping `horel-21`.

- **Header:** The Chicago Police Department dataset (NeurIPS 2021, Datasets & Benchmarks) → `/publication/horel-21/`.
- **Caption:** "Illustration: separately released public records linked into one documented dataset, in which each person's records from every source are joined".
- **Figure:** the paper's contribution is data engineering, so the figure shows only that. On the left are four small tables, "complaints", "use of force", "rosters" and "awards", each released on its own. Each table's cells have their own style: filled, light, hollow and grey. Grey lines lead into a "record linkage" node (two linked rings). An arrow goes to "one linked dataset" on the right: a small relational graph where each person is a dot, joined to record squares in the matching source styles. One person is highlighted in accent with a soft halo. That person's records come from all four sources, and one record is shared with a neighbouring person.
- **Neutral by design:** no violence, no police imagery or badges, no faces, and no real names, places or numbers. People are plain dots, and the coordinates are a fixed hand-made layout.
- **Phones:** the figure is drawn whole, like the other diagrams. Four enlarged labels do not fit stacked at 390 px, so on phones the left side reads "public records" and the four table glyphs stay. This uses two new CSS classes in the "Rotating hero figures" block of `assets/css/main.css`: `.lg` is shown on desktop only, and `.sm` on phones only (not for cropped figures). The "record linkage" label sits under the node so that it clears the incoming lines at both sizes.
- **Verification:**
  - Hugo 0.169.0 (`--gc --minify`): 0 errors, 0 warnings.
  - Link script: 243 internal references, 0 broken.
  - `?fig=cpd-data`: headless Chromium's `--dump-dom` shows that figure visible and the other 20 `hidden`.
- **Page weight (homepage):** HTML + CSS + inline JS is 76.4 KiB raw and 18.4 KiB gzipped, or 27.7 KiB with the portrait and favicon. Before this figure it was 72.4 / 17.7 / 27.0 at `e7e199a`. The HTML alone is 68.2 KiB, still well under the ~120 KB point for lazy loading.
- **Screenshots** in `/var/tmp/website/alt-shots/signature-rotating-2/`:
  - `cpd-data_desktop.png`: 1440 wide, light mode
  - `cpd-data_mobile.png`: 390 wide
  - `cpd-data_desktop_dark.png`: 1440 wide, dark mode

  The first shots showed two problems, both now fixed:
  - the four labels overlapped on phones;
  - edges showed through the hollow record squares.
- **Preview:** rebuilt into `/var/tmp/website/preview/signature`. The server on http://127.0.0.1:1313 was left untouched.
- **Open point:** in light mode the grey "awards" cells use `--rule`, the same grey as the MRD grid's control cells, so they are the faintest of the four. Say if you want them stronger.

Lorenzo Masoero (masoerl), last updated 2026-11-08. Prepared with Claude.
