<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Telegram API

[![SocQ Telegram API](https://img.shields.io/badge/API-SocQ%20Telegram%20API-F64C31)](https://socq.ai/apis/telegram?utm_source=github&utm_medium=readme&utm_campaign=socq-telegram-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/telegram/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/telegram-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/telegram-api/actions/workflows/check.yml)

6 endpoints for public Telegram data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Telegram APIs](https://socq.ai/apis/telegram?utm_source=github&utm_medium=readme&utm_campaign=socq-telegram-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-telegram-api) · [Documentation](https://docs.socq.ai/api-manual/telegram/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Telegram Channel Info API](#telegram-channel-info-api) | `telegram/channel-info` | Look up public Telegram channels by t.me link for title, description, avatar, verification badge and subscriber count. | 0.5 credits ($0.0025) per result | 2026-11-17 |
| [Telegram Channel Scraper API](#telegram-channel-scraper-api) | `telegram/channel-posts` | Collect a public Telegram channel's posts newest first, with text, publish time, views, reaction totals, photos and videos. | 0.3 credits ($0.0015) per result | 2026-11-17 |
| [Telegram Post Detail API](#telegram-post-detail-api) | `telegram/post-detail` | Get specific Telegram posts by link, from any public channels, with full text, publish time, views, reactions and media. | 0.5 credits ($0.0025) per result | 2026-11-17 |
| [Telegram Comments API](#telegram-comments-api) | `telegram/post-comments` | Collect up to 100 comments under a Telegram channel post from its discussion group, with author, text, time, reactions and reply target. | 0.3 credits ($0.0015) per result | 2026-11-17 |
| [Telegram Similar Channels API](#telegram-similar-channels-api) | `telegram/similar-channels` | Find the channels Telegram recommends next to a public channel, each with username, title, verification badge and subscriber count. | 0.5 credits ($0.0025) per result | 2026-11-17 |
| [Telegram Channel Search API](#telegram-channel-search-api) | `telegram/channel-search` | Search a public Telegram channel's full post history for a word or phrase and get the matching posts with text, date and views. | 0.5 credits ($0.0025) per result | 2026-11-17 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-telegram-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/telegram-api.git
cd telegram-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs telegram/channel-info

# Python 3.8+ (standard library only)
python3 examples/python/socq.py telegram/channel-info payloads/channel-info.json

# cURL + jq
bash examples/curl/socq.sh telegram/channel-info '{"urls":["https://t.me/durov","https://t.me/tginfo"]}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Telegram Channel Info API

Look up public Telegram channels by t.me link for title, description, avatar, verification badge and subscriber count.

- **Request:** `POST https://api.socq.ai/v1/telegram/channel-info`
- **publicId:** `telegram/channel-info`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/channel-info.json`](payloads/channel-info.json)

Look up public Telegram channels by their t.me link and get back the name, description, avatar, verification badge and subscriber count. It suits channel vetting before a sponsorship, keeping a watchlist of crypto or news channels up to date, or enriching a list of channel links collected elsewhere. The raw result file also carries the number of photos, videos and links each channel has shared.

Provide up to 50 public Telegram channel links in urls (t.me/name or t.me/s/name). Each channel found returns one row and costs 0.5 credits; links to channels that are not public are skipped, and the task fails with a refund only when none of them can be read.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `urls` | array of string | Yes | Links to public Telegram channels, up to 50, such as https://t.me/durov. (items 1 to 50) |

```bash
curl -X POST "https://api.socq.ai/v1/telegram/channel-info" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://t.me/durov",
    "https://t.me/tginfo"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/telegram/channel-info](https://docs.socq.ai/api-manual/telegram/channel-info) · Landing page: [socq.ai/apis/telegram/channel-info](https://socq.ai/apis/telegram/channel-info?utm_source=github&utm_medium=readme&utm_campaign=socq-telegram-api)

### Telegram Channel Scraper API

Collect a public Telegram channel's posts newest first, with text, publish time, views, reaction totals, photos and videos.

- **Request:** `POST https://api.socq.ai/v1/telegram/channel-posts`
- **publicId:** `telegram/channel-posts`
- **Price:** 0.3 credits ($0.0015) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/channel-posts.json`](payloads/channel-posts.json)

Read a public channel's feed from the latest post backwards. Every post comes with its text, publish time, view count, the total of its emoji and Star reactions, and its attached photos and videos. Pull the last few days for a daily digest, or set a larger limit to back-fill a channel's history for analysis.

Provide one public Telegram channel link in url. Posts are returned newest first; results_limit defaults to 20 and accepts integers from 1 to 1000. Each post costs 0.3 credits, and a channel that cannot be read fails with a refund.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `results_limit` | integer | No | (value 1 to 1000; default `20`) |
| `url` | string | Yes | Link to a public Telegram channel, such as https://t.me/durov. |

```bash
curl -X POST "https://api.socq.ai/v1/telegram/channel-posts" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://t.me/durov",
  "results_limit": 20
}'
```

Full docs: [https://docs.socq.ai/api-manual/telegram/channel-posts](https://docs.socq.ai/api-manual/telegram/channel-posts) · Landing page: [socq.ai/apis/telegram/channel-posts](https://socq.ai/apis/telegram/channel-posts?utm_source=github&utm_medium=readme&utm_campaign=socq-telegram-api)

### Telegram Post Detail API

Get specific Telegram posts by link, from any public channels, with full text, publish time, views, reactions and media.

- **Request:** `POST https://api.socq.ai/v1/telegram/post-detail`
- **publicId:** `telegram/post-detail`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/post-detail.json`](payloads/post-detail.json)

When you already know which posts matter, pass their links and get each one back in full: text, publish time, views, total reactions, photos and videos. Posts from different channels can be mixed in one request, which makes it handy for refreshing the numbers on a set of tracked announcements or campaign posts. The raw result file keeps the per-emoji reaction counts and the forwarding source.

Provide up to 50 Telegram post links in urls, in the form t.me/channel/post_id. Each post found returns one row and costs 0.5 credits. Posts that have been deleted or sit in a private channel are skipped; the task fails with a refund only when none of them can be read.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `urls` | array of string | Yes | Links to posts in public Telegram channels, up to 50, such as https://t.me/durov/510. (items 1 to 50) |

```bash
curl -X POST "https://api.socq.ai/v1/telegram/post-detail" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://t.me/durov/510"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/telegram/post-detail](https://docs.socq.ai/api-manual/telegram/post-detail) · Landing page: [socq.ai/apis/telegram/post-detail](https://socq.ai/apis/telegram/post-detail?utm_source=github&utm_medium=readme&utm_campaign=socq-telegram-api)

### Telegram Comments API

Collect up to 100 comments under a Telegram channel post from its discussion group, with author, text, time, reactions and reply target.

- **Request:** `POST https://api.socq.ai/v1/telegram/post-comments`
- **publicId:** `telegram/post-comments`
- **Price:** 0.3 credits ($0.0015) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/post-comments.json`](payloads/post-comments.json)

Channels that link a discussion group let readers comment under each post. This endpoint reads that thread for one post and returns each comment with its author name, text, time, reaction total and the comment it answers, so you can see how a community reacted to an announcement or pull feedback into a support or research workflow.

Provide one Telegram post link in url. Comments come from the discussion group linked to the channel, and you pay only for the comments returned. results_limit defaults to 100, which is also the maximum. Each comment costs 0.3 credits.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `results_limit` | integer | No | (value 1 to 100; default `100`) |
| `url` | string | Yes | Link to a post in a public Telegram channel, such as https://t.me/durov/510. |

```bash
curl -X POST "https://api.socq.ai/v1/telegram/post-comments" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://t.me/tginfo/4455",
  "results_limit": 100
}'
```

Full docs: [https://docs.socq.ai/api-manual/telegram/post-comments](https://docs.socq.ai/api-manual/telegram/post-comments) · Landing page: [socq.ai/apis/telegram/post-comments](https://socq.ai/apis/telegram/post-comments?utm_source=github&utm_medium=readme&utm_campaign=socq-telegram-api)

### Telegram Similar Channels API

Find the channels Telegram recommends next to a public channel, each with username, title, verification badge and subscriber count.

- **Request:** `POST https://api.socq.ai/v1/telegram/similar-channels`
- **publicId:** `telegram/similar-channels`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/similar-channels.json`](payloads/similar-channels.json)

Telegram shows a row of related channels on every public channel. Give it one channel link and get that list back, usually around ten channels, each with its username, title, verification badge and subscriber count. Start from a competitor or a channel you already advertise on, and use the results to widen a media list or map a niche.

Provide one public Telegram channel link in url. Each recommended channel returns one row and costs 0.5 credits, in the order Telegram lists them.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `url` | string | Yes | Link to a public Telegram channel, such as https://t.me/durov. |

```bash
curl -X POST "https://api.socq.ai/v1/telegram/similar-channels" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://t.me/durov"
}'
```

Full docs: [https://docs.socq.ai/api-manual/telegram/similar-channels](https://docs.socq.ai/api-manual/telegram/similar-channels) · Landing page: [socq.ai/apis/telegram/similar-channels](https://socq.ai/apis/telegram/similar-channels?utm_source=github&utm_medium=readme&utm_campaign=socq-telegram-api)

### Telegram Channel Search API

Search a public Telegram channel's full post history for a word or phrase and get the matching posts with text, date and views.

- **Request:** `POST https://api.socq.ai/v1/telegram/channel-search`
- **publicId:** `telegram/channel-search`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/channel-search.json`](payloads/channel-search.json)

Find every time a channel talked about something, however far back it was posted. The search runs over the channel's whole history and returns the matching posts with their text, date, views and reactions. Check whether a project was ever mentioned by an influential channel, or collect all of a channel's posts about a product launch.

Provide one public Telegram channel link in url and the words to look for in query. results_limit defaults to 20 and accepts integers from 1 to 100. Each matching post costs 0.5 credits; a search with no match costs nothing. Photos in search results link to the post that shows them.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `query` | string | Yes | Words to look for in the channel's posts. |
| `results_limit` | integer | No | (value 1 to 100; default `20`) |
| `url` | string | Yes | Link to a public Telegram channel, such as https://t.me/durov. |

```bash
curl -X POST "https://api.socq.ai/v1/telegram/channel-search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://t.me/durov",
  "query": "TON",
  "results_limit": 20
}'
```

Full docs: [https://docs.socq.ai/api-manual/telegram/channel-search](https://docs.socq.ai/api-manual/telegram/channel-search) · Landing page: [socq.ai/apis/telegram/channel-search](https://socq.ai/apis/telegram/channel-search?utm_source=github&utm_medium=readme&utm_campaign=socq-telegram-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Telegram data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
