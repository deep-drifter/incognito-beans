# CREMA — notes for whoever edits this site

An independent, Michelin-style coffee guide. Shops are rated in **cups**. The guide is worldwide in
ambition and starts in Atlanta, so nothing in the interface should describe it as an Atlanta-only site.

These notes exist so that anyone — including future you — can change this site without rediscovering
how it fits together.

---

## How it runs

| Piece | What it does |
| --- | --- |
| GitHub repo | Holds the files. Editing a file here is how the site changes. |
| Vercel | Watches the repo and republishes within a minute or two of a commit. No build step. |
| Sanity | Holds the shops. The page fetches them in the browser when someone opens the site. |

Live at `incognito-beans.vercel.app`. Sanity project `cxp9yxyf`, dataset `production`.

### The files

| File | Purpose |
| --- | --- |
| `index.html` | The whole site: markup, styles and behaviour in one file. |
| `robots.txt` | Tells search engines and AI crawlers they're welcome, and where the sitemap is. |
| `sitemap.xml` | The list of pages for search engines. One entry today. |
| `manifest.webmanifest` | Lets people add CREMA to a phone home screen. |
| `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon.png` | Home-screen and tab icons. |
| `NOTES.md` | This file. |

**When the domain changes**, update it in four places: the `<link rel="canonical">` and the `og:` tags
at the top of `index.html`, plus `robots.txt` and `sitemap.xml`.

---

## The shop data

The page asks Sanity for every document of type `shop`. Fields in use today:

`name` · `neighborhood` · `cups` (0–3) · `price` ("$", "$$", "$$$") · `image` · `vibe` ·
`workFriendly` · `bestDrinks[]` · `address` · `review`

It also reads fields that **don't exist yet**, so they start working the moment they're added in Sanity:

| Field | Why you'd add it |
| --- | --- |
| `coordinates` | A "33.7540, -84.3711" string — right-click a shop in Google Maps and paste. Easiest way to put pins in the right place. |
| `lat` / `lng` or `location` | Same thing as separate numbers or a Sanity geopoint. |
| `award` | `none`, `selection`, `one`, `two`, `three` — replaces counting cups, and lets a shop sit unrated without showing up. |
| `city` | Needed once the guide leaves Atlanta. Until then every shop is Atlanta. |

### Two switches near the top of the script

```js
const EMPTY_CUPS_MEANS_SELECTION = true;
```
A shop with an empty `cups` field shows as ✦ Selection. That keeps old entries visible. Once every shop
has an `award` field, set this to `false` and unrated shops stay hidden until they're rated.

```js
const PINNED_COORDS = { 'portrait coffee': [33.73838, -84.42279], … };
```
Hand-looked-up coordinates for shops that have none in Sanity. A shop that's in neither place falls back
to the middle of its neighbourhood and the map says the pin is approximate. **Prefer putting coordinates
in Sanity** over adding to this list.

---

## The rating system

Cups judge **what's in the cup and how you're treated** — nothing else. Vibe, work-friendliness, price
and menu are tags people search by; they never earn cups. (Michelin scores food for stars and comfort
separately, for the same reason.)

| Award | What it means |
| --- | --- |
| ✦ Selection | Worth a stop. Good coffee made with care. |
| 1 Cup | Worth a visit if you're nearby. Excellent coffee, reliably. |
| 2 Cups | Worth a detour across town. Exceptional coffee and hospitality. |
| 3 Cups | Worth a dedicated trip. Benchmark coffee — among the best anywhere. |

Behind the award: **the cup** (the same test order every visit — black coffee, a cortado, matcha if it's
on the menu) counts double, plus **craft** signals visible from the counter, plus **hospitality**.
No cups from a single visit. Three cups needs repeat visits and a second inspector. The score suggests;
people decide.

A rating is always counted by repeating a small mark side by side — never by nesting rings, which can't
be counted below about 20px.

---

## Brand

- **Fonts:** Playfair Display for headings and the wordmark, Plus Jakarta Sans for everything else.
- **The mark — "The Crema Cone":** a pour-over cone with its top band filled, standing for the layer of
  crema. One drawing at every size. Never add anything below the point: a stem turns it into a cocktail
  glass. It lives as `<symbol id="i-mark">` near the top of the body.
- **The Seal** (print only): the cone inside a ring, "THE CREMA GUIDE" arced above, city and year below,
  cups beneath the cone, one ring per cup. For window decals and plaques, reissued each year.
- **Colours** live as CSS variables in one block at the top of `index.html` — change them there and the
  whole site follows, including the map.

| Token | Light | Dark |
| --- | --- | --- |
| `--bg` / `--surface` | `#F5F0E8` / `#FFFCF7` | `#17110D` / `#211913` |
| `--ink` / `--ink-2` | `#2A1E16` / `#6F5E51` | `#F3E6D4` / `#BCA68F` |
| `--accent` (gold as text) | `#8F6019` | `#DFAB44` |
| `--gold` (mark, cup icons) | `#A9762A` | `#DFAB44` |

Two golds in light mode is deliberate: text needs 4.5:1 contrast, graphics only 3:1, and one gold can't
do both on cream.

**Glass** belongs only where it floats over something rich — the nav, the sticky search bar, the map
controls, the card on the map. Cards on a flat background stay solid. Over a photo the panel goes nearly
opaque, or the text contrast changes with the picture.

---

## Page rules

- Order: hero → **How We Rate** → The Guide (search, filters, map, cards) → Nominate.
- The search and filters are a glass bar that sticks while you're inside the guide. On phones the filters
  become one swipeable row so the bar stays small.
- Nothing is labelled "AI" unless it really is AI.
- Shop text is built through DOM helpers, never pasted into HTML strings, so a quote mark in a review
  can't break the page.
- Loading shows skeletons; a failed fetch shows "The guide didn't load" with a Retry button — never an
  empty "no results", which reads as "there are no shops".

---

## The map

MapLibre GL (loaded only when the map is about to scroll into view) with free OpenFreeMap vector tiles,
recoloured in code from the palette above. Businesses, house numbers, boundaries and highway shields are
stripped out, so the only places on the map are CREMA's.

Things that will bite you:

- MapLibre positions a marker by setting `transform` on its **outer** element. Put hover scaling or
  `position` there and pins drift away from their coordinates — style the inner button instead.
- `cooperativeGestures: true` keeps the page scrolling normally over the map; zoom needs ⌘/Ctrl-scroll
  or two fingers.
- Re-fit the map when a filter changes, not on every keystroke, or it lurches while people type.
- Directions open Apple Maps on Apple devices and Google Maps everywhere else.

---

## Accessibility

The site is built to WCAG 2.2 level AA, and it's worth keeping it there.

- 4.5:1 contrast for text, 3:1 for icons. A symbol typed as a character (like ✦) counts as text.
- Everything clickable at least 24×24px.
- Everything reachable by keyboard, with visible focus. Opening the map card moves focus into it;
  Escape closes it and hands focus back.
- One live region (the "Showing 2 of 3 shops" line). Don't add more — a live list re-reads itself on
  every keystroke.
- Panels hidden with opacity are still read by screen readers: mark them `inert` when closed.
- Decorative icons get `aria-hidden="true"`.
- **Photos need real descriptions.** Today a screen reader announces only the shop's name for each photo.
  Adding an "Image description" field in Sanity is the biggest remaining accessibility gap.

To check: open the site, then in Chrome right-click → Inspect → Lighthouse → Accessibility. Then Tab
through the page without a mouse, and zoom to 200%.

---

## What's next, in order

1. **Coordinates into Sanity**, plus an "Image description" field.
2. **Next.js + Sanity on Vercel**, so every shop gets its own page rendered on the server. The AI
   assistants' crawlers (ChatGPT, Claude, Perplexity) don't run JavaScript, so today's list is invisible
   to them; Google can see it but a real page per shop is still better.
3. **Semantic search** over reviews (Sanity Dataset Embeddings) so "quiet place to read with good matcha"
   works properly.
4. **A phone app** (Expo/React Native on the same Sanity content) with near-me, saved shops and a visit
   passport. Apple rejects apps that are only a repackaged website, so it needs those.

---

## Third-party pieces to replace eventually

| Borrowed | Where it's from | Why to change it |
| --- | --- | --- |
| Hero photo | Unsplash | Own photography is part of the brand. |
| Background music | Pixabay CDN | Host a licensed track in the repo and point `MUSIC_SRC` at it. |
| Map tiles | OpenFreeMap | Free and fine at this size; Mapbox is the upgrade if you want a designer tuning the map. |

## Tags (room, amenities, intent)

Tags describe the **place**. They never affect the cup rating — that stays coffee
and hospitality only.

Inspectors tap them in the Studio (Shop → "The room" tab) from five predefined
grids defined in `schemaTypes/tags.js`:

| Field | What it covers |
|---|---|
| `vibeTags` | atmosphere — cozy, design-forward, quiet, historic building |
| `workTags` | outlets, wifi, noise, seating, laptop policy |
| `amenities` | oat milk, dogs, kids, parking, step-free entry, hours |
| `menuTags` | food, matcha, tea, alcohol, retail beans |
| `goodFor` | intent — remote working, first date, with kids, coffee nerds |
| `proposedTags` | free text, for anything not standardised yet |

**Values are slugs on purpose** (`oat-milk`, `dog-friendly-patio`). They are stable,
they read cleanly in a URL, they group identical shops no matter who tagged them,
and an assistant answering "dog friendly coffee in Atlanta" can match them exactly.
`TAG_TITLES` in index.html maps slug → human title and **must be kept in step with
`schemaTypes/tags.js`** — regenerate it from `ALL_TAG_GROUPS` when tags change.

Proposed tags are searchable immediately but are not filterable until promoted into
a grid. That gap is deliberate: it is what stops "dog friendly", "dog-friendly" and
"Dog Friendly" becoming three different filters.

On the site, tags render as a "Good to know" chip row on each card. Each chip is a
button that drops its label into the search box, so tag filtering needs no new UI.
Tags are also emitted in the JSON-LD as `keywords` and `amenityFeature`
(`LocationFeatureSpecification`), which is how ACS-style queries reach the guide
through search engines and AI assistants.

The legacy free-text `vibe` / `workFriendly` string fields were replaced by these
arrays. Old values still sit in the dataset but nothing reads them; the Studio shows
them as unknown fields with a Remove button.
