# Video Update Instructions

Use this guide when updating the public video and speaking content for `mlogdberg.com`.

## Goal

Keep the site's video content current in a way that improves:

- credibility
- discoverability
- speaker visibility
- proof of current activity

The site should show both:

- a small curated `Featured` set with the strongest proof content
- a broader `Timeline` of talks, interviews, podcasts, and video appearances

## Where The Content Lives

Primary files:

- `_data/videos.yml`
- `_pages/videos.md`
- `_pages/about.md`
- `assets/css/style.css`

How the site uses them:

- `_data/videos.yml` is the source of truth for the video list
- `_pages/videos.md` renders:
  - `Featured` videos first
  - `Timeline` after that
- `_pages/about.md` shows only a short featured-video section

## Current Sorting Rules

Featured:

- controlled manually with `featured: true`
- intended for best proof, not strict chronology

Timeline:

- shows only entries where `featured != true`
- sorted by `date` descending when `date` exists
- falls back to `year` display when exact date is not available

Current template behavior is in `_pages/videos.md`.

## Video Entry Schema

Each item in `_data/videos.yml` should use these fields where available:

```yml
- title: Example title
  source: YouTube
  year: 2026
  date: 2026-04-16
  why: Why this item matters for Mattias's credibility or positioning.
  description: One short plain-English description of the video content.
  watch_url: https://www.youtube.com/watch?v=example
  link_label: Watch on YouTube
  note: Optional confidence note about verification or source quality.
  featured: false
```

Field guidance:

- `title`: Use the public title unless it is clearly broken
- `source`: Keep short and recognizable, for example `YouTube`, `Turbo360 Podcast`, `DevUP Talks`, `Microsoft Reactor`
- `year`: Always include
- `date`: Use exact date only when verified
- `why`: Explain why this strengthens Mattias's expert profile
- `description`: One short summary, not marketing fluff
- `watch_url`: Best public destination available
- `link_label`: Must match the link type honestly
- `note`: Use when the source is indirect or confidence is limited
- `featured`: Only `true` for the strongest items

## Link Quality Rules

Prefer links in this order:

1. direct YouTube watch URL
2. direct video page with embedded player
3. official event/session page
4. event recap or archive page

Do not pretend an archive page is a direct video link.

Use clear labels:

- `Watch on YouTube`
- `Watch video`
- `Open video page`
- `Open event page`
- `Read recap`
- `Open archive listing`

Use `note` when the public source confirms the session but not a direct watch URL.

## Trusted Sources

Start with these sources first.

High-confidence sources:

- Mattias YouTube channel: `https://www.youtube.com/@mlogdberg/featured`
- DevUP Talks playlist: `https://www.youtube.com/playlist?list=PL7XW6cBZ47dEJBvo9arzUi5r8qVpobba7`
- DevUP website: `https://www.devup.solutions`
- YouTube playlist feed:
  - `https://www.youtube.com/feeds/videos.xml?playlist_id=PL7XW6cBZ47dEJBvo9arzUi5r8qVpobba7`
- Turbo360 podcast pages
- Microsoft Reactor event pages
- Microsoft Learn show pages
- Integration User Group pages
- BizTalk360 / Integrate event and recap pages
- Mattias's own blog posts when they contain embedded video IDs

Secondary sources:

- Contica interview pages
- Sessionize speaker profile
- SlideShare decks only as supporting evidence, not primary video proof

## Recommended Search Process

When looking for new or missing videos, search in this order:

1. Check the YouTube channel and playlist first
2. Check the YouTube playlist feed for machine-readable entries
3. Search Mattias's own blog for embedded YouTube videos
4. Search official event pages
5. Search recap/archive pages only if direct video pages are not available

## Known Good Retrieval Tactics

These worked during the last update cycle:

- YouTube playlist feed can expose exact titles, dates, and watch IDs even when the normal channel HTML is noisy:

```text
https://www.youtube.com/feeds/videos.xml?playlist_id=PL7XW6cBZ47dEJBvo9arzUi5r8qVpobba7
```

- Older blog posts can reveal embedded YouTube IDs for direct watch links
- Integration User Group pages may expose video pages even when direct YouTube links are not obvious

## Known Issues

- YouTube channel HTML may show a cookie/consent wall when fetched automatically
- Prefer `https://www.devup.solutions` over `https://devup.solutions`
- The DevUP site appears to be a JavaScript app, so some fetches may only return the shell page
- Older conference sessions often have only recap or archive confirmation, not direct video URLs
- Some entries have only `year` because exact dates were not verified

## Content Strategy

Add to `Featured` only if the item is one of these:

- recent and clearly relevant to Mattias's current positioning
- strong technical proof of expertise
- high-quality public talk or interview
- likely to help a first-time visitor trust the site quickly

Good `Featured` candidates tend to be:

- recent YouTube talks
- Microsoft Reactor / Microsoft Learn sessions
- strong Turbo360 podcast appearances

Keep older conference material in the timeline unless it is unusually important.

## What To Look For Next

Priority updates:

- new DevUP Talks playlist items
- new videos published on Mattias's YouTube channel
- direct watch URLs for currently indirect entries
- stronger exact dates for older conference talks
- better direct links for:
  - `Integration Love Story med Mattias Lögdberg`
  - older Integrate sessions
  - older Integration User Group sessions

## Update Workflow

When asked to update videos:

1. Read this file
2. Inspect `_data/videos.yml`
3. Verify new sources on the web
4. Prefer replacing weak archive links with stronger direct video links
5. Add or update entries in `_data/videos.yml`
6. Keep `Featured` curated
7. Keep timeline entries in good date order by adding `date` where possible
8. Update `_pages/videos.md` only if the rendering model needs to change
9. Update `_pages/about.md` only if the featured set changes materially
10. Summarize:
    - what was added
    - what was upgraded from indirect to direct links
    - what still needs confirmation

## How To Ask A Future Agent

Tell the agent something like:

`Read VIDEO_UPDATE_INSTRUCTIONS.md and update the video list for this site.`

Or:

`Follow VIDEO_UPDATE_INSTRUCTIONS.md, check the YouTube channel and DevUP Talks playlist, and add any new verified videos to _data/videos.yml.`

## Current Important Verified Sources

These were verified during the last update pass:

- DevUP Talks playlist feed:
  - `https://www.youtube.com/feeds/videos.xml?playlist_id=PL7XW6cBZ47dEJBvo9arzUi5r8qVpobba7`
- DevUP Talks playlist:
  - `https://www.youtube.com/playlist?list=PL7XW6cBZ47dEJBvo9arzUi5r8qVpobba7`
- Mattias YouTube channel:
  - `https://www.youtube.com/@mlogdberg/featured`
- Turbo360:
  - `https://turbo360.com/podcast/managed-identities-with-mattias-logdberg`
  - `https://turbo360.com/podcast/azure-integration-and-security-challenges-and-best-practices`
  - `https://turbo360.com/podcast/inside-devops-enhancing-integration-environments-and-security`
- Microsoft:
  - `https://developer.microsoft.com/en-us/reactor/events/19230/`
  - `https://learn.microsoft.com/en-us/shows/logic-apps-community-day-2024/master-paas-networking-for-ais`
- Integration User Group:
  - `https://www.integrationusergroup.com/azure-integration-dtap-series-how-to-go-from-development-to-production-part-1-api-management/`
  - `https://www.integrationusergroup.com/azure-integration-dtap-series-how-to-go-from-development-to-production-part-2-azure-logic-apps/`
  - `https://www.integrationusergroup.com/past-events/`
- Integrate / BizTalk360:
  - `https://www.biztalk360.com/integrate-2018/`
  - `https://www.biztalk360.com/integrate-2020-remote/`
  - `https://www.biztalk360.com/blog/integrate-2021-remote-day-3-recap/`
  - `https://www.biztalk360.com/blog/integrate-2022-day-3-recap/`
- Contica:
  - `https://contica.se/integration-love-story-with-mattias-logdberg/`
