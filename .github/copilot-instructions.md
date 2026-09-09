## Copilot instructions for this repo

This project is an Eleventy (11ty) static site using Nunjucks templates and YAML data. Prefer small, precise suggestions grounded in the structures below.

### Tech/context

- Static site generator: Eleventy 3.x (`@11ty/eleventy`)
- Templates: Nunjucks under `src/`
- Data: YAML under `src/_data/`
- Output: `_site/`
- Scripts: `npm run dev` (serve), `npm run build` (generate), `npm run format` (auto-format), `npm run lint` (check formatting)
- Formatting: Prettier (config in `.prettierrc`, VS Code format-on-save via `.vscode/settings.json`)
- CI: GitHub Actions in `.github/workflows/` — `build.yml` (Eleventy build) and `prettier.yml` (formatting check). Note `prettier.yml`'s job is _named_ `validate-yaml` but runs `npx prettier --check .` across the whole repo, so a failure there is usually JSON/JS/CSS, not YAML.
- Deploy: Cloudflare Workers static assets. `wrangler.jsonc` must exist — without it `wrangler deploy` runs its first-run setup wizard, which auto-accepts in CI and re-runs the build a second time.

### Data contracts (authoritative)

- Field labels and ordering are defined in `src/_data/specs.yml` (groups → fields with `key` and `label`).
- Chips live in `src/_data/chips-*.yml` (one file per series, e.g., `chips-m.yml`, `chips-a.yml`).
- Every chip entry must include:
  - `id` (unique across all chip files)
  - `name`
  - `specs` (object keyed by the `key` values declared in `specs.yml`)
- Series live in `src/_data/series.yml` and reference chips by `id` under `ranks[*].variants`.
- Device categories live in `src/_data/devices.yml` and reference chips by `id` under `devices[*].variants`.
- News posts are Markdown in `src/news/posts/`, not YAML. Shared front matter (layout, tag, permalink) lives in `src/news/posts/posts.11tydata.js`, so each post carries only `title`, `date`, `summary` and `categories`.

### Build-time behavior

`.eleventy.js` loads YAML and creates the following collections:

| Collection                   | Source                             | Purpose                                                                                                                 |
| ---------------------------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `devicesCollection`          | `devices.yml`                      | Raw device categories for listing pages                                                                                 |
| `devicePagesCollection`      | `devices.yml` (flattened)          | One entry per device, enriched with `categoryId`/`categoryName`, `generationGroups`, `url`, `discontinued`, `chipNames` |
| `deviceGroupPagesCollection` | `devices.yml` (grouped categories) | One entry per group (e.g. iPhone 17, iPad Pro), with `allVariants`                                                      |
| `seriesCollection`           | `series.yml`                       | Raw series for listing pages                                                                                            |
| `rankPagesCollection`        | `series.yml` (flattened ranks)     | One entry per rank/tier, enriched with `seriesId`/`seriesName`, `siblingTiers`, `prevGenTier`, `nextGenTier`            |
| `chipsCollection`            | `chips-*.yml`                      | All chips enriched with `groupedSpecs`                                                                                  |
| `chipPagesCollection`        | `chips-*.yml` + devices + series   | All chips enriched with `groupedSpecs`, `devices` list, `rank` info, `url`, `hasOwnPage`                                |
| `chipVariantPagesCollection` | `chipPagesCollection` filtered     | Only chips with `hasOwnPage` — the ones that get their own variant page                                                 |
| `generationPagesCollection`  | `series.yml` + `chips-*.yml`       | One entry per generation with merged tier specs, `allVariants`, `tiers` (links to each tier page), `onSale`             |
| `newsPosts`                  | `src/news/posts/*.md`              | Every post tagged `post`, sorted newest first. Give same-day posts a time to order them; otherwise filename decides     |

Each chip is enriched with `groupedSpecs` computed from `specs.yml`. Templates read from `groupedSpecs` — don't recompute in templates.

`rankPagesCollection` entries include navigation helpers computed at build time:

- `siblingTiers` — other tiers in the same generation (e.g. Pro alongside Max)
- `prevGenTier` / `nextGenTier` — equivalent tier in the previous/next generation

`devicePagesCollection` entries include `generationGroups` — an array of `{ genId, label, chipIds, isCurrent }` grouping variants by generation, newest first.

Not every chip gets a variant page. A chip whose id has no `<cpu>-<gpu>` suffix (N1, C1, C1X, R1, T1/T2, U1/U2, W1–W3, all S-series) is the only member of its rank, so its rank page already says everything a variant page would. Those chips set `hasOwnPage: false`, are excluded from `chipVariantPagesCollection`, and their `url` points at the rank page instead. Always link via `chip.url` rather than assembling the path.

`devicePagesCollection` and `generationPagesCollection` treat a device as discontinued when **either** `device.deprecated` **or** its group's `deprecated` is set. Checking only `device.deprecated` silently misses whole product lines — devices inside a deprecated group are usually not flagged individually.

### URL structure

| URL pattern                             | Template                    | Collection                   |
| --------------------------------------- | --------------------------- | ---------------------------- |
| `/chips/`                               | `chips/index.njk`           | (all series + generations)   |
| `/devices/`                             | `devices/index.njk`         | (all categories)             |
| `/series/<series>/`                     | `series/series.njk`         | `seriesCollection`           |
| `/chips/<gen>/`                         | `series/generation.njk`     | `generationPagesCollection`  |
| `/chips/<gen>/<tier>/`                  | `series/rank.njk`           | `rankPagesCollection`        |
| `/chips/<gen>/<tier>/<cpu>-<gpu>/`      | `chips/chip.njk`            | `chipVariantPagesCollection` |
| `/devices/<category>/`                  | `devices/devices.njk`       | `devicesCollection`          |
| `/devices/<category>/<group>/`          | `devices/device-group.njk`  | `deviceGroupPagesCollection` |
| `/devices/<category>/<device>/`         | `devices/device.njk`        | `devicePagesCollection`      |
| `/devices/<category>/<group>/<device>/` | `devices/device.njk`        | `devicePagesCollection`      |
| `/current/<subset>/`                    | `current/*.njk`             | (static pages)               |
| `/compare/custom/`                      | `compare/custom.njk`        | (static page)                |
| `/compare/gaming/`                      | `compare/gaming.njk`        | (static page)                |
| `/compare/ml-ai/`                       | `compare/ml-ai.njk`         | (static page)                |
| `/compare/video-editing/`               | `compare/video-editing.njk` | (static page)                |
| `/gen-gains/m-series/`                  | `gen-gains/m-series.njk`    | (static page, Chart.js)      |
| `/gen-gains/a-series/`                  | `gen-gains/a-series.njk`    | (static page, Chart.js)      |
| `/news/`, `/news/page/<n>/`             | `news/index.njk`            | `newsPosts` (20 per page)    |
| `/news/<slug>/`                         | `_layouts/post.njk`         | `newsPosts`                  |
| `/api/search.json`                      | `api/search.njk`            | (search index)               |
| `/api/chips.json`                       | `api/chips.njk`             | (full chip data)             |
| `/sitemap.xml`, `/robots.txt`           | `sitemap.njk`, `robots.njk` | (generated)                  |

Example: chip `m4-pro-14-20` → `/chips/m4/pro/14-20/`; chip `m1-8-8` → `/chips/m1/base/8-8/`.

For grouped device categories (iPhone, iPad, Watch, AirPods), devices nest under their group slug, e.g. `/devices/iphone/17/air/`. For flat categories (Mac), devices sit directly under the category, e.g. `/devices/mac/macbook-pro-16/`.

### Global helpers (Nunjucks)

- `getChips(ids, collections)` — returns enriched chip objects for the given list of `id`s in order.
- `getDevicesForChips(ids)` — returns a de-duplicated list of `{id, name, categoryId, categoryName, groupId, groupName, deprecated, url}` for every device that has _ever_ used any of the given chip IDs. Deliberately **not** filtered by `deprecated` — this is the historical list; the compare table's "Current Devices" row is the current one.
- `getChipRange(ids, collections)` — returns a human-readable range string, e.g. `"M2 – M4"`.
- `getCurrentChips(categoryId, sectionOrGroupId?)` — returns current-gen primary chip IDs for non-deprecated devices in a category. Pass a `section` ID for Mac (`"laptop"` / `"desktop"`), a `group` ID for grouped categories, or omit for all. Historical variants are excluded — only the newest generation per device is returned.
- `groupCompanionsBySeries(ids)` — groups companion (non-primary) chip IDs by series for the companion-chip section on device pages.
- `getCurrentDevices(categoryId, sectionOrGroupId?)` — same filtering as `getCurrentChips`, but returns device objects `{id, name, variants, categoryId, categoryName, url}` suitable for passing to `deviceGrid`. The `variants` field is trimmed to current-gen primary chips.
- `relatedPosts(collections, pageIds, limit?)` — posts whose `categories` are an ancestor or descendant of this page's id, e.g. an `m5-max` page matches a post filed under `m5`. Returns `[]` for an unknown id, so hardware pages can call it unconditionally.

### Filters (Nunjucks)

- `genId` — extracts generation prefix from a rank/chip ID (`"m4-pro"` → `"m4"`).
- `tierSlug` — extracts tier from a rank ID (`"m1-pro"` → `"pro"`, `"m1"` → `"base"`).
- `chipTier` — extracts tier from a chip ID (`"m4-pro-14-20"` → `"pro"`, `"m1-8-8"` → `"base"`).
- `chipVariant` — extracts variant suffix from a chip ID (`"m4-pro-14-20"` → `"14-20"`).
- `familyLabel` — family label from a rank/chip id (`"m5-ultra"` → `"M5"`, `"c1-x"` → `"C1"`, `"a16"` → `"A16"`). Derived from the generation id, so the label can't drift from the family URL it links to. Prefer this over string-stripping tier words off a name.
- `companionChips` — inverse of `primaryChips`: keeps only the non-primary companion chips (C, N, R, T, U, W).
- `map` — maps an array by a property name. Note Nunjucks has **no** Jinja-style `map(attribute=…)`; collect values with an explicit loop.
- `resolveCategories` — turns a post's `categories` ids into `{label, url}` links. An id that names no real hardware is dropped with a `[news] unknown category id` warning rather than linked to a 404, so watch the build output after writing a post.
- `postDate` / `isoDate` — a post date as `9 September 2026` (en-GB, UTC) and as `2026-09-09` for `datetime` attributes.
- `primaryChips(ids, referenceIds?)` — filters a list of chip IDs to keep only "primary" SoCs (A, M, S series). When H-series chips are present and no A/M/S chips exist, H is promoted to primary. All other series (R, U, W, T, N, C) are tertiary and filtered out. Used to derive what shows in tables, cards, and headings on device pages.

### Macros

- **Comparison table**: `src/_includes/macros/compare-table.njk`
  - Import: `{% from "macros/compare-table.njk" import compareTable %}`
  - Usage: `{% set items = getChips(["m4-max-16-40"], collections) %} {{ compareTable(items) }}`
  - Renders fields based on `items[0].groupedSpecs` and aligns columns by index across items.
  - The "Current Devices" row lists only devices for which this is the _current_ chip. The carousel below carries the full history.
  - `.compare-table-wrapper` breaks out of the page `--max-width` so wide tables can use the whole viewport before scrolling. The table is `width: max-content; min-width: var(--prose-width)`.
- **Focused comparison table**: `src/_includes/macros/focused-compare-table.njk`
  - Import: `{% from "macros/focused-compare-table.njk" import focusedCompareTable %}`
  - Usage: `{% set items = getChips(["m4-max-16-40"], collections) %} {{ focusedCompareTable(items, ["cpu_cores", "gpu_cores", "memory_bandwidth_gb_s"]) }}`
  - Renders a subset of spec keys for a group of chips. Skips rows where no item has data.
- **Device carousel**: `src/_includes/macros/device-carousel.njk`
  - Import: `{% from "macros/device-carousel.njk" import deviceCarousel %}`
  - Usage: `{% set devices = getDevicesForChips(chipIds) %} {{ deviceCarousel(devices) }}`
  - Renders a responsive grid of device cards (it is no longer a scrolling carousel, despite the name) under the heading "All Devices, Including Discontinued". Discontinued devices are dimmed and badged in place, and the list stays in `devices.yml` order — do **not** partition it into current-then-discontinued, because deprecation is per-device and that would split one product family across two sections. Used on chip, rank, and generation pages.
- **Device grid**: `src/_includes/macros/device-grid.njk`
  - Import: `{% from "macros/device-grid.njk" import deviceGrid %}`
  - Usage: `{{ deviceGrid(devices, collections) }}` or `{{ deviceGrid(devices, collections, "/devices/mac/") }}`
  - Renders a grid of device cards with chip count and range summary. Active devices first, `deprecated` devices last with a "Discontinued" badge. Pass `baseUrl` when device URLs aren't pre-computed. Used on `/current/*` and device-group pages.
- **Spec card**: `src/_includes/macros/spec-card.njk`
  - Import: `{% from "macros/spec-card.njk" import specCard %}`
  - Usage: `{{ specCard(chip) }}`
  - Renders a summary card of key chip highlights (CPU/GPU/NPU cores, memory bandwidth, memory options, process node). Used on individual chip pages.
  - The CPU breakdown adapts to whichever core tiers a chip has, e.g. `2S + 4P + 6E` (M6), `6S + 12P` (M5 Max), `4P + 6E` (M4).
- **Generation nav**: `src/_includes/macros/gen-nav.njk`
  - Import: `{% from "macros/gen-nav.njk" import genNav %}`
  - Usage: `{{ genNav(prevGenTier, nextGenTier, siblingTiers) }}`
  - Renders previous/next generation links and same-generation sibling tier links. Used on rank pages.
  - Takes an optional 4th argument to relabel the links row: generation pages pass `gen.tiers` with `"Tiers:"` to link down to their tier pages.
- **Post list**: `src/_includes/macros/post-list.njk`
  - Import: `{% from "macros/post-list.njk" import postSnippets, postCategories, relatedPostList %}`
  - `postSnippets(posts)` renders date/title/summary cards — the homepage passes the newest 3, `/news/` passes a page of 20.
  - `postCategories(categories | resolveCategories)` renders the "Covers" footer on a post.
  - `relatedPostList(posts, heading)` renders nothing when `posts` is empty, so hardware pages can call it unconditionally.
- **Breadcrumbs**: `src/_includes/macros/breadcrumbs.njk`
  - Import: `{% from "macros/breadcrumbs.njk" import breadcrumbs %}`
  - Usage: `{{ breadcrumbs([{ label: "Mac", url: "/devices/mac/" }, { label: "MacBook Air 13″" }]) }}`
  - Renders an accessible `<nav>` breadcrumb trail. The last crumb has `aria-current="page"` and no link. Always prepends a "Home" crumb automatically.

### Page templates

- `src/chips/index.njk` — `/chips/`, every series and its generations
- `src/devices/index.njk` — `/devices/`, every product line and its groups/devices
- `src/series/series.njk` — series listing (e.g. M Series)
- `src/series/generation.njk` — generation family comparison (e.g. M4 Family) with device carousel
- `src/series/rank.njk` — rank/tier variant comparison (e.g. M4 Pro) with device carousel and gen-nav
- `src/chips/chip.njk` — individual chip variant spec sheet with spec card and device carousel
- `src/devices/devices.njk` — device category listing (e.g. Mac)
- `src/devices/device-group.njk` — group-level device listing for grouped categories (e.g. iPhone 17)
- `src/devices/device.njk` — individual device chip comparison (e.g. MacBook Pro 14); shows multi-generation sections when `generationGroups` has more than one entry
- `src/compare/custom.njk` — ad-hoc chip comparison (JS-driven, URL-shareable)
- `src/compare/gaming.njk` — gaming-focused chip comparison using `focusedCompareTable`
- `src/compare/ml-ai.njk` — ML/AI-focused chip comparison using `focusedCompareTable`
- `src/compare/video-editing.njk` — video editing focused chip comparison using `focusedCompareTable`
- `src/current/macbooks.njk` — current MacBook lineup using `getCurrentChips`/`getCurrentDevices`
- `src/current/macs.njk` — current desktop Mac lineup
- `src/current/iphones.njk` — current iPhone lineup
- `src/current/ipads.njk` — current iPad lineup
- `src/current/watches.njk` — current Apple Watch lineup
- `src/current/airpods.njk` — current AirPods lineup
- `src/gen-gains/m-series.njk`, `src/gen-gains/a-series.njk` — generational-gains charts (Chart.js; data in `assets/gen-gains-*.js`, not YAML)
- `src/news/index.njk` — `/news/`, paginated 20 posts per page
- `src/_layouts/post.njk` — a single post, with breadcrumbs, date and the "Covers" category footer
- `src/api/search.njk` — the search index, `src/api/chips.njk` — full chip JSON
- `src/_layouts/base.njk` — also renders the footer site map, generated from `devicesCollection`, `seriesCollection` and `generationPagesCollection`

### Specs conventions

- Use exact keys from `specs.yml`; missing values are omitted.
- Types:
  - Numbers for counts/sizes (e.g., `cpu_cores: 16`, `transistor_count_billion: 28`).
  - Booleans as YAML booleans (`true`/`false`).
  - Arrays for multiple values (e.g., `memory_options: [8, 16, 24, 32]`, `hardware_acceleration: ["H264", "HEVC"]`).
  - Strings for names/types (e.g., `memory_type: "LPDDR5X-8533"`).
- Units: encode values to match labels (e.g., GHz if label says "(GHz)"). Avoid conflicting inline comments.
- Display order is controlled solely by `specs.yml`.

### Device data conventions

- `deprecated: true` on a device marks it as discontinued. It still appears in its category listing (sorted last) and keeps its page, but is excluded from `/current/*` pages and `getCurrentChips`/`getCurrentDevices` results.
- `section` on a flat-category device (e.g. Mac) allows filtering by sub-group: `"laptop"` or `"desktop"`. Pass the section id as the second argument to `getCurrentChips`/`getCurrentDevices`.
- Grouped categories (iPhone, iPad, Watch, AirPods) use `groups` instead of `devices` at the top level. Each group has an `id`, `name`, and `devices` array. A `deprecated: true` group is excluded from `/current/*` pages.
- A device is discontinued if `device.deprecated` **or** its group's `deprecated` is set — always test both.
- `variants` should list every chip configuration ever offered, newest first. Historical variants are filtered at build time so only the newest generation appears in `getCurrentChips` results.

### IDs and naming

- Chip `id` must be unique across all `chips-*.yml` files.
- Recommended id pattern: `<series><gen>[-tier][-cpu]-<gpu>` (lowercase, hyphens), e.g., `m4-max-16-40`, `a17-pro-12-24`.
- Keep `name` human-readable (e.g., `"M4 Pro"`).

### Chip ordering

Chips in `chips-*.yml` files should be ordered by:

1. **Generation** (newest to oldest)
2. **Tier** (Ultra → Max → Pro → base)
3. **GPU cores** (most to fewest within each tier)

Example M-series order: `m6-12-12`, `m5-ultra-36-80`, `m5-ultra-30-64`, `m5-max-18-40`, `m5-max-18-32`, `m5-pro-18-20`, `m5-pro-15-16`, `m5-10-10`, `m5-9-10`, `m5-10-8`, `m4-max-16-40`, etc.

### Series ordering

`series.yml` order is **M, A, then alphabetical** (M, A, C, H, N, R, S, T, U, W). This is load-bearing beyond presentation: `.eleventy.js` derives `genOrder` from the sequence of ranks in this file, and `currentGenPrimaryChips` uses `genOrder` to decide which chip is "current" for a device. Reordering whole series blocks is safe (comparisons only run between chips within one device); reordering ranks _inside_ a series is not.

The footer's "Chips by Series" column re-sorts to fully alphabetical at render time — M and A are highlighted there rather than hoisted.

### Adding chips from Wikipedia

When adding chip data from Wikipedia's comparison tables:

1. **Locate the table**: Use the "Comparison of M-series processors" or "Comparison of A-series processors" table at https://en.wikipedia.org/wiki/Apple_silicon
2. **Extract specs systematically**:
   - CPU: cores (P+E breakdown), clock speeds (GHz)
   - GPU: vendor, cores, EUs, ALUs, frequency (MHz), TFLOPS, ray tracing support
   - NPU: cores, TOPS
   - Memory: type (e.g., "LPDDR5-6400"), bandwidth (GB/s), bus width (bits), channels, options (GB array)
   - Media engines: hardware acceleration array, video decode/encode counts, ProRes engines, AV1 decode
   - Semiconductor: process node (nm), transistor count (billion), die size (mm²)
   - Release date (YYYY-MM-DD format)
3. **Match keys to specs.yml**: Use exact keys from `specs.yml` (e.g., `performance_cores`, `gpu_tflops`, `memory_type`)
4. **Handle variants**: Create separate chip entries for each GPU/CPU configuration (e.g., M3 has 8-core and 10-core GPU variants)
5. **Order correctly**: Insert chips in the proper order (see "Chip ordering" above)

### News posts

Posts live in `src/news/posts/<slug>.md` and publish at `/news/<slug>/`. Front matter is only:

```yaml
---
title: "Mac mini and Mac Studio get M6, M5 Pro, M5 Max and M5 Ultra"
date: 2026-08-25
summary: "One or two sentences with real numbers. Doubles as the meta description."
categories: ["m6", "m6-12-12", "mac/mac-mini"]
---
```

- `date` is the announcement date, not the on-sale date. `YYYY-MM-DD` on its own is the norm; add a time (`2026-09-09 12:00:00`) when several posts share a day, since ordering otherwise falls back to filename. Times are ordering metadata only — `postDate` renders in UTC and never shows them.
- `categories` are ids from the category index: a series (`m`), generation (`m5`), rank (`m5-max`), chip variant (`m5-max-18-40`), device category (`iphone`), group (`iphone/17`) or device (`iphone/17/e`, `mac/mac-mini`). They power the "Covers" footer and `relatedPosts` on hardware pages, so list every chip and device the post is actually about. A bad id only warns — check the build output for `[news] unknown category id`.
- The homepage shows the newest 3 via `postSnippets`; `/news/` paginates the rest.

House style, derived from the existing posts — match it:

- **The headline names the product and what it got**, e.g. "Mac mini and Mac Studio get M6, M5 Pro, M5 Max and M5 Ultra", "iPhone 17e launches with a new A19 variant", "Mac Pro discontinued; AirPods Max 2 arrives with H2". Not a headline about an analytical finding — the finding goes in the body.
- Open with the product, linked, and its release date. Details, comparisons and interpretation follow.
- **Link every chip, device and related post on first mention**, using `chip.url`-shaped paths: `/chips/m5/max/18-40/`, `/devices/mac/mac-mini/`, `/news/<slug>/`. Posts cross-link each other freely when one follows on from another.
- Use a Markdown comparison table when comparing two or more chips, with the variant linked in the first column and `18c CPU / 40c GPU` core notation in prose.
- Bold the one sharp fact per section (**36 GB, take it or leave it**), not whole paragraphs.
- Use `###` subheads once a post covers more than one product or theme; short posts don't need them.
- Close with what the release displaces: which models are discontinued, what a chip replaces, or what is left on older silicon.
- Every number must come from `src/_data/` or the source spec sheet. Check claims like "first" or "only" against the YAML before writing them.

### Search index

`src/api/search.njk` builds `/api/search.json`; `assets/search.js` consumes it. Every page on the site is indexed except device category and group pages.

- Entry shape: `{type, id, name, detail, url}` plus `weight` (devices and standalone pages) and `keywords`/`discontinued` (devices only).
- `type` is one of `family`, `tier`, `chip`, `series`, `device`, `news`, `page` — it selects the result icon (`assets/search.js`) and the ranking boost.
- Values are emitted with `| dump | safe`, **not** as HTML-escaped text. This is a JSON API and the client escapes for HTML itself; interpolating normally double-encodes apostrophes into `&#39;`.
- **Entry order is load-bearing.** `search.js` sorts by score with a stable sort, so ties fall back to file order. Emit broadest first: family → tier → variant → series → device → page.
- Devices carry `keywords` (the chip family names inside them) so a query like `m5` matches the Macs that ship one. Nothing else would match them.
- Ranking = match score (3 name-prefix / 2 name-contains / 1 other) + 0.1 for silicon types + the entry's `weight` (product line, minus 0.8 if discontinued).
- Skip a rank page only when its sole chip has no variant page of its own — otherwise the rank URL goes unindexed.
- News posts are indexed as `type: "news"` with `detail` set to the formatted date and `keywords` built from the summary plus each resolved category label — so "mac studio" finds a post that never says it in the title. They carry a negative weight so hardware outranks coverage of it.

### Common tasks

- Add a chip variant:
  1. Append to the appropriate `chips-*.yml` with `id`, `name`, `specs` in the correct order (newest gen → highest tier → most GPU cores).
  2. Add the `id` to the correct series `ranks[*].variants` in `series.yml`.
  3. If the chip is used in any device, add its `id` to `devices[*].variants` in `devices.yml`.
  4. Build and verify the comparison table.
- Add a news post:
  1. Create `src/news/posts/<slug>.md` with `title`, `date`, `summary`, `categories` — nothing else; the rest comes from `posts.11tydata.js`.
  2. Write it to the house style above, linking every chip and device mentioned.
  3. Build and check for `[news] unknown category id` warnings, then confirm it appears on `/` and `/news/`.
- Add a new field:
  1. Add to `specs.yml` with `key` + `label` in the right group.
  2. Populate `specs` for relevant chips. Missing values are fine.
  3. Tables update automatically via `groupedSpecs`.
- Add a new chip family file (e.g., `chips-s.yml`):
  1. Create `src/_data/chips-s.yml`.
  2. Update `.eleventy.js` to load it into `chipsCollection` and `chipPagesCollection`.
  3. Add a series entry in `series.yml`.
- Add a new device:
  1. Add it under the correct category in `devices.yml` with `id`, `name`, and `variants` (list of chip IDs).
  2. A page is automatically generated at `/devices/<category>/<device>/`.

### Do / Don't

- Do:
  - Keep YAML valid and typed (numbers, booleans, arrays, strings).
  - Use `getChips` and the comparison macro instead of hardcoding tables.
  - Use `deviceCarousel` macro for showing related devices.
  - Add fields to `specs.yml` rather than templates.
  - Ensure all `variants` reference existing chip `id`s.
  - Run `npm run lint` or `npm run build` to verify changes.
- Don't:
  - Reuse `id`s.
  - Add ad-hoc fields not declared in `specs.yml`.
  - Modify `groupedSpecs` in templates.
  - Hardcode device lists — use `getDevicesForChips` instead.
  - Assume a lower-binned chip variant is available in a device just because a higher-binned variant is. Only add chip IDs to `devices[*].variants` when confirmed by Apple's website or other authoritative sources.

### Quick checks

- `npm run build` completes successfully.
- `npm run lint` passes (Prettier formatting). Run it after any `npm install` — npm rewrites `package.json`/`package-lock.json` in its own style, which fails the check.
- No broken internal links: every `href` in `_site` resolves to a generated page.
- Tables include the expected fields and align across items.

### Guidance for Copilot

- Propose concise, file-scoped changes (YAML entries or small Nunjucks edits).
- Respect existing patterns for IDs, specs keys, and macro usage.
- When introducing a new series, remember to include it in `.eleventy.js` so chips appear in `chipsCollection` and `chipPagesCollection`.

### Keeping these instructions up to date

When making structural changes to the project, update `.github/copilot-instructions.md` in the same session:

- **New macro added** → add an entry under "Macros" with import path, usage example, and purpose.
- **New global helper or filter added** → add an entry under "Global helpers" or "Filters".
- **New collection added** → add a row to the Build-time behavior table and describe the enrichment.
- **New URL pattern or template added** → add a row to the URL structure table and a line under "Page templates".
- **New YAML convention introduced** (e.g. a new top-level field on devices) → document under "Device data conventions" or "Specs conventions".
- **New page type added** → also index it in `src/api/search.njk` and `src/sitemap.njk`, and link it from the footer site map in `base.njk`.
- **News writing conventions change** → update "News posts" so the house style stays the one source for it.
- **Common task changes** → update the relevant step in "Common tasks".

Keep descriptions short and grounded in actual code — the instructions are a quick reference, not prose documentation.
