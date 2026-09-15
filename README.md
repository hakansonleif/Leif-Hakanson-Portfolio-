# Leif Hakanson — Portfolio

Static site for GitHub Pages. No build step, no dependencies to install.

## Structure

```
index.html                     the whole page (339 KB)
assets/
  letters/       *.jpg         three recommendation letter scans
  papers/        *.pdf         Ukraine research paper, Monday Effect paper
  models/        *.xlsx        Deere diagnostic, HAM Munich model, Westin REF model
  certificates/  *.pdf         OSINT (Centri), Cybersecurity Fundamentals (IBM)
  cv/            *.pdf         the one-page CV, linked from the hero and the footer
  og-image.jpg                 1200x630 link-preview card
  favicon-32.png               browser tab icon
  apple-touch-icon.png         180px iOS home-screen icon
```

`og-image.jpg` and both icons are GENERATED, not hand-made. `make_og.py` pulls the
hero photograph out of `v6_test.html`, composes the card and exports the monogram
icons; `build_site.py` runs it on every build. Edit the script, never the outputs.

`index.html` must sit at the repository root, with `assets/` beside it. All
internal links are relative, so the site also works by double-clicking
`index.html` locally.

## Deploying

From inside this folder:

```bash
git init
git remote add origin https://github.com/hakansonleif/Leif-Hakanson-Portfolio-.git
git fetch origin
git checkout -b main
git add index.html assets README.md
git commit -m "Rebuild portfolio: globe, references, model viewers, assets split out"
git push -u origin main --force
```

`--force` is there because this replaces the old single-file `index.html`.
Drop it if you would rather merge by hand.

Pages then serves it at https://hakansonleif.github.io/Leif-Hakanson-Portfolio-/
within a minute or two.

## Things worth knowing

**External dependencies.** The globe pulls three files from public CDNs at
runtime: d3 7.9.0 and topojson 3.0.2 from cdnjs, and the world map data
(`world-atlas@2/countries-50m.json`) from jsDelivr. If any of them fail the
globe shows a fallback message and the rest of the page is unaffected. To make
the site fully self-contained, download those three files into `assets/vendor/`
and change the three URLs at the top of `index.html` and in the globe script.

**The 50m map file matters.** The lower-resolution 110m file omits Singapore,
Luxembourg, Malta and Monaco entirely. Do not switch it back.

**Dashboards sit behind a "Show statistics" button.** The Deere, Munich and
Westin project cards each carry a collapsed `.dash-panel`. On first click the dashboard
is built into it, the panel animates open, and the marks are revealed in
sequence: figure tiles and cards fade up, lines draw themselves along their own
path length, bars grow from the baseline, heatmap cells stagger in. The reveal
uses the Web Animations API (`element.animate`), not CSS transitions, because a
transition started inside a zero-height panel never commits its start value and
the marks snap straight to their end state. It respects
`prefers-reduced-motion`, where everything simply appears.

"Browse all sheets" opens a separate lightbox holding only the raw spreadsheet
tables.

Both come from `OVERVIEW` and `WORKBOOKS` objects inside `index.html`, extracted
from the workbooks at build time. They are a snapshot: if you edit either .xlsx,
the page will not update on its own and both objects have to be regenerated.

**Chart palette.** The two series colours `#8A3F3F` and `#A8761F` are deliberate
steps of your burgundy and gold. They were checked against a colour-vision
validator and clear the lightness band, chroma floor, colourblind separation
(ΔE 15.4) and 3:1 contrast on white. The raw brand values `#6B3E3E` / `#C9A876`
fail the chroma and contrast gates, so do not swap them back into the charts.
Each dashboard has one "Show the numbers" control above the chart grid that swaps
every chart in that dashboard for its underlying table, so nothing is reachable
only by colour. It used to be a button per chart, which meant thirteen of them.

**Numbers match Excel exactly.** The viewer honours each cell's number format,
so what it shows is what the workbook shows, including any formatting quirks in
the source.

## The Deere workbook

`assets/models/Deere_Financial_Diagnostic.xlsx` is not one of the original course
files. It consolidates five submissions (PAF bonus 1, PAD Excel Data, WIKI 2, 3
and 4) into a single live model of 225 formulas across seven sheets. Nothing in
it is pasted: every margin, ratio, cost of capital and return recalculates from
the reported inputs.

Two definitional points were recovered by reconciling against the originals and
are worth keeping if the model is ever rebuilt:

- **Cost of debt and the capital weights use AVERAGE balances across the year**,
  not closing balances. Closing balances do not reproduce the source figures.
- **Return on equity and the weights use equity INCLUDING minority interests.**
  The leverage ratio on the liquidity sheet uses the figure excluding them, which
  is what the source did. Both are carried so each ratio uses the right base.

Sheet 6 holds eight live audit checks that recompute on every recalculation. All
eight pass. Total shareholder return is computed here from the monthly price and
dividend series in WIKI 4, because the source workbook has the data but only one
populated result cell.

## Content corrections, September 2026

Six claims on the page did not survive a check against the source files and the CV.
They are fixed in `build_site.py` under section 8B, each with the anchor it replaces,
so the edit is visible rather than buried in the HTML.

| Was | Is | Why |
|---|---|---|
| "Five-tab integrated Excel model" | "Seven-sheet" | The workbook linked on that card has 7 sheets |
| "61/65 PAF audit checks passed" | "Eight audit checks, all pass" | 61/65 has no support in any source file. `3_Audit_Results` in the original holds 25 status cells; the consolidated workbook holds 8 live checks |
| "$486B Reconstruction Pledged" | "Reconstruction Needs" | World Bank RDNA3, March 2024, is an estimate of needs. Pledged funding is far lower and the paper itself says needs |
| "46% Global Sunflower Oil" | "Share of Sunflower Oil Exports" | The chart measures export share, not output |
| "Eight months at Shangri-La Singapore" | "Seven months" | CV dates are 02.2023 to 08.2023 |
| "Lycée Privée de l'Alma" | "Lycée Privé" | *Lycée* is masculine. The same typo is still on the CV PDF |
| "Third year at EHL" | "Final year at EHL" | Against "2022 - 2027" a reader does the arithmetic and gets year four or five |

Two things a reader could still catch, left alone because only Leif can settle them:

- The **Student Safety Team role reads Supervisor**, corrected by Leif in September 2026.
  The Barrière letter says *member*; an earlier draft of this page said Head of Team.
- The **CV says German B2 and lists no Spanish**; the page says German B1 and Spanish B1.
  One of the two documents is wrong.

## Structural additions

- **Experience section**, between About and Projects, with the four CV placements and the
  Student Safety Team. The site previously had no employment history at all, which is a
  strange gap in something described as an extension of a CV.
- **CV download** in the hero (ghost button beside the two existing calls to action) and as
  a fourth block in the contact footer.
- **Nav marks the section in view** through an IntersectionObserver, and `section` carries
  `scroll-margin-top: 92px` so anchor targets stop landing underneath the fixed header.
- **Collapsing a dashboard scrolls its card back into view** when the button has gone above
  the fold, which it does on every dashboard once open.
- Certificates now sit **above** Education rather than below it, at the bottom of the page.
- Emoji removed from the three paper buttons, and "Read more" on the focus cards is now a
  bordered button like every other control.

## Reach fixes, September 2026

**Link previews.** The page had no `description`, no Open Graph, no Twitter card and no
favicon, so pasting the URL into LinkedIn, WhatsApp or an email produced a bare string.
All of it now sits in the head, pointing at `og-image.jpg`. The URLs in those tags are
ABSOLUTE and hardcoded to

    https://hakansonleif.github.io/Leif-Hakanson-Portfolio-/

which is `SITE_URL` at the top of `build_site.py`. Serve the site from anywhere else and
those tags have to change, or the preview breaks: scrapers do not resolve relative paths.
After deploying, force LinkedIn to re-read the page at
https://www.linkedin.com/post-inspector/ , because it caches previews aggressively.

**Mobile menu.** Below 768px the nav used to hide five of its six links, leaving only
"Get in Touch" reachable and every section findable only by scrolling all 11 screens.
There is now a hamburger button that opens the full list as an opaque dropdown, closes on
a link click or Escape, and resets itself on resize above 768px. The panel is fully
opaque on purpose: at 98% the hero heading ghosted through it.

**Keyboard access.** The five focus cards were `<div onclick>` with no tabindex, the only
controls on the page that a keyboard could not reach. They now carry `role="button"`,
`tabindex="0"` and a live `aria-expanded`, and `focusKey()` handles Enter and Space
(Space has its default suppressed so it does not scroll the page). The globe canvas has an
`aria-label` naming what it shows and pointing at the text lists beside it. Visible focus
rings are on the cards, the menu button, the dashboard toggles and the file buttons.

## Contact

The footer runs five blocks: Email, Phone, LinkedIn, Based In, CV. `hakansonleif@gmail.com`
is the primary address and the one the `mailto:` uses; `leif.hakanson@ehl.ch` sits under it
as a second line and should be deleted once the student account lapses.

## Still open

- The three reference letters are 7% of page height and start 11.4 screens down. They are
  the strongest evidence on the site and almost nobody will scroll that far. Consider
  lifting one pull-quote into the hero or the top of Experience.
