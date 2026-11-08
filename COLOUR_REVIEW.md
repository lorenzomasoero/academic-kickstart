# Colour mode and "Curious? See more"

Branch `alt-colour` in `/var/tmp/website/alt-colour`, based on the live commit `9f4bfb6`. There are three commits: `2fae91f`, `5471a71` and `854445e`. Nothing was pushed. The live-site clone `/var/tmp/website/alt-signature` was not touched.

Preview: rebuilt into `/var/tmp/website/preview/colour`, which the existing server shows at http://127.0.0.1:1313/. To try Colour mode, click **Colour** in the header, or open `/?palette=colour`.

## What changed

**Feature 1: Colour mode, on demand**

- **Toggle.** A small pill sits in the header after Publications and CV. It shows a palette icon and the label "Colour". It is a `<button>` with `aria-pressed`, and the choice is saved in `localStorage` under `palette` (`colour` or `mono`). Mono is the default. In Colour mode the icon's four dots take the four hues.
- **No flash.** A ~250-byte inline script in `<head>` (`layouts/baseof.html`) sets `data-palette="colour"` on `<html>` before first paint. It reads `?palette=colour|mono` first, then the saved choice. The URL parameter wins but is not saved.
- **Without JS.** The toggle is rendered with `hidden`, and only the script reveals it. So a visitor without JS sees Mono and no dead button.
- **CSS.** Everything lives in two blocks at the end of `assets/css/main.css`: "Palette toggle" and "Colour palette".
  - The hues are variables under `:root[data-palette="colour"]`, with a dark set inside `prefers-color-scheme: dark`.
  - Mono's variables and rules are unchanged.
- **Snapshots.** The partials now carry semantic classes `.c1`–`.c4`.
  - In Colour mode each class sets `--accent` to its hue, and `--rule` to the hue's soft tint, for that element and its children. Every existing rule (`.a`, `.ring`, `.ln`, `.band`, `.off`, …) then recolours itself.
  - In Mono the classes match no rule.
  - Two helpers: `.solid` turns a light Mono tint into the full hue in Colour mode. `.oc` is a shape drawn only in Colour mode, used once in `agents-ab`.
  - The MRD grid gets direct rules, because its cell classes are set by the homepage script.
- **Rest of the page (Colour mode only):**
  - section headings (`h2`, including the year headings) in teal;
  - the News date column in vermilion;
  - link underlines turn vermilion on hover; link colour itself is unchanged;
  - publication thumbnail borders take the soft tints, cycling blue, teal, vermilion;
  - the "See more" panel's rule and labels.

**Mono is unchanged.** I compared headless screenshots of the live build (`9f4bfb6`) and this branch in Mono for `switchback`, `power` and `cpd-data` (light and dark), and for `replicability`, `trait-alloc`, `agents-ab` and `more-for-less` (light). These include every partial whose paths I split. In every case the intro, figure and caption are pixel-identical. The only differences are:

- the new toggle in the header;
- the new "Curious? See more" line below the caption.

**Feature 2: "Curious? See more"**

- **The panel.** Each snapshot's caption now ends with a native `<details>` labelled "Curious? See more", so the panel opens without JS. It contains:
  - **The idea** and **Why it matters**, from `data/more.yaml`;
  - **Read**: the paper page, plus PDF and arXiv links, taken from the paper's front matter (`url_pdf`, `url_preprint`).
- **Link labels.**
  - A `url_pdf` that points to an arXiv abstract page is labelled "arXiv".
  - A `url_preprint` identical to `url_pdf` is not repeated.
  - Otherwise the labels match the publication list ("PDF", "arXiv", "Preprint").
- **Next snapshot.** Each panel ends with "Next snapshot: <title> →". It goes to the next entry in `data/hero.yaml`, wrapping from the last to the first, and opens that figure with its panel open and focus on its summary. So a visitor can walk through all 21 snapshots in order.
  - Without JS, the link goes to `#s-<id>`. A `:target` rule, active only without JS, then shows that figure in place of the default one.
- **`?more`.** `?more` opens the panel on load, e.g. `/?fig=bandits&more`. I added it for the screenshots, and it is handy for sharing.
- **Build check.** The build stops with an error if a snapshot has no entry in `data/more.yaml`.
- **Room for "Ask more".** Each `<details>` carries `data-paper="/publication/<slug>/"`. A Hugo comment in `layouts/home.html` marks where an "Ask more" box would go: at the end of `.more-body`. Nothing is built: no form and no network call.

## Palette

Four hues, each with a soft tint. I checked them with the dataviz skill's validator (`validate_palette.js`: lightness band, chroma floor, colour-blind separation, normal-vision separation, contrast). Contrast ratios are WCAG, measured against the page background: `#ffffff` in light mode and `#15171a` in dark mode.

| Slot | Role in the figures | Light | Contrast | Dark | Contrast | Soft tint (light / dark) |
|---|---|---|---|---|---|---|
| `c1` blue | treatment, the main object, observed data | `#1d5ba8` | 6.74 | `#5294e6` | 5.76 | `#bbcee5` / `#2d496c` |
| `c2` vermilion | what the paper is about: interference, spillover, extrapolation, time saved, "before" | `#c4471c` | 4.92 | `#dc6a33` | 5.26 | `#edc8bb` / `#653824` |
| `c3` teal | control, the third arm or group, the "good" outcome | `#00806a` | 4.89 | `#1a9f80` | 5.40 | `#b2d9d2` / `#174d43` |
| `c4` violet | a fourth part: shared pool, people, a fourth arm | `#7a4fc4` | 5.61 | `#9b7be8` | 5.47 | `#d7caed` / `#4b3f6c` |

Page accents: headings `#00806a` light (4.89) and `#3fbf9f` dark (7.84). News dates `#c4471c` light (4.92) and `#e8834f` dark (6.66). Links keep Mono's `#1d5ba8` and `#8cb8f5`.

- **Text.** All four hues reach at least 4.5:1 in both modes, so they pass WCAG AA for text as well as for graphics (3:1).
- **Colour-blind safety.** `c1`–`c3` pass every check for every pair, in both modes. Worst all-pairs separation, light: colour-blind ΔE 11.1, normal vision 16.9. Dark: 10.8 and 17.5.
- **`c4` violet.** All four pass the checks for neighbouring slots (light: ΔE 11.1 colour-blind, 24.4 normal). But violet next to blue fails the all-pairs check (light: ΔE 4.8 colour-blind, 12.9 normal), which the skill also warns about. So `c4` only appears where shape or position also tells it apart from blue:
  - hollow arms against solid ones in `bandits`;
  - circles against squares in `cpd-data`;
  - a filled area against lines in `two-pops`;
  - the top row in `trait-alloc`;
  - a labelled node in `triagent`.
- **Shape still encodes the groups.** Filled/hollow/light and labels still encode the groups exactly as in Mono. Colour is a second channel.
- **Soft tints.** These are about 1.5:1 on white. They are used only where Mono already uses the grey `--rule` (about 1.3:1): control cells, strips, borders. So nothing gets fainter than it is today.

Per figure, in Colour mode:
- **mrd:** treated-buyer rows vermilion, treated-seller columns teal, cells where both are treated blue, both control grey. The buyer and seller dots match their rows and columns.
- **switchback:** treatment blue, control teal.
- **more-for-less:** observed blue; extrapolation and band vermilion.
- **ibp:** first visits and the staircase vermilion; repeat visits and strips blue.
- **triagent:** agent blue, answer violet, gap flagged vermilion, docs updated teal.
- **agents-ab:** control teal, treatment blue, simulated vermilion, real blue.
- **bandits:** arms blue, teal and violet; interference links, halo and label vermilion.
- **early-stop:** path blue, stop and time saved vermilion, threshold teal.
- **edm:** treated blue, control teal, interference vermilion.
- **spillover:** direct blue, indirect and halo vermilion, no spillover teal.
- **double-rand-reg:** treated blue, control teal, "before" vermilion, "after" blue.
- **cuped:** adjusted blue, unadjusted vermilion, effect line teal.
- **power:** power blue, prior vermilion, expected power teal.
- **demo:** winning arm and check teal; the other two arms blue and vermilion.
- **scaled-priors:** standard prior vermilion, scaled prior blue, later counts teal.
- **power-rare:** curve blue, optimum vermilion, breadth teal, depth violet.
- **two-pops:** population 1 blue, population 2 vermilion, shared violet.
- **replicability:** the three matching clusters and their links blue, vermilion and teal; "no match" stays grey.
- **finite-approx:** infinite blue; finite and truncation vermilion.
- **trait-alloc:** groups blue, vermilion and teal; pool and shared columns violet.
- **cpd-data:** sources blue, vermilion, teal and grey; people and linkage violet.

## Editing `data/more.yaml`

- **One file.** All the panel text is in `data/more.yaml`, one entry per snapshot id. The ids are the same as in `data/hero.yaml`:
  ```yaml
  bandits:
    source: arXiv 2405.18621 abstract      # not shown on the site
    idea: >-
      Two or three plain sentences …
    why: >-
      One or two sentences …
  ```
- **Writing the text.** `>-` folds the lines into one paragraph, so you can wrap freely. Plain text only. The text is escaped, so do not use HTML or Markdown.
- **Links.** These come from the paper's front matter. Fix a link there, and it updates both the publication list and the panel.
- **Adding a snapshot.** Add an entry to `data/more.yaml` too, or the build fails with "data/more.yaml: no entry for snapshot …".
- **Sources.**
  - Fourteen entries were drafted from arXiv abstracts, fetched from the arXiv API and saved in `/var/tmp/website/alt-dl/abstracts/`. Twelve came by ID (`<id>.xml`). Two came by exact-title search (`scaled-priors-search.xml` → 2106.15480, `two-pops-search.xml` → 2403.02154).
  - **Seven have no abstract** anywhere I could use: `triagent`, `early-stop`, `edm`, `spillover`, `power`, `trait-alloc` and `cpd-data`. None has an arXiv record found by title search, and their front-matter `abstract` is empty. Those seven were drafted from the title and the snapshot caption only, and say so in their `source` line. Please check them first; each "why" is a single cautious sentence.

## Screenshots

In `/var/tmp/website/alt-shots/colour/`:
- **Colour, light, 1440 wide:** `colour_light_mrd.png`, `colour_light_bandits.png`, `colour_light_demo.png`, `colour_light_replicability.png`
- **Colour, dark:** `colour_dark_cpd-data.png`, `colour_dark_spillover.png`
- **Mono:** `mono_light_switchback.png`, `mono_light_power.png`. The live build's versions are next to them, as `live_light_switchback.png` and `live_light_power.png`. They differ only in the header toggle and the "See more" line.
- **Panel open:** `panel_open_desktop_agents-ab.png` (1440 wide) and `panel_open_phone_bandits.png` (390 wide)
- **Toggle in both states:** `toggle_light_mono-top_colour-bottom.png` and `toggle_dark_mono-top_colour-bottom.png` (2× zoom; Mono on top, Colour below)

I also looked at all 21 snapshots in Colour mode (light), and at four in dark mode. After that review I made the second lines in `two-pops` and `double-rand-reg` solid, so they match their markers.

## Verification

- **Build:** Hugo 0.169.0 (`--gc --minify`), 0 errors and 0 warnings. The preview build `--quiet -d …/preview/colour` exited 0.
- **Links:** a script checked all 299 internal `href`/`src` references in 25 pages, including `#s-<id>` fragments: 0 broken. The live build has 257, also 0 broken.
- **`?fig=<id>`:** with `&more`, the dumped DOM showed exactly that figure visible, with its panel open, for 21 of 21 ids.
- **In a browser session** (headless Chromium driven over a local DevTools port):
  - the toggle flips `data-palette` and `aria-pressed`, and saves the choice;
  - the choice survives a reload and carries over to `/publication/`;
  - `?palette=mono` overrides a saved "colour";
  - "Next snapshot" shows the next figure with its panel open and focus on the summary, and wraps from `cpd-data` to `mrd`;
  - over 25 clicks, "New snapshot" always left exactly one figure visible.
- **With scripts disabled:**
  - Mono, and the toggle is hidden;
  - the MRD grid is shown, and its panel opens;
  - `/#s-power` shows the `power` snapshot.
- **Server:** `public/` was served on 127.0.0.1:8814 only, and stopped afterwards. Chromium was closed after each shot, and the DevTools port was closed after the tests.

## Page weight (homepage)

| | Live (`9f4bfb6`) | This branch |
|---|---|---|
| HTML | 68.2 KiB | 90.7 KiB |
| CSS | 8.1 KiB | 12.0 KiB |
| HTML + CSS + inline JS, raw | 76.4 KiB | 102.7 KiB |
| HTML + CSS + inline JS, gzipped | **18.4 KiB** | **25.2 KiB** |
| gzipped, plus portrait and favicon | 27.7 KiB | 34.6 KiB |

- **JS:** inline JS grew from 1,131 to 2,094 bytes (+963), well under the ~4 KB limit.
- **HTML:** most of the HTML growth is the 21 panels' text. The HTML is still under the ~120 KB point where lazy loading would be worth it.

## "Ask more": what a backend version would need

- **Server-side call.** A serverless function (one HTTP endpoint) would call an LLM service. The API key would stay in the function's environment and never reach the page.
- **Abuse protection.** Rate limiting per IP and per session, a cap on question length, and a bot check such as a proof-of-work or challenge token.
- **Cost limits.** A hard monthly spending cap on the service account, a small maximum answer length, and caching of repeated questions.
- **Grounding.** Each answer would draw only on that paper's abstract and text. The `data-paper` attribute names the paper. The answer would quote or link its source, and would say "the paper doesn't say" rather than guess. It would also need clear labelling as machine-generated.
- **Front end.** A small form at the end of `.more-body`, shown only when JS runs, so the static page still works without it.

## Unsure, or for you to decide

- **The seven entries without abstracts** (listed above) are the weakest. Their "why" sentences are generic. If you can share the abstracts, I can redraft them from the real text.
- **"PDF" labels.** Some `url_pdf` links are landing pages, not PDFs: doi.org, projecteuclid.org, amazon.science, the NeurIPS poster page and Google Drive. The panel calls them "PDF", like the publication list does. Renaming them, for example to "Journal" or "Paper", would be a front-matter or template change across the site.
- **`cpd-data`.** The panel text is deliberately minimal and neutral, like its figure.
- **Violet next to blue.** In `bandits`, a violet (hollow) arm sits next to blue (solid) ones. Fill style tells them apart, but under some colour-blind simulations the hue alone does not.
- **Page touches.** Teal headings and vermilion dates are the boldest of them. If they feel like too much, delete those two lines in the "Light touches" block of `main.css`.
- **The saved choice is site-wide.** The palette choice persists across all pages, but only the homepage and the publication list show colour in any quantity.

Lorenzo Masoero (masoerl), last updated 2026-11-08. Prepared with Claude.
