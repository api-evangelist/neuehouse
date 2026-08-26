---
name: neuehouse-locate-houses
description: Retrieve NeueHouse house/location records and the membership, workspace and private-events pages, with the caveats an agent needs about which houses actually operate.
api: NeueHouse Content API
operations:
  - get_wp_v2_location
  - get_wp_v2_location_by_id
  - get_wp_v2_pages
  - get_wp_v2_search
generated: '2026-08-26'
method: generated
source: openapi/neuehouse-content-api-openapi.yml
---

# Find NeueHouse houses and membership information

Base URL: `https://www.neuehouse.com/wp-json`. Anonymous reads, no key.

## Steps

1. **List houses** — `get_wp_v2_location`

   `GET /wp/v2/location?per_page=100&_fields=id,slug,title,link`

   Three records exist:

   | id | slug | title | link |
   |---|---|---|---|
   | 6421 | `bradbury` | Bradbury | `https://bradbury.neuehouse.com` |
   | 210 | `los-angeles` | Hollywood | `/location/los-angeles/` |
   | 104 | `new-york` | Madison Square | `/location/new-york/` |

2. **Read the operating caveat before answering a user.**

   The content estate is **not** the operating estate. NeueHouse ceased operations at all locations
   in September 2025 and filed Chapter 7. Convene Hospitality Group acquired the brand and the
   **Madison Square** flagship out of bankruptcy and relaunched it in January 2026. The **Los Angeles
   houses were not acquired and are not reopening**, but their location records are still served by
   this API. Never tell a user a house is open on the strength of a `location` record alone.

3. **Fetch one house** — `get_wp_v2_location_by_id`

   `GET /wp/v2/location/{id}?_embed`

   Note: the `bradbury` record links to `https://bradbury.neuehouse.com`, which serves a real
   NeueHouse site but presents a `*.netlify.app` certificate. **TLS verification fails on that
   host.** Do not disable certificate verification to reach it — report the failure instead.

4. **Get membership and workspace information** — `get_wp_v2_pages`

   `GET /wp/v2/pages?per_page=100&_fields=id,slug,title,link`

   Fifteen pages, including `membership`, `workspace`, `private-events`, `house-rules`,
   `membership-agreement`, `privacy-policy`, `terms-conditions` and `accessibility`. Fetch one by id
   for its rendered `content`.

5. **Search across everything** — `get_wp_v2_search`

   `GET /wp/v2/search?search=<term>&per_page=20` returns mixed-type results with `id`, `type`,
   `subtype`, `title` and `url`.

## What this API will NOT give you

- **No pricing.** NeueHouse publishes no membership rates anywhere; the membership page routes to an
  application form and prices are disclosed only after applying. Do not infer or estimate a price.
- **No availability, booking or member data.** There is no reservation or account surface here.
- **No events detail.** See `neuehouse-browse-programming` for what the event type does and does not carry.

## Errors

WordPress envelope — `404 rest_no_route`, `400 rest_invalid_param`, `404 rest_post_invalid_id`,
`401 rest_cannot_create` on any write attempt. Full catalog in `errors/neuehouse-problem-types.yml`.
