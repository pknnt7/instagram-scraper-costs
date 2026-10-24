# instagram post scraper: what each request really costs, which plan fits your volume, and a working setup for public profiles

Most people searching for an instagram post scraper want the same three things: pull captions, like counts, and comment counts from public posts at a workable speed, without maintaining a proxy pool, and without getting blocked on request #40. ScraperAPI handles this with a single REST endpoint that routes your requests through rotating proxies and its own anti-bot logic. The part most articles skip is the billing: ScraperAPI charges credits per request, the price of a request depends on the domain and parameters, and Instagram is not on its published fixed-cost list. Below is the full plan table, the real cost math, and a tested workflow for public profiles.

> **Quick pick:** if you're testing the waters, start with the free trial before paying anything. Most solo developers fit comfortably in Hobby ($49/mo, 100,000 credits). If your monthly volume is unpredictable, Scaling ($475/mo) is the first tier with pay-as-you-go overage. 👉 [Try ScraperAPI free for 7 days — 5,000 credits, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

## Your three realistic options for scraping Instagram posts

There are basically three routes, and they suit very different situations.

Open-source scrapers come first. GitHub hosts several ready-made toolkits that pull post-level data — captions, likes, comments, media URLs — and some of them work without login or API keys. They cost nothing upfront. The tradeoff is that you own everything that goes wrong: Instagram changes its internal API regularly, and when a script breaks at 2 a.m., you're the on-call engineer.

Dedicated Instagram scraper APIs are the second route. Several vendors sell Instagram-specific endpoints that return a post as parsed JSON in one call. These are convenient, but you're locked into one platform's scope — if you later need Amazon pricing or Google results, that's a second vendor and a second bill.

General-purpose scraping APIs are the third route, and this is where ScraperAPI sits. One API key, one billing system, and the same credit pool works for Instagram posts, e-commerce pages, SERPs, or anything else with a URL. You write the parsing logic for Instagram yourself (more on that below), but you never touch proxies, CAPTCHAs, or browser fingerprinting.

If you'll only ever scrape Instagram and want zero parsing work, a dedicated Instagram API is a reasonable choice. If Instagram is one of several data sources in your pipeline, one general API with shared credits usually wins on cost and maintenance.

## ScraperAPI plans and pricing

ScraperAPI currently publishes seven self-serve tiers plus a free plan. Every paid tier includes the same feature set: JavaScript rendering, premium residential and mobile IPs, CAPTCHA and anti-bot handling, JSON auto-parsing, unlimited bandwidth, and a 99.9% uptime guarantee. The tiers differ in credits, concurrency, geotargeting, and overage handling.

| Plan | Monthly price | Annual (per month) | API credits | Concurrent threads | Buy |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | [Get Free](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [Get Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [Get Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [Get Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [Get Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [Get Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [Get Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Contact Sales](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

Annual billing takes 10% off every paid tier. Two entry ramps don't appear as cards in the plan grid: a 7-day trial with 5,000 credits (no credit card needed) and a permanent free plan with 1,000 credits capped at 5 concurrent connections. Professional currently carries a limited-time bonus of 250,000 credits, and Advanced 500,000 — worth checking before you commit at those tiers. 👉 [See current plan prices and bonus credit offers](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## The credit math most reviews skip

Here's the part that decides whether your budget actually holds: a credit is not a request.

A standard HTML request costs 1 credit. But Instagram's anti-bot environment means your real-world requests will rarely be "standard." Each add-on stacks on top:

- `render=true` (JavaScript rendering): +10 credits
- `premium=true` (residential proxies): +10 credits
- `premium=true` + `render=true` combined: 25 credits — not 20
- `ultra_premium=true`: 30 credits; with rendering, 75 credits
- Screenshot capture: +10 credits
- Bot-protection bypasses (Cloudflare, Datadome, PerimeterX/Human, Akamai): +10 credits each, auto-detected

That non-linear stacking is documented, but you have to dig into the docs to find it. Run the numbers on Hobby and it matters: 100,000 credits sounds like 100,000 pages, but if your workflow needs residential proxies plus rendering, you're looking at roughly 4,000 requests. On a simpler setup — plain requests, no rendering — the same credits stretch to the full 100,000.

Three policies work in your favor:

1. **Failed requests are free.** You're only charged for successful responses (HTTP 200 and 404) and for requests you cancel within the 70-second processing window. Hard failures cost nothing.
2. **You can audit costs before running.** The dashboard's Domain Multiplier looks up the credit cost of any URL, and there's a programmatic endpoint (`api.scraperapi.com/account/urlcost`) that accepts all the same parameters as a real request.
3. **Every response reports its own cost.** The `sa-credit-cost` response header tells you exactly what a request consumed, so you can log and monitor burn rate in your own code.

You can also set a `max_cost` per request as a hard ceiling, which is a decent safety net when experimenting with expensive parameter combos on an unfamiliar site.

## What does Instagram actually cost per request?

This is the question the pricing page won't answer directly, and it's worth being precise about it.

ScraperAPI publishes fixed credit costs for specific domain categories: Amazon, Walmart, and eBay cost 5 credits per request; Google and Bing (including all subdomains) cost 25; LinkedIn costs 30. **Instagram does not appear in that published list.** Its cost depends on which route your requests take and which parameters you enable, so the honest answer is: check it yourself before scaling up.

The check takes about a minute. Either open the Domain Multiplier in your dashboard and paste an Instagram post URL, or call the cost endpoint with your intended parameters:


https://api.scraperapi.com/account/urlcost?api_key=API_KEY&url=https://www.instagram.com/p/EXAMPLE/&render=true


It returns the exact credit cost for that URL with those settings. The API Playground lets you do the same thing interactively and see the actual response. ScraperAPI's own docs recommend this step because domains switch protection measures over time — a URL that costs 1 credit today might route through a bypass tomorrow and cost 11.

One more practical tip: set `max_cost` on your first Instagram batches. If the cost comes back higher than expected, the request gets rejected instead of silently draining your credit pool.

## A working setup for public Instagram profiles

You don't need a special Instagram product on ScraperAPI — the generic API works, with a bit of setup. A published walkthrough demonstrates the full flow against a public Instagram profile that aggregates old-house listings in Poland, and the shape of it generalizes:

1. **Fetch the profile.** Call Instagram's web profile info endpoint (`i.instagram.com/api/v1/users/web_profile_info/?username=...`) through ScraperAPI, including the `x-ig-app-id` header with the public web app ID. This returns the user ID, profile data, the first page of posts, and a pagination cursor. Adding custom headers costs nothing extra.
2. **Paginate through posts.** Hit Instagram's GraphQL query endpoint with the user ID, a page size (24 is the safe default), and the cursor from the previous response. Each page comes back as JSON you parse yourself.
3. **Extract what you need.** Per post: the shortcode (which gives you the post URL), image URLs, caption, comment count, like count, and timestamp. From there it's ordinary data work — the tutorial's example parses price and floor area out of real-estate captions with regex and filters for houses under 200,000 PLN.

On performance: that walkthrough reports scraping a 300-post profile in roughly 25 seconds, about 2 seconds per page. Your mileage depends on concurrency — Hobby's 20 threads is the floor, and higher tiers raise it in steps up to 500.

Two cautions. First, Instagram's internal endpoints are undocumented and change without notice; if a script that worked last month suddenly returns errors, the endpoints moved, and you'll need to update the URLs or headers. Second, this workflow targets public profiles and posts. Private accounts are a different matter entirely, and no scraper in this category touches them legitimately.

## Hidden limits that change which plan you pick

A few restrictions live below the plan cards and genuinely affect the choice.

**Geotargeting.** Hobby and Startup can only target the US and EU. If your Instagram research needs to appear from Japan, Brazil, or Australia, Business ($299/mo) is the entry point for country-level targeting across 50+ countries. No parameter combination gets around this on the lower tiers.

**Credit rollover.** Unused credits expire at the end of each billing cycle. There's no banking of unused volume, so don't buy a year's worth of headroom in one month.

**What happens when credits run out.** On Hobby, Startup, and Business, scraping simply stops until renewal — unless you've opted into auto-upgrade, which bumps you to the next plan automatically (available on Hobby through Professional). Scaling and above get pay-as-you-go instead: you keep scraping past your allowance at a fixed per-credit rate up to a spending cap you set. For a monitoring job that can't have gaps, PAYG is the reason to pick Scaling over Business even if 3M credits theoretically covers your volume.

**Analytics history.** Hobby and Startup keep usage analytics for 30 days only; Business and up keep them indefinitely. Minor, but annoying if you do monthly cost reviews.

## FAQ

**Is there a free way to test Instagram scraping on ScraperAPI?**
Yes. The 7-day trial includes 5,000 credits with no credit card, and after it ends you keep a free account with 1,000 credits at up to 5 concurrent requests. That's enough to run the profile-and-pagination workflow on a few accounts and measure your real per-request cost before spending anything.

**How many Instagram posts can I scrape per month on Hobby?**
It depends entirely on your parameters. Plain requests at 1 credit each give you up to ~100,000 requests; add premium proxies and rendering and the same pool covers roughly 4,000. Since Instagram isn't on the fixed-cost domain list, verify your specific setup with the urlcost endpoint, then divide your credits by that number.

**Do failed requests cost credits?**
No. Only successful responses (200 and 404) and requests you cancel inside the 70-second window are billed. Everything else — timeouts, blocks, errors — is free.

**Do credits roll over to next month?**
No. Unused credits expire at the end of each billing cycle.

**Can I scrape Instagram from non-US/EU locations on the cheapest plans?**
No. Hobby and Startup are limited to US and EU geotargeting. Country-level targeting across 50+ countries starts at Business.

## Picking a plan by scenario

If you're validating an idea or scraping a handful of profiles weekly, the free trial plus the permanent free plan will carry you for a while — spend nothing until you've measured your actual credit cost per post.

If you've measured and you're burning 20,000–80,000 credits a month, Hobby at $49 (or $44.10 on annual billing) is the natural fit. Startup makes sense only if you genuinely need 1M credits or the extra concurrency for batch jobs; the jump from 100K to 1M credits for 3× the price is steep if your volume sits in between.

Two situations justify skipping straight past Startup and Business: you need country-level targeting outside the US and EU (Business minimum), or your monthly volume swings unpredictably and gaps are unacceptable (Scaling, with pay-as-you-go overage at a fixed rate). Professional and Advanced are for teams running multi-million-credit pipelines where priority support and request prioritization pay for themselves.

Before you commit to any paid tier, run your target profile through the cost endpoint once and multiply. Then 👉 [start your 7-day trial with 5,000 credits and test your real workflow first](https://www.scraperapi.com/?fp_ref=coupons) — the numbers you get from your own URLs beat any estimate, including the ones in this article.
