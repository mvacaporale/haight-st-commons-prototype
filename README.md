# Haight St. Commons — Platform Prototype

Design mockups for the planned [Haight St. Commons](https://www.haightstreetcommons.org/) self-serve housing platform — a site where the network's ~30 SF co-op / co-living houses manage their own profiles and room listings, and seekers browse and apply.

These are **static, self-contained HTML mockups** exported from Claude artifacts. They use dummy data (the fictional house "Fernwood") and have **no backend** — they exist to communicate the intended design and flows to collaborators and to the engineer who will build the real Next.js + Supabase app.

## What's here

| File | What it is |
|------|------------|
| [`index.html`](index.html) | Landing page linking all mockups (start here) |
| `haight-st-commons.html` | **Public home page** — Houses / Events / Projects directory, illustrated hero, mission band |
| `house-dashboard-flow.html` | **Signed-in house dashboard** (canonical prototype) — dashboard, house info, create-listing, with a "Builder notes" overlay |
| `commons-line-directory.html` | "The Commons Line" house directory + tag-bar filter |
| `house-directory.html` | Alternate house directory layout |
| `house-scrapbook.html` | Single house profile, scrapbook treatment |
| `commons-ledger.html` | Ledger-style directory mockup |
| `hero-banner.html` | Standalone hero / banner treatment |

## Viewing

Open any `.html` file directly in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Editing / collaborating

Each mockup is a single HTML file with inline CSS (and some inline JS) — edit the markup and styles directly. The shared design system is: serif display headings, dark-teal chrome, green accent, cream (`#faf9f5`) background.

To propose changes, fork or clone, edit, and open a pull request:

```bash
git clone https://github.com/mvacaporale/haight-st-commons-prototype.git
cd haight-st-commons-prototype
# edit, commit, push a branch, open a PR
```

## Status

Prototype / design-reference only. Nothing here is production code — the real platform (accounts, Supabase schema, listing lifecycle) is not yet built.
