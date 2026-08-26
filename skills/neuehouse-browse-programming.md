---
name: neuehouse-browse-programming
description: Browse and search NeueHouse cultural programming (events), press coverage and houses through the public WordPress content API.
api: NeueHouse Content API
operations:
  - get_wp_v2_event
  - get_wp_v2_event_by_id
  - get_wp_v2_categories
  - get_wp_v2_media_by_id
generated: '2026-08-26'
method: generated
source: openapi/neuehouse-content-api-openapi.yml
---

# Browse NeueHouse programming

NeueHouse publishes its cultural programming through the WordPress REST API on its own host.
Reads are anonymous — no key, no signup. Base URL: `https://www.neuehouse.com/wp-json`.

## Before you start

- There is **no API key and no developer account**. Do not look for one.
- Everything here is **read-only**. Every write route returns `401 rest_cannot_create` unless you
  hold a WordPress Application Password, which NeueHouse issues only internally.
- There are **no published rate limits and no rate-limit headers**. Be conservative: the site's
  `robots.txt` asks for a 10-second crawl delay, which is a reasonable pace to respect.

## Steps

1. **List events** — `get_wp_v2_event`

   `GET /wp/v2/event?per_page=100&page=1&_fields=id,slug,title,date,link,featured_media,categories`

   Use `_fields` to keep payloads small. The collection held **467** events when this skill was
   written. Read `X-WP-Total` and `X-WP-TotalPages` from the response headers and page until you
   reach `X-WP-TotalPages` — requesting a page beyond it returns `400 rest_post_invalid_page_number`.

2. **Narrow the set** — same operation, more parameters

   - `?search=<term>` for full text
   - `?after=2026-01-01T00:00:00&before=2026-12-31T23:59:59` for a date window (ISO 8601)
   - `?categories=<id>` to filter by taxonomy — get ids from `get_wp_v2_categories`
   - `?orderby=date&order=desc` for most recent first

3. **Fetch one event** — `get_wp_v2_event_by_id`

   `GET /wp/v2/event/{id}`

   Add `?_embed` to inline the featured image and taxonomy terms in one round trip instead of
   following `_links` yourself.

4. **Resolve imagery** — `get_wp_v2_media_by_id`

   `GET /wp/v2/media/{featured_media}` returns `source_url`, `alt_text` and `media_details.sizes`.
   Prefer `?_embed` on step 3 over calling this separately.

## What this API will NOT give you

State this plainly rather than guessing:

- The `event` record carries **no `content` and no `excerpt` field** — there is no body copy.
- The `acf` object on event records was observed **empty**. Start times, venue detail and ticket
  links are **not** retrievable here. Send users to the event's `link` for those, or to the
  Luma page at `https://luma.com/neuehouse`.
- The `neuejournal` and `posts` collections answer `200` but are **empty** (`X-WP-Total: 0`).

## Errors

All errors use the WordPress envelope, not RFC 9457:

```json
{"code":"rest_no_route","message":"No route was found matching the URL and request method.","data":{"status":404}}
```

- `404 rest_no_route` — wrong path or wrong method; re-check against `/wp-json/`.
- `400 rest_invalid_param` — read `data.params` for the offending argument.
- `400 rest_post_invalid_page_number` — you paged past `X-WP-TotalPages`.
- `401 rest_cannot_create` — you attempted a write. There is no third-party write access.

See `errors/neuehouse-problem-types.yml` for the full catalog.
