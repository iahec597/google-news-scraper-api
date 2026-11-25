# Google News Scraper API: What Fields Come Back, What Does a Request Cost, and Which Plan Do You Need?

Google doesn't publish an open search API for Google News, so every "Google News scraper API" on the market does the same job: fetch news.google.com, parse the results page, and hand you structured JSON. ScraperAPI ships a dedicated structured endpoint for exactly that at `/structured/google/news`, which puts it in a different category from a generic proxy that returns raw HTML.

The number that decides your bill is 25. Google-family requests bill at 25 credits per page, so a plan's credit allowance buys far fewer Google News pages than the headline figure suggests. That multiplier, the parameter set, and the JSON shape are what this page covers.

> **Quick pick:** start on ScraperAPI's free plan. It gives you 1,000 credits with a 5-connection concurrency cap, which at 25 credits per Google News page works out to **40 requests** — enough to confirm the response shape, the market codes and the date filters against your own pipeline before you pay for anything.
> 👉 [Check the current ScraperAPI plans and credit allowances](https://scraperapi.com/pricing?fp_ref=coupons)

## The plans, and what each one actually controls

ScraperAPI sells eight tiers. Paid tiers are priced by monthly credit allowance rather than by request, and the live dollar figure for each one sits on the pricing page; the structure below is what decides whether a tier is large enough for your job.

| Plan | What's confirmed about it | Billing mechanics | Get it |
| --- | --- | --- | --- |
| Free | 1,000 API credits, maximum 5 concurrent connections | Free | [Start on the free allowance](https://scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | Auto-upgrade when credits run out (opt-in) | Monthly credits, annual option | [View the Hobby tier](https://scraperapi.com/pricing?fp_ref=coupons) |
| Startup | Auto-upgrade when credits run out (opt-in) | Monthly credits, annual option | [View the Startup tier](https://scraperapi.com/pricing?fp_ref=coupons) |
| Business | Auto-upgrade when credits run out (opt-in) | Monthly credits, annual option | [View the Business tier](https://scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | Auto-upgrade **and** pay-as-you-go up to a monthly cap | Monthly credits, annual option, PAYG | [View the Scaling tier](https://scraperapi.com/pricing?fp_ref=coupons) |
| Professional | Auto-upgrade **and** pay-as-you-go up to a monthly cap | Monthly credits, annual option, PAYG | [View the Professional tier](https://scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | Pay-as-you-go up to a monthly cap | Monthly credits, annual option, PAYG | [View the Advanced tier](https://scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom pricing applied across your usage; Ultra Premium requests carry their own minimum rate | Contracted through sales | [See what Enterprise covers](https://scraperapi.com/pricing?fp_ref=coupons) |

Three mechanics matter more than the tier names:

**Auto-upgrade** moves you to the next plan the moment credits run out, and it's opt-in on Hobby through Professional. Useful if you'd rather absorb a spike than have a nightly job fail. Bad if you want a hard spending ceiling.

**Pay-as-you-go** keeps you on the same plan past your limit at a fixed per-credit rate, capped at a monthly number you set. It's available on Scaling, Professional, Advanced and Enterprise. For a news-monitoring job whose volume swings with breaking events, this is the setting that prevents both silent failures and surprise overage.

**Annual plans can renew early** for a fresh batch of credits. If a quarterly crawl needs more headroom than a monthly cycle gives you, that's the lever.

## The number that decides your bill: 25 credits per page

Credits are not one-per-request. ScraperAPI prices each domain class separately, and Google sits near the top of the list.

| Request type | Credits per request |
| --- | --- |
| Standard page | 1 |
| Amazon | 5 |
| Google and Bing, including subdomains | **25** |
| LinkedIn | 30 |
| Sites behind Cloudflare, DataDome or PerimeterX, when bypassed | +10 on top of the base cost |

Google News pages fall in the Google family, so budget 25 credits each. That reframes every plan comparison. The free allowance is 40 Google News fetches, not 1,000. A 5-connection concurrency cap also caps throughput regardless of how many credits you hold, which matters if you're polling dozens of queries on a schedule.

Two controls keep this predictable. The **Domain Cost Estimator** in your dashboard returns the exact credit cost for any URL you plan to hit, so you can confirm the Google News endpoint's price on your account rather than trusting a table. And **`max_cost`** caps a single scrape, so one expensive URL can't quietly drain a day's allowance.

If your monthly Google News volume is N pages, your credit requirement is roughly N × 25. That's the only arithmetic you need before choosing a tier.
👉 [See how many credits each plan includes](https://scraperapi.com/pricing?fp_ref=coupons)

## Calling the endpoint: synchronous GET or async POST with webhooks

There are two ways in. The synchronous structured endpoint takes GET parameters on `api.scraperapi.com/structured/google/news`, with `api_key` and `query` required. The async endpoint at `async.scraperapi.com/structured/google/news` takes a JSON POST and accepts a `queries` array instead of a single `query`, plus a `callback` object pointing at a webhook.

That `queries` array is the practical difference. A monitoring job tracking twenty brand terms can submit them in one async request and receive results by webhook, rather than opening twenty synchronous connections against a 5-connection limit. If your pipeline is built around request/response, the async model means restructuring around a callback handler. Worth knowing before you write the client.

## Parameters that change your results

The endpoint mirrors a set of Google parameters rather than exposing a single keyword field.

| Parameter | What it does |
| --- | --- |
| `query` / `queries` | Search keywords; `queries` array only on the async endpoint |
| `tld` | Which Google edition to hit. Defaults to `us` (google.com). Also `uk`, `ca`, `de`, `es`, `fr`, `it`, `jp`, `in`, `cn`, `sg`, `mx`, `ae`, `br`, `nl`, `au`, `tr`, `sa`, `se`, `pl` |
| `country_code` | Used together with `tld` when you need the page as seen from another country |
| `hl` / `gl` | Interface language; boosts matches from a given country |
| `start` | Offset into the result list; `start=10` makes the tenth result the first returned |
| `num` | Number of results |
| `time_period` | Recency window: `1H`, `1D`, `1W`, `1M`, `1Y` |
| `date_range_start` / `date_range_end` | Explicit window in `YYYY-MM-DD`, must be used as a pair |
| `output_format` | `json` (default) or `csv` |
| `UULE` | Encoded region string, for example `w+CAIQICINUGFyaXMsIEZyYW5jZQ` |
| `IE` / `OE` | Input and output character encoding |

The `tld` list is the part most teams under-check. Twenty markets are documented, and localisation is a two-parameter job: `tld` picks the edition, `country_code` and `UULE` pin the geography. Running `hl=en` against `tld=jp` returns a different result set than `hl=ja`, and neither matches what a reader in Tokyo actually sees without a `UULE` or `country_code`.

Recency filtering is handled by `time_period` and the date-range pair. For a news-monitoring pipeline, that's usually what you want instead of a sort switch: "everything published in the last 24 hours" is a cleaner filter than "sorted by date, top 100," because it can't silently return stale results when a topic goes quiet.

## What you get back

ScraperAPI's structured response returns JSON with a `search_information` block (`query_displayed`, `total_results`, `time_taken_displayed`) and an `articles` array. The article-level keys are documented on the endpoint page, and they're worth opening before you design a schema, because the category has not settled on one.

SerpApi's `news_results` shape is the closest thing to a convention here, and it shows what a usable news record looks like:

| Field | Type | Why it matters |
| --- | --- | --- |
| `position` | integer | Rank of the article in the result set |
| `title` | string | Headline |
| `link` | string | Article URL |
| `source` | object | `name`, `icon`, `authors[]` |
| `thumbnail` / `thumbnail_small` | string | Image URLs |
| `date` | string | Relative publication time ("2 hours ago") |
| `iso_date` | string | ISO timestamp — the field you actually store |
| `snippet` | string | Short excerpt |
| `stories` | array | Nested related coverage of the same event |
| `story_token` | string | Token for pulling the full coverage cluster |

Two things in that list are the difference between a demo and a database. `iso_date` is sortable and storable; the relative `date` string is neither. And the `stories` / `story_token` pair gives you event clustering, so five outlets covering one announcement arrive as one story with five articles rather than five unrelated rows.

The response envelope carries navigation metadata too: `menu_links` and `sub_menu_links` for topics and sub-topics, `related_topics`, `related_publications`. If you're building a topic browser rather than a keyword monitor, those lists save you a separate discovery step.

ScraperAPI returns the same information in a different envelope. Budget a normalisation layer if you're comparing providers or planning to switch: the array is called `articles` rather than `news_results`, and the metadata block sits at `search_information`. Apify's Google News actor also uses `news_results`, which tells you which naming your downstream code will most likely inherit.

## news.google.com is not the Google Search News tab

Worth stating plainly because it changes your acceptance tests. SerpApi draws the distinction in its own docs: its Google News scraper hits the news.google.com site, not the News tab inside Google Search. The two produce different result sets from the same query.

The same distinction applies to any scraper in this category, including ScraperAPI's `/structured/google/news` endpoint. If your requirements doc says "the News tab on google.com/search," verify which surface the endpoint actually parses before you sign off on output quality. The mismatch usually shows up as missing stories your stakeholders saw with their own eyes.

## Where the other services in this category differ

All five options below solve the same underlying problem. They differ in how you call them and what they hand back.

| Service | Integration form | Distinguishing detail |
| --- | --- | --- |
| SerpApi | Single GET with `engine=google_news`; official Python and JS packages, plus cURL | `q` accepts `site:` and `when:` operators; `so=0/1` switches between relevance and date ordering; `topic_token`, `publication_token`, `section_token` and `story_token` address topics, publishers and single stories by token |
| Oxylabs | Command line: `make scrape TOPIC_ID=<topic_id>` | Topics are addressed by long token IDs, not keywords — a Business topic ID runs to a 60-character string |
| ScrapingBee | Python, Node and cURL samples | Returns title, source, snippet and date as JSON |
| Apify | Python via `uv`, plus an MCP server | One item per page of results, each carrying the search parameters, search metadata (total results, pages processed) and a `news_results` array |
| lewisdonovan/google-news-scraper | Lightweight scraper, plus an MCP server launched with `npx -y google-news-scraper-mcp` | Lets Claude, Cursor and other MCP clients query Google News directly |

The MCP detail is a real signal about how these tools get used now. Two of the five ship an MCP server, which means the consumer is increasingly an AI agent doing research rather than a cron job writing rows. If you're wiring Google News into an assistant, that changes what "good output" means: fewer fields, faster response, looser schema.

The parameter sets also split cleanly. SerpApi gives you date *ordering* (`so=1`) and story clustering tokens. ScraperAPI gives you explicit recency windows (`time_period`, `date_range_start`/`date_range_end`) and a documented list of twenty Google editions. If newest-first ordering is a hard requirement in your spec, confirm it against the endpoint's actual parameter list before building the pipeline around it.

## Who shouldn't pay for this

If you need thirty headlines a day for a personal reading list, an open-source scraper and a cron job will do it, and no credit maths is involved. Same for a one-off research task: a manual export or a single afternoon with a free tier beats a subscription you'll forget to cancel.

The paid case starts when three things are true at once. You need stable field names across months, because your schema and dashboards depend on them. You need localisation, because your market isn't the US and `tld` plus `hl` plus `UULE` is doing real work. And you need the job to keep running unattended, which is what concurrency limits, `max_cost` and pay-as-you-go actually buy you.

## Picking a tier

Match the tier to your monthly Google News page count times 25 credits, then check whether your queries run in parallel.

- **Under 40 pages total, one-time validation** → free plan, no decision needed.
- **A few hundred pages a month, one or two queries** → the smallest paid tier that covers N × 25; ignore the higher tiers entirely.
- **Volume that spikes with breaking news** → a tier with pay-as-you-go (Scaling and above) so a busy week doesn't stall the job.
- **Twenty-plus queries per run, results consumed by another service** → use the async endpoint with `queries` and a webhook, and size concurrency accordingly.
- **Strict newest-first ordering or event clustering in the spec** → check SerpApi's `so` and `story_token` against your requirements first; that's the gap in the ScraperAPI parameter set you can verify.

Two things to confirm before you pay. First, run your exact Google News URL through the Domain Cost Estimator so the 25-credit figure is confirmed on your account rather than assumed, and set `max_cost` on the request. Second, check that the `tld` list covers your target market together with the `hl` value you plan to send. A market that isn't in the twenty documented editions means falling back to `country_code` plus `UULE`, and that's better discovered during the free 40 requests than after a month of collection.

👉 [Compare ScraperAPI plans and pick your credit allowance](https://scraperapi.com/pricing?fp_ref=coupons)
