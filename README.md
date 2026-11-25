# SEO rank tracking API: managed feed or your own scraper? Costs, plan limits, and the depth math that decides it (with a full ScraperAPI plan table)

A rank tracking API returns keyword positions, position changes, the landing page URL that actually ranked, SERP features, competitor positions and history as JSON you can push into your own dashboard. You can buy that as a finished feed from a rank tracker, or assemble it yourself from raw Google SERP responses. ScraperAPI sits in the second camp: its Google SERP endpoint costs 25 credits per request, which is roughly 40,000 SERP pulls on the $149/month Startup plan and 200,000 on the $475/month Scaling plan.

Below is the full plan table, the arithmetic that turns credits into "keywords per day", and the documented limits that decide whether building beats buying for your team.

> **Short version:** buy a managed rank tracker if you want keyword projects, scheduled checks and history without writing storage code. Build on ScraperAPI if you already run a database and a scheduler, and you want control over what gets parsed (AI answers, shopping blocks, local packs) at a per-request price of about $0.0037 on Startup.
> 👉 [Compare every ScraperAPI plan and its current credit allowance](https://www.scraperapi.com/pricing?fp_ref=coupons)

## What a rank tracking API returns, and what a position number hides

A rank tracking API lets you create, manage and retrieve keyword ranking data programmatically, so rankings land in your own dashboards, tools, databases and reports instead of a vendor's UI. The field set is fairly standard across vendors: project and keyword management, tracking by location and device, positions and position change, the ranking landing page URL, SERP feature presence, competitor positions, tags and annotations, and historical data.

Position alone understates visibility. A keyword can sit at #3 and still lose the click to ads, a featured snippet, a local pack, shopping results, a video block or a People Also Ask box. That is why the useful APIs return the SERP composition alongside the rank, not just an integer.

Two things to check before you compare prices:

- **Granularity of location.** Country-only tracking is fine for broad reporting; city or postal-code targeting matters if you run local campaigns.
- **Freshness model.** Some APIs run a live SERP check on every request. Others return what their crawler collected on the schedule you configured, with a separate endpoint to force an out-of-schedule recheck. Both are legitimate; they produce different bills and different latency.

## ScraperAPI plans and the real cost of one SERP

ScraperAPI bills in credits. A standard page costs 1 credit, but a Google or Bing SERP request (including subdomains) costs 25 credits. That single number drives every calculation below.

| Plan | Credits / month | Concurrent threads | Geotargeting | Price | Purchase |
| --- | --- | --- | --- | --- | --- |
| Hobby | 100,000 | 20 | US & EU | On pricing page | [Select Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | US & EU | $149/mo, or $134.10/mo billed annually ($1,609.20/yr) | [Select Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Country-level | On pricing page | [Select Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | Country-level | $475/mo, or $427.50/mo billed annually ($5,130/yr) | [Select Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Country-level | On pricing page | [Select Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Country-level | On pricing page | [Select Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Country-level | On pricing page | [Select Enterprise](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Analytics history is capped at the last 30 days on Hobby and Startup and unlimited from Business upward. Geotargeting is the sharper dividing line: Hobby and Startup cover the US and EU only, while Business and above unlock country-level targeting for other markets.

The SERP endpoint itself takes `query`, `country_code`, `tld`, `hl` and `gl` for language and location, `start` for the result offset, `output_format` for JSON or CSV, and `tbs` for time-restricted results such as the past day. There is an async version that accepts an array of queries and posts results to a webhook when the batch finishes.

## Turning credits into keywords per day

One request returns one page of results. The docs' paging example uses `num=10`, and `start=10` moves you to page two, so a top-100 pull is ten requests per keyword.

- **Top-10 depth, checked daily:** 25 credits per day, about 750 credits per keyword per month. Startup's 1,000,000 credits cover roughly 1,300 keywords.
- **Top-100 depth, checked daily:** 250 credits per day, about 7,500 credits per keyword per month. The same Startup allowance covers about 130 keywords; Scaling's 5,000,000 credits cover about 660.
- **Cost per request:** $149 ÷ 40,000 requests works out to about $0.0037 on Startup, and $475 ÷ 200,000 to about $0.0024 on Scaling.

That spread is the actual decision. A project with 300 keywords at top-100 depth checked daily needs about 2.25 million credits a month, which puts you past Startup. A project with 300 keywords at top-10 depth needs about 225,000 credits, which fits Startup with room to spare. Depth multiplies cost tenfold, so decide early whether you need the full top 100 or only the first page.

👉 [Check the Startup plan's current price and credit allowance](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Buy vs build: the documented limits that shape the pipeline

Vendors publish these in their API docs, and each one changes what you have to build yourself.

| Documented limit | What it means for your system |
| --- | --- |
| A "top 100" request can return fewer than 100 rows, because a search engine may serve fewer than 100 organic results for a query | Store rank as nullable and never assume a fixed row count when joining datasets |
| Date-range queries capped at a 10-day span, and the date-range parameter cannot be combined with the "number of days" parameter (Similarweb) | A quarterly backfill has to live in your own database; you cannot re-pull three months in one call |
| Batch mode in blocks of 200 keywords, and batch requests cannot also carry a date or single-keyword parameter (Similarweb) | Plan your scheduler around batch boundaries |
| 5 requests per second per API key; overruns return "Too Many Requests", and repeated overruns trigger a temporary block (SE Ranking) | Batch jobs need explicit delay and backoff, not just a retry loop |
| 100 keywords per request (SEOptimer) | Chunk keyword lists at submission time |
| Country-level tracking only, across 20 supported countries (SEOptimer) | Postal-code or city targeting rules that vendor out for local SEO work |

ScraperAPI's own throttle is different in kind: exceeding your concurrency limit returns a 429, the request is turned away rather than queued, and you are not charged for it. Credits pause until renewal unless you enable auto-upgrade (available on Hobby through Professional) or pay-as-you-go overage (Scaling, Professional, Advanced, Enterprise, up to a monthly cap you set).

There is also the part nobody documents because it is not a bug: ScraperAPI returns a SERP, not a ranking. Your code has to parse the result list, find your domain, record the position, and diff it against yesterday to produce movement. That is a few hundred lines of work, and it is the reason managed APIs exist.

## The managed alternatives, with verified pricing

| Product | Model | Verified price | Coverage |
| --- | --- | --- | --- |
| SEOptimer | Live SERP check on every request | $50/mo for 500 monthly keyword ranking positions, $100/mo for 1,500, $200/mo for 5,000; annual equivalents $37.50/mo billed $450/yr, $75/mo billed $900/yr, $150/mo billed $1,800/yr | 20 countries, country level only, desktop and mobile; up to 100 keywords per request; API has no trial and bills immediately |
| SE Ranking | Usage-based credits | From $50.00 per 250K credits, pay-as-you-go, 100K credit trial | Google, Bing, Yahoo and YouTube; country or city; scheduled checks plus a recheck endpoint; top-100 per keyword; 35+ SERP features; 5 req/s per key |
| Mangools (SERPWatcher) | Project-based API | Not published in the documentation reviewed; key issued instantly from a free account, no card required | Country, state and city targeting; desktop and mobile; tags, annotations, keyword metrics |
| Similarweb (Rank Ranger) | Campaign-based `rank_top100` endpoint | Not published; key issued through an account manager | Google desktop and mobile; city, state and postal-code geo; SERP features returned when `pi=true` |

Two pricing notes worth keeping straight. SEOptimer's annual figures are a discounted monthly rate plus a total annual charge, so $37.50/mo and $450/yr describe the same plan. SE Ranking's $50 is the entry price of a credit pack, not a flat monthly subscription with a fixed keyword count.

## Which setup fits your case

- **You want positions in a client dashboard next week, with no ingestion code:** a managed tracker is cheaper than your engineering time. SE Ranking covers the widest engine list, including YouTube.
- **You need "right now" positions and nothing fancier than country-level:** SEOptimer's per-request live check and position-count pricing are easy to forecast.
- **You need full top-100 results plus SERP features and short lookback windows in one call:** Similarweb's endpoint is built for that shape of query.
- **You need postal-code targeting:** Similarweb documents it explicitly with examples.
- **You need raw SERPs at volume, your own scoring, or parsing the endpoint does not cover:** build on ScraperAPI. It is a fetching layer with a structured Google endpoint, not a rank tracker, and you own the projects, scheduling and history.

## FAQ

**Can I track just one country?**
Yes, with limits. ScraperAPI's SERP endpoint accepts `tld` and `country_code`, and the docs list domains including google.co.uk, google.de, google.co.jp and google.com.br. Plan geotargeting still applies: Hobby and Startup are US and EU only, so Japan or Brazil tracking needs Business or above.

**How often is the data updated?**
Whatever Google serves at the moment you call. There is no crawl schedule to inherit; your cron defines freshness. If you want vendor-side daily checks with historical rows stored for you, that is the managed model, not this one.

**Why does my top-100 pull return only 87 rows?**
Because the engine returned fewer than 100 organic results for that query. Similarweb documents this explicitly, and it applies to any API reading live SERPs. Handle short result sets in your storage schema instead of treating them as errors.

**What happens when I hit my limits?**
On ScraperAPI you get a 429 and the request is rejected, not queued, with no extra charge. Auto-upgrade moves you to the next plan when credits run out on Hobby through Professional; pay-as-you-go keeps requests flowing at a fixed per-credit rate on Scaling and above, capped at a monthly amount you choose.

**Is 25 credits per SERP expensive?**
At Startup's effective rate that is roughly $0.0037 per Google request. Ten thousand keyword checks a day at top-10 depth costs about $37 in credits per day. The comparison to make is against managed APIs charging per stored position, where the vendor absorbs the request cost and the retries.

## Before you pick a plan

Work out two numbers first: monthly SERP requests (keywords × checks per day × pages per keyword × 30) and the countries you actually need. Those two answers rule out most of the table above on their own.

Then confirm one operational detail: whether your expected volume sits comfortably inside a plan's credit allowance or on its boundary, because a project that fits Startup at top-10 depth will not fit it at top-100 depth. If you are within a few percent of the cap, the overage mechanics matter more than the headline price.

👉 [Open the ScraperAPI pricing page and match a plan to your request volume](https://www.scraperapi.com/pricing?fp_ref=coupons)
