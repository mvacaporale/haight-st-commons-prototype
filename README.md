# Haight St. Commons — Platform Prototype

Design mockups for the planned [Haight St. Commons](https://haight-st-commons.org/) self-serve housing platform — a site where the network's ~30 SF co-op / co-living houses manage their own profiles and room listings, and seekers browse and apply.

These are **static, self-contained HTML mockups** exported from Claude artifacts. They use dummy data (the fictional house "Fernwood") and have **no backend** — they exist to communicate the intended design and flows to collaborators and to the maintainer who will build the real thing. The planned design is described in [Platform design](#platform-design) below.

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

## Platform design

Goal: houses update their own info and room openings, seekers can see who has openings right now, and the site stays easy for one volunteer to maintain. **No app server** — a static site plus Supabase.

### Pieces

| Piece | Where it lives | Notes |
|-------|----------------|-------|
| Public site | Existing repo [`haight-st-commons.github.io`](https://github.com/haight-st-commons/haight-st-commons.github.io), GitHub Pages, `haight-st-commons.org` | One home page, generated from Supabase data at build time |
| House dashboard | A static page on the same site (e.g. `/dashboard/`) | Talks to Supabase directly with `supabase-js`; see `house-dashboard-flow.html` |
| Data + auth | Supabase (free tier) | Postgres tables, row-level security (RLS), magic-link email login |
| Small server-side jobs | Supabase Edge Functions + `pg_cron` | Geocode on save, listing expiry, re-confirm emails |
| Build | GitHub Action in the site repo | Rebuilds the home page on every house/listing save and nightly |

No Next.js / Node server is needed: RLS is the security boundary, so the browser can read and write Supabase directly.

### Data model

- **`house`** — the persistent profile: name, `slug`, private `address`, neighborhood, bedrooms, residents, tags, description, contact, photos, `status` (`current` / `past`), `legacy_anchors` (see SEO).
- **`listing`** — zero or more per house: `house_id`, `move_in`, `rooms`, `rent`, `dues`, `term`, `apply_url`, `note`, `created_at`, `confirmed_at`, nullable `review_status` (unused in v1).
- **`house_members`** — maps a login (`user_id`) to a `house_id`. 1–3 resident emails per house.
- **`public_houses`** view — everything the public site needs, **without** `address`. The build and the public page read only this.

RLS: anyone can read the public view and active listings; a signed-in user can insert/update/delete `house` and `listing` rows only where `house_id` is in their `house_members` rows.

### Accounts

- Houses apply through the existing "Add your House" form. An admin approves, creates the `house` row and adds resident emails to `house_members`. Approval = the **Verified** chip.
- Login is a Supabase magic link (email only). No passwords, no SMS verification.
- When a resident moves out, swap their email in `house_members`.

### Listing lifecycle

- Publishing a listing is live immediately (no moderation in v1).
- A daily `pg_cron` job expires listings where `now − confirmed_at > 45 days`; an Edge Function emails a "still open?" nudge around day 40 (Resend or similar free tier). Clicking "Still open" bumps `confirmed_at`.
- "Mark filled" removes the listing; house info stays up.
- Applications are never collected by the site — seekers go to each house's own `apply_url`.

### How the public site is built

1. A Supabase database webhook on `house` / `listing` changes triggers a `repository_dispatch` on the site repo; a nightly schedule runs as a backstop (and keeps the free Supabase project from pausing).
2. The GitHub Action reads `public_houses` + active listings and renders `index.html` with a small static generator (Eleventy, or a short Node script with a template).
3. GitHub Pages serves the result at `haight-st-commons.org`, exactly as today.

## Keeping the SEO

Today all search value sits on one URL: the home page. Its ~4,000 words of house text are plain, crawlable HTML, and `#house` links are part of that same page as far as Google is concerned. The redesign keeps that page and makes it better.

**Must keep**

- **Same domain and URL.** Keep `CNAME` = `haight-st-commons.org` and the home page at `/`. Keep `/projects/` and `/newspaper/` at their current paths (or 301 them if they move).
- **Houses in the HTML, not injected by JavaScript.** The build writes every house card (name as an `<h2>`, description, neighborhood, photo, contact/apply links) into `index.html`. Filters only show/hide cards that are already in the page. (The `haight-st-commons.html` mockup renders cards from JS — production must not.)
- **Keep the text.** Each house keeps a real description paragraph, not just tags. Keep the "Houses of Our Past" section (`status = past`); it's indexed content and history.
- **Old anchors keep working.** Current links look like `#beacon-house` (generated from the heading by the bootstrap-toc plugin) and, for most houses, a hand-written id like `#embassy`. Give each house card `id="{slug}"` using the heading-style slug, and render each `legacy_anchors` value as an extra empty anchor element next to it. Seed both from the current `index.html` before launch.
- **Expired listings never remove a house.** Only the "opening" badge disappears.

**Cheap improvements to ship with the redesign**

- A more descriptive `<title>` and meta description (e.g. "Haight St. Commons — co-op and co-living houses in San Francisco"), unique ones for `/projects/` and `/newspaper/`.
- `<link rel="canonical" href="https://haight-st-commons.org/">`, Open Graph/Twitter tags with a share image.
- `sitemap.xml` and `robots.txt` (neither exists today).
- JSON-LD `Organization` for the network, with the house list as an `ItemList`.
- Image `alt` text = house name; compress photos.
- Fix the five broken links on the current page that are missing `https://` or `mailto:` (e.g. `chrysalisberkeley.com`, `heartloomhouse@gmail.com`).

**Launch checklist**

1. Before launch: verify the domain in Google Search Console and note current queries/clicks; replace the dead Universal Analytics tag (`UA-83416444-1`) with GA4 or Plausible.
2. Test every old anchor (both id styles) against the new page.
3. Launch, submit `sitemap.xml` in Search Console, and watch coverage and queries for 2–4 weeks.

## Maintenance

What the maintainer actually does once this is live:

- **Approve new houses** in the Supabase table editor (create `house`, add member emails).
- **Occasionally fix a member email** or move a house to `past`.
- **Nothing for listings** — houses post, confirm and close them; expiry is automatic.

Secrets: the Supabase anon key is public by design (RLS protects data). The service-role key lives only in GitHub Action secrets and Edge Function env vars. Both Supabase and GitHub Pages run on free tiers at this scale.

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

Prototype / design-reference only. Nothing here is production code — the Supabase project, dashboard page and site build are not yet built.
