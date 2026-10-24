# python scraper library: Which One Fits Your Project, Where Each Breaks, and the Credit Costs Most Reviews Skip

Most people searching for a Python scraper library want the same answer: which one should I actually install? The short version is that three open-source options cover nearly everything — requests + BeautifulSoup for static HTML, Playwright for pages that need a real browser, and Scrapy when you're crawling thousands of URLs systematically. Everything else is a variation on those three, or a sign that your problem has outgrown libraries entirely and you should hand the hard parts to a managed scraping API [S3/S25].

This guide walks through what each library is good at, the specific situations where they fall apart, and what it looks like when you switch the infrastructure side of scraping to a service like ScraperAPI — including current plan prices, credit costs that most reviews gloss over, and the limits that only show up in the billing docs.

> **Quick picks**
> - Scraping static pages (news, Wikipedia, server-rendered product pages): requests + BeautifulSoup. Free, running in 20 minutes.
> - Large, systematic crawls: Scrapy. Free and open source.
> - JS-heavy sites you control the browser for: Playwright. Free, but you maintain the browser binaries.
> - Sites behind Cloudflare, Datadome, or PerimeterX, or any project where proxy rotation and CAPTCHA handling aren't things you want to build: a managed API like ScraperAPI. Plans start at **$49/month** ($44.10/month billed annually), you get a 7-day trial with 5,000 free credits and no credit card, and independent reviews report a 7-day money-back guarantee on paid plans — 👉 [try ScraperAPI's free trial](https://www.scraperapi.com/?fp_ref=coupons)

## The three libraries that matter, and what each one is actually for

The comparison below reflects how working scrapers get built, not a feature-list beauty contest. A detailed 2026 evaluation of six Python scraping libraries measured them on raw HTTP performance, parsing ergonomics, JavaScript rendering, async support, maintenance burden, and time from `pip install` to first working scraper [S3/S25].

**requests + BeautifulSoup** has been the default Python scraping combination since roughly 2012, and that's not nostalgia — it covers the vast majority of publicly accessible pages. requests handles sessions, headers, cookies, redirects, and retries (its own docs note that connection pooling is "100% automatic"), while BeautifulSoup parses HTML with CSS selectors and `.find()` / `.find_all()`. Two caveats worth knowing before you commit: it's single-threaded by default, so sequential requests get slow at scale, and it executes no JavaScript — anything rendered client-side comes back as empty content. BeautifulSoup's built-in parser is also slower than lxml; the official docs put it plainly: "If you can, I recommend you install and use lxml for speed" [S3/S25].

**Scrapy** is a full crawling framework, in production for over fifteen years with more than 500 contributors, and it's still actively shipping. Version 2.19.0 landed in September 2026 with a new RemoteControl extension for inspecting running crawls over HTTP, and an aiohttp-based download handler that became the default outside a reactor [S2]. The framework handles retries, rate limiting, user-agent rotation, and deduplication out of the box, and exports to JSON, CSV, or XML without extra work. The trade-off is a steeper learning curve, no built-in JavaScript rendering (you bolt on scrapy-playwright or scrapy-splash), and genuine overkill for a 50-URL one-off script [S3/S25].

**Playwright**, maintained by Microsoft, is the most reliable option for SPAs, infinite scroll, login flows, and cookie-consent walls — it drives Chromium, Firefox, and WebKit through one API, and its auto-waiting eliminates the race conditions that plagued Selenium-era code. The costs are operational: you download and keep browser binaries in sync with `playwright install`, each request is slower than a plain HTTP call, and memory adds up fast. The same evaluation notes that 100 concurrent Playwright contexts will strain most machines [S3/S25].

One naming trap while we're here: **Scrapely** is not Scrapy. It's a separate extraction library from the same maintainers that learns what to pull from example pages ("No xpaths, regular expressions, or hacky python code"), and it's closer in spirit to BeautifulSoup than to the Scrapy framework. The project's own docs warn that its `train()` implementation is basic and reference-only, while the extraction code itself is production-ready [S1]. Most library roundups never mention this distinction, and it matters if you're evaluating tools by name.

## Where libraries stop being the answer

Every library above shares the same blind spot: they solve *your* code, not the network between you and the target site. The moment a site sits behind Cloudflare, Datadome, or PerimeterX, or requires rotating residential IPs, or serves content only after JavaScript runs in a fingerprinted browser, the library stops being the project. You're suddenly maintaining proxy pools, headless browser fleets, and CAPTCHA-solving pipelines — and that maintenance is most of the real cost.

The numbers back this up. One 2026 analysis modeled total scraping cost using US developer salary medians multiplied by 1.3, and found that engineer time dwarfs everything else: at 100,000 requests per month, maintenance accounts for roughly 72% of total cost; even at a million requests monthly it's still about half [S3/S25]. Framework comparisons rarely mention this because it makes the library choice almost secondary.

That's the point where a managed API like ScraperAPI enters. You send a URL and your API key; it returns the page HTML, having handled proxy rotation, JavaScript rendering, retries, and CAPTCHA bypass on its side [S24][S16]. Your scraping code shrinks to an HTTP call, and the infrastructure problems become someone else's uptime guarantee.

## What a request actually costs: the credit math nobody explains

This is the part most reviews gloss over, and it changes which plan makes sense. ScraperAPI bills in credits, but a credit is not always a page.

A standard page costs 1 credit. Then it gets site-specific: Amazon pages cost 5 credits, Google and Bing (including their subdomains) cost 25, and LinkedIn costs 30. Sites behind bot protection like Cloudflare, Datadome, or PerimeterX add 10 credits per request when the service bypasses them [S9]. Stack those charges on a site that also needs JavaScript rendering and a single page can consume dozens of credits — one third-party analysis cited a rendered protected page running about 75 credits, which is why it estimated the 1,000-credit free plan covers roughly 13 such pages per month at that site [S13]. The exact number always depends on the target, not on a formula.

Before assuming any plan is too small or too big, check your actual targets. The dashboard includes a Domain Cost Estimator that tells you the exact credit cost for any URL, and you can cap any single scrape with a `max_cost` parameter so one bad request can't drain your balance [S9].

Two more operational details worth knowing up front:

- **Concurrency is hard-capped.** If you exceed your plan's concurrent thread limit, requests come back as HTTP 429 and are refused — they don't queue, and you're not charged for them [S9]. If your job needs 60 parallel requests, the Hobby plan's 20 concurrent threads will bottleneck you regardless of how many credits you have.
- **Requests are capped at 50MB** per response, and ScraperAPI's own docs recommend setting a 70-second timeout in your application to get the best success rates, especially on hard-to-scrape domains [S24].

## ScraperAPI plans and pricing

All plans include JavaScript rendering, premium residential and mobile IPs, CAPTCHA and anti-bot detection, automatic retries, unlimited bandwidth, structured data endpoints, DataPipeline access, and a 99.9% uptime guarantee — the differences are credits, concurrency, and geo-targeting [S9]. Annual billing takes 10% off every paid tier.

| Plan | Price (monthly) | Price (billed annually) | Credits | Concurrent threads | Get started |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | $0 | 1,000/mo | 5 | [Sign up free](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 | $44.10/mo | 100,000 | 20 | [View Hobby pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10/mo | 1,000,000 | 50 | [View Startup pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10/mo | 3,000,000 | 100 | [View Business pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50/mo | 5,000,000 | 200 | [View Scaling pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50/mo | 10,500,000 | 300 | [View Professional pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50/mo | 21,500,000 | 500 | [View Advanced pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Talk to sales](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

Geo-targeting is a real dividing line between tiers: Hobby and Startup are limited to US and EU locations, Business and above unlock country-level targeting, and the analytics dashboard only keeps 30 days of history on the lower tiers (unlimited on Business and up) [S9].

You can test all of this without a credit card: the free plan gives 1,000 credits per month with up to 5 concurrent connections, and the first 7 days after signup come with 5,000 credits at a larger scale [S9][S11][S13]. If you need extra testing credits beyond that, support can grant more on request [S9].

For context on where the break-even sits against per-GB competitors: one comparison found ScraperAPI cheaper below roughly 500K pages per month, while providers that bill by data volume, such as Bright Data, pull ahead past the million-page mark as their per-GB pricing compresses at scale [S19]. In the 500K–1M range, the same analysis says the choice comes down to unblocking power and features rather than price alone [S19].

## What happens when you run out of credits (and the settings worth changing)

The default behavior is simple: scraping pauses until your credits renew. But the pricing FAQ documents three escalation paths that almost no comparison article covers [S9]:

1. **Upgrading** — a manual move to a higher plan from the billing page, usually with a better per-credit rate. Available anytime, on any plan.
2. **Auto-upgrade** — opt-in, and only on Hobby through Professional. When credits run out, your account moves to the next plan automatically instead of stopping.
3. **Pay-as-you-go** — on Scaling, Professional, Advanced, and Enterprise only. You keep scraping past your allowance at a fixed per-credit rate, capped at a monthly spending limit you set. No plan change.

The distinction matters more than it looks. If you enable auto-upgrade on Hobby and one busy month burns through 100,000 credits, you've auto-billed yourself into Startup territory. If you'd rather have predictable spend, pay-as-you-go on a higher plan with a strict cap does that job instead. Annual subscribers can also renew early to get a fresh batch of credits [S9].

On the integration side, there are six ways to call the service: the standard API endpoint, an async endpoint, a proxy port (so you can point an existing requests-based script or Scrapy project at it by changing one proxy setting), structured data endpoints for Google and Amazon results, the DataPipeline service for bulk and scheduled jobs, and SDKs for select languages [S24]. The proxy-port route is the lowest-friction option for people who already have Python scraper library code working — the parsing logic stays untouched, only the network layer changes. There's also an MCP Server and a LangChain integration if you're wiring scraping into LLM agents, and a community n8n node for no-code workflows [S24].

## When you should stick with a library and skip the API entirely

Honesty section. ScraperAPI is the wrong purchase in several common cases, and pretending otherwise helps nobody.

If your targets are static HTML — documentation sites, Wikipedia, blog archives, server-rendered product pages — requests + BeautifulSoup will finish the job in an afternoon and cost nothing. There's no CAPTCHA wall, no IP rotation problem, nothing for a paid service to solve. Same for a one-time migration crawl of a few hundred pages: Scrapy's free tooling or even a plain script beats a subscription [S3/S25].

If you need JavaScript rendering but the site has no serious bot protection, Playwright on your own machine does the job, and the only cost is browser maintenance [S3/S25]. And if you're crawling at genuine scale with engineering time to spare, Scrapy plus a proxy provider you manage yourself can be cheaper at the high end — that's exactly the region where the per-GB competitors win on price [S19].

The case for ScraperAPI is specific: your targets fight back, you're spending real engineering hours on unblocking instead of on the data, and your volume sits in the range where $44–475 per month is cheaper than the salary-hours alternative [S3/S25]. If that sentence describes your project, the pricing below is the decision; if it doesn't, close the tab and `pip install` something free.

## FAQ

**Does the ScraperAPI free trial require a credit card?** No. The 7-day trial provides 5,000 credits with no card required, and the standing free plan adds 1,000 credits per month with up to 5 concurrent connections [S9][S10][S11].

**Why did one request cost me 25 or 30 credits?** Search engines and some platforms are priced specially: Google and Bing cost 25 credits per request, LinkedIn 30, Amazon 5, and a standard page 1. Protected sites add 10 credits when the bypass runs [S9].

**How does ScraperAPI compare to just using Scrapy?** They solve different layers. Scrapy is a free open-source framework for building crawlers — it handles request scheduling, dedup, and pipelines, but the proxies, rendering, and CAPTCHA work stay on you [S2][S25]. ScraperAPI is infrastructure-as-a-service for exactly those parts. They're not mutually exclusive: you can point Scrapy's download layer at ScraperAPI's proxy port and keep your spiders as-is [S24].

## Before you pick a plan

Two checks settle most decisions. First, run your target domains through the dashboard's cost estimator — a plan sized on standard pages can be 25x too small if most of your volume is Google or LinkedIn, and Hobby's 100,000 credits is 100,000 standard pages but only 20,000 Amazon pages [S9]. Second, check your concurrency needs against the plan caps, since 429s don't queue.

Beyond that, the table above is the whole decision: static-HTML and small free-tier jobs need nothing paid; Hobby at **$49/month** ($44.10 annually) covers modest protected targets; country-level targeting starts at Business; and past a million pages a month it's worth comparing the per-GB competitors too, since their pricing bends in your favor at that scale [S19]. When your target domains check out, 👉 [start your 7-day ScraperAPI trial and test it on your real targets](https://www.scraperapi.com/?fp_ref=coupons) before committing to a tier.
