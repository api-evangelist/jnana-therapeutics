---
name: jnana-therapeutics-company-profile
description: >-
  Assemble a factual profile of Jnana Therapeutics — the RAPID chemoproteomics platform, the
  programs, the leadership and advisory roster — from the company's own content API rather than
  from third-party summaries.
api: jnana-therapeutics:jnana-therapeutics-content-api
base_url: https://www.jnanatx.com/wp-json
operations:
  - listPages
  - getPage
  - listTeamMembers
  - getTeamMember
  - listTeamDepartments
  - getMediaItem
generated: '2026-08-23'
method: generated
source: openapi/jnana-therapeutics-content-openapi.yml
---

# Build a Jnana Therapeutics company profile from primary source

Use the company's own content API. Every statement you produce this way is traceable to a URL Jnana
published, which is the difference between a profile and a summary of summaries.

## 1. Enumerate the pages

    GET https://www.jnanatx.com/wp-json/wp/v2/pages?per_page=20&_fields=id,slug,link,title

Operation: `listPages`. Nine pages at harvest time (2026-08-23):

| id | slug | what it holds |
|----|------|---------------|
| 7  | home | positioning |
| 14 | rapid-platform | the RAPID chemoproteomics platform, in detail |
| 17 | programs | PKU and immune-mediated disease pipeline |
| 19 | team | roster landing page |
| 21 | join-us | careers |
| 23 | news | archive landing page |
| 25 | contact-us | address and general inquiries |
| 3  | privacy-policy | privacy policy |
| 35 | terms-of-use | terms of use |

## 2. Read the two pages that carry the science

    GET /wp/v2/pages/14    # RAPID platform
    GET /wp/v2/pages/17    # Programs

Operation: `getPage`. The body arrives as rendered HTML in `content.rendered`. Strip tags before
summarising; the pages carry inline SVG style blocks that will pollute a naive text extraction.

## 3. Pull the roster from the custom post type, not the HTML page

    GET /wp/v2/team-department?per_page=100
    GET /wp/v2/team?per_page=100&_fields=id,slug,link,title,team-department&_embed

Operations: `listTeamDepartments`, `listTeamMembers`. `team` is a custom post type registered by
this deployment, and `team-department` is its taxonomy — four terms at harvest time, including
Leadership with five members. Nine profiles were published.

`getTeamMember` returns the full biography in `content.rendered`. `_embed` inlines the department
terms and the headshot so you do not need `getMediaItem` separately; use `getMediaItem` only when
you want the `media_details` size variants.

## 4. Cross-check the corporate facts

The company's ownership is stated in its own press release, not inferred:

    GET /wp/v2/posts?slug=otsuka-pharmaceutical-completes-acquisition-of-jnana-therapeutics-inc

Otsuka Pharmaceutical completed the acquisition on 2024-09-23 for $800M plus up to $325M in
development and regulatory milestones; Jnana became a direct subsidiary of Otsuka America, Inc.

## Rules

- **Counts change.** Every number above was read from `X-WP-Total` on 2026-08-23. Re-read it; do
  not restate a stale count as current.
- **Do not present the content API as a Jnana product.** It is the WordPress surface behind the
  marketing site. Jnana publishes no developer program, no API documentation and no SDK, and
  `api.jnanatx.com`, `docs.jnanatx.com` and `developer.jnanatx.com` do not resolve.
- **Do not pull `/wp/v2/users`.** Author accounts are not part of a company profile.
- **Respect the pacing.** `Crawl-delay: 10` in robots.txt is the only published pacing signal; no
  rate-limit headers are returned.
- **Errors** use the WordPress envelope `{"code","message","data":{"status"}}`, not RFC 9457.
