---
name: jnana-therapeutics-news-monitoring
description: >-
  Monitor Jnana Therapeutics press releases, media coverage, presentations and insights from the
  company's own content API, filtered by category and date, without scraping HTML.
api: jnana-therapeutics:jnana-therapeutics-content-api
base_url: https://www.jnanatx.com/wp-json
operations:
  - listPosts
  - getPost
  - listCategories
  - searchContent
generated: '2026-08-23'
method: generated
source: openapi/jnana-therapeutics-content-openapi.yml
---

# Monitor Jnana Therapeutics news

Jnana Therapeutics publishes no developer program. Its news archive is nonetheless available as
structured JSON through the WordPress REST content API behind the marketing site, which is a far
better source than parsing the rendered `/news/` page.

Everything below is read-only. There is no credential to obtain and none is accepted.

## 1. Learn the categories once

    GET https://www.jnanatx.com/wp-json/wp/v2/categories?per_page=100

Operation: `listCategories`. At harvest time (2026-08-23) the archive used seven terms:

| id | slug | posts |
|----|------|-------|
| 5  | press-releases | 43 |
| 6  | in-the-news | 16 |
| 13 | presentations | 3 |
| 7  | insights | 2 |
| 12 | publications | 0 |
| 14 | blogs | 0 |
| 15 | videos | 0 |

Ids are per-deployment. Re-read them rather than hardcoding, because a WordPress term can be
renumbered.

## 2. Pull the window you care about

    GET /wp/v2/posts?categories=5&after=2026-01-01T00:00:00&per_page=100&orderby=date&order=desc&_fields=id,date,slug,link,title,categories

Operation: `listPosts`.

- `categories` takes the ids from step 1. Omit it for the whole archive.
- `after` / `before` window on publish date; `modified_after` / `modified_before` window on last
  edit, which is what you want if you are watching for silent corrections to a press release.
- `_fields` trims the response. Without it every item carries the full rendered HTML body plus a
  `yoast_head` blob, which is roughly two orders of magnitude more bytes than a headline feed
  needs.
- `per_page` is capped at 100.

## 3. Page correctly

Read `X-WP-Total` and `X-WP-TotalPages` from the response headers, or follow
`Link: <...page=2>; rel="next"`. Do not increment `page` blindly — a page number past the end
returns `400 rest_post_invalid_page_number`, not an empty array.

## 4. Fetch a full item only when you need the body

    GET /wp/v2/posts/{id}

Operation: `getPost`. Add `?_embed` to inline the author record, the featured image and the
category terms in one round trip instead of three.

## 5. Search when you do not know the category

    GET /wp/v2/search?search=phenylketonuria&per_page=20

Operation: `searchContent`. This crosses posts, pages and team profiles (73 objects indexed at
harvest time) and returns id/title/url/type/subtype. Follow the `id` back into `getPost` or
`getPage` for the body.

## Rules

- **Pace yourself.** No rate-limit header of any family is returned, so you get no runtime signal
  and no warning before an edge rule fires. `robots.txt` sets `Crawl-delay: 10`; treat ten seconds
  between requests as the contract. Responses are edge-cached for 600 seconds, so polling faster
  than that returns the same bytes anyway.
- **Errors are not RFC 9457.** The envelope is `{"code": "...", "message": "...", "data": {"status": N}}`.
  Branch on `code`, not on the message string. See `errors/jnana-therapeutics-problem-types.yml`.
- **Do not attempt writes.** An OPTIONS preflight on `/wp/v2/posts` returns `allow: GET`. Any
  POST/PUT/DELETE will return `401 rest_forbidden`, and retrying will not change that — there is no
  public credential path.
- **Do not harvest `/wp/v2/users`.** It is anonymously readable and returns WordPress account
  display names and slugs. It is not part of this task and it is not content the company chose to
  publish.
- **Attribute correctly.** This is a company-operated marketing site, not a data product. Jnana
  Therapeutics has been a subsidiary of Otsuka America since 2024-09-23; press releases dated after
  that are frequently issued by Otsuka about Jnana assets rather than by Jnana.
