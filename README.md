# residential proxies: how ScraperAPI's credit system works, what each page really costs, and when a scraping API beats buying proxy bandwidth

Most people searching for residential proxies don't actually want a pile of IP addresses — they want their scraper to stop getting blocked. ScraperAPI approaches that problem from a different angle than the usual proxy vendors: instead of selling you raw proxy ports billed by the gigabyte, it wraps a pool it advertises as 40 million+ IPs (residential, datacenter, and mobile, across 50+ countries) behind a single API call. You send a URL plus your API key, and proxy rotation, retries, CAPTCHA handling, and optional JavaScript rendering happen automatically. This guide breaks down its current plans, the credit math that determines your real cost per page, and the limits that only show up in the documentation.

> **Quick take:** for most solo developers and small teams, the **Hobby plan ($49/month, or $44.10/month billed annually, 100,000 credits, 20 concurrent threads)** covers typical price-monitoring and SEO workloads. Every paid plan starts with a 7-day free trial (5,000 credits, no credit card), so you can measure your actual credit burn before paying anything. 👉 [check current plan prices and start the free trial](https://www.scraperapi.com/pricing?fp_ref=coupons)

## What you're actually buying: an API with a residential pool, not a proxy list

Traditional residential proxy providers sell you access to ports and bandwidth. You write the rotation logic, handle bans, and manage sessions yourself. ScraperAPI abstracts that layer away. The residential pool is one resource inside it, reached through request parameters:

- `premium=true` routes your request through the harder-to-block residential and mobile pool (+10 credits per request)
- `ultra_premium=true` goes through the highest-grade pool (+30 credits, paid plans only)
- `country_code` handles geotargeting at no extra credit cost
- `session_number` keeps the same IP across a sequence of requests
- `render=true` spins up a headless browser for JavaScript-heavy pages (+10 credits)

One practical consequence: if you want a batch of fixed IPs you control long-term — for account management or ad verification — this product doesn't do that. You can't pin a specific IP or ASN. What you get instead is less infrastructure to maintain, plus structured data endpoints that return parsed JSON for Amazon product and search pages, Google Search and Shopping, and Walmart, so a site redesign doesn't break your HTML parser at 2 a.m.

## Current plans and pricing: the full lineup

Here is everything currently shown on the official pricing page. Monthly price first, annual-billed equivalent second (annual billing takes a flat 10% off every plan):

| Plan | Monthly | Annual (per month) | Credits | Threads | Get started |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49 | $44.10 | 100,000 | 20 | [ buy Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [ buy Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [ buy Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [ buy Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [ buy Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [ buy Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [ request a quote](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Scaling is the plan the page itself flags as most popular. There's also a free tier: sign up and get 1,000 free credits with a maximum of 5 concurrent connections, no card required.

Features are identical across all paid plans — JS rendering, premium residential and mobile IPs, advanced bypassing, structured data APIs, the DataPipeline, full crawler access, and pay-as-you-go support. What changes is volume, concurrency, geotargeting precision, and analytics retention. Two details worth noting before you pick a tier: Hobby and Startup only support geotargeting for the US and EU, and their analytics dashboards only keep the last 30 days of data. Country-level targeting starts at Business.

## The real cost per page: credit multipliers, explained with actual numbers

This is the part most comparison posts skip, and it's where budgets blow up.

ScraperAPI charges in credits, and a "credit" is one standard page request. But the docs price specific domains differently: **Amazon, Walmart, and eBay cost 5 credits per request; Google and Bing (all subdomains included) cost 25; LinkedIn costs 30.** On top of that, parameters stack: JS rendering adds 10, premium proxies add 10, and any request that successfully bypasses Cloudflare, Datadome, PerimeterX, or Akamai adds another 10.

Run the math on a realistic job. A Startup plan gives you 1,000,000 credits, which works out to roughly $0.000149 per credit on paper. Scrape 1,000,000 plain static pages and that's what you pay. But an Amazon search results page with JS rendering and premium routing can consume 15–30 credits per page — meaning your million credits might actually cover somewhere between 40,000 and 100,000 pages. One independent review that did this exact calculation put the effective per-page cost at 20–30x the headline number, and called the credit multiplier the single most common user complaint. The rules are documented; they're just not on the first screen of the pricing page.

Three official tools keep this under control:

1. **Domain Cost Estimator** — in your dashboard, look up the credit cost of any URL before you run a job. There's also an API endpoint (`/account/urlcost`) if you'd rather script it.
2. **API Playground** — test a target URL at scale-readiness and see actual credit usage per request before committing credits to a full run.
3. **`max_cost` parameter** — cap the credit spend of any single request, so a runaway loop can't drain your balance.

The billing fairness rules are also documented: you're only charged for successful requests (200 and 404 status codes), plus requests you cancel yourself before the 70-second timeout. Failed blocks don't bill. Every response includes an `sa-credit-cost` header showing exactly what that request consumed.

If your workload leans heavily on Google or LinkedIn, price out the true per-page cost first — those domains burn 25–30 credits even before rendering and premium routing stack on top. 👉 [open the pricing page and size your plan against your real per-page cost](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Three limits that only appear in the docs

None of these show up on the pricing page's first screen, and all three affect day-to-day use.

**What happens when credits run out.** By default, scraping pauses until your credits renew. Two mechanisms prevent that, and neither covers every plan. Auto-upgrade (your account moves to the next tier automatically when credits deplete) is available on Hobby through Professional. Pay-as-you-go (keep scraping past your limit at a fixed per-credit rate, up to a monthly cap you set) only exists on Scaling, Professional, Advanced, and Enterprise. Read that again if you're eyeing Hobby: **the entry plan has no pay-as-you-go option.** Hit 100,000 credits and your jobs stop until next cycle. Annual subscribers can also renew early for a fresh batch of credits, which heavy users can exploit as a top-up.

**Concurrency is a hard wall.** Exceed your plan's thread count and the API returns a 429 — requests are rejected, not queued, with no extra charge. Twenty concurrent threads on Hobby gets tight fast on production jobs pushing tens of thousands of pages a day. The $49 price tag is honest about credits but says nothing about how quickly 20 threads saturates.

**Sticky sessions have a shelf life.** `session_number` keeps the same IP across requests, but per third-party testing the session persists roughly an hour at most. For scraping workflows that's plenty; for ad verification or long-running account operations that need an identity that outlives an hour, a dedicated residential or mobile provider is the better tool category.

## ScraperAPI vs. bandwidth-billed residential proxy providers

Anyone searching for residential proxies will also run into providers that bill by the gigabyte. The pricing models don't convert directly — one charges per request with parameter multipliers, the other per GB of traffic — so the decision rests on which cost structure matches your workload:

| Provider | Billing model | Listed entry pricing | Notable terms |
| --- | --- | --- | --- |
| ScraperAPI | Credits per request | $49/mo for 100k credits (Hobby) | 7-day free trial; only 200/404 responses billed |
| IPRoyal | Pay-as-you-go bandwidth | around $1.75/GB | unused traffic reportedly doesn't expire |
| Bright Data | Pay-as-you-go bandwidth | listed around $8/GB in a recent comparison | 400M+ IPs advertised |
| Decodo | Monthly GB plans | roughly $2.20–3.75/GB entry tiers | 3-day free trial |

Mid-volume buyers generally see $3–4/GB at the 100 GB level across the market. Rough translation: if your targets are cheap to fetch (1 credit per page, ~500 KB per page), ScraperAPI's effective rate lands in the same ballpark as bandwidth billing. If every page costs 25+ credits, the math tilts hard toward bandwidth-billed providers for high-volume scraping of hard targets.

A simple decision rule, based on how the billing models behave:

- You already have a scraping framework and just need proxy capacity, with unpredictable volume → bandwidth billing with no-expiry traffic fits better.
- You're targeting standard e-commerce, SERP, and public-data pages and don't want to maintain rotation, retries, CAPTCHA solving, or parsers → the API's abstraction is worth the premium, and the default pool clears most non-hardened targets without touching the expensive tiers.
- You're hammering heavily protected platforms at serious volume → either way, unit costs multiply; test small before scaling on any provider.

## FAQ

**Is there a free trial?** Yes. Seven days with 5,000 API credits, no credit card required. There's also a standing free tier with 1,000 credits and 5 concurrent connections, and you can contact support if you need more credits for testing.

**How much does annual billing save?** A flat 10% off every plan. Hobby drops from $49 to $44.10 per month ($58.80 saved over a year); Startup drops from $149 to $134.10. You can cancel a subscription at any time from the dashboard or via support, and you won't be charged for cancelling.

**What happens if a request fails?** Nothing, cost-wise. Only successful requests (HTTP 200 and 404) are billed. If you cancel a request yourself before 70 seconds elapse, that one does bill — let long-running renders finish.

**Which languages can I use?** Anything that can issue HTTP requests. Official SDKs cover Python, Node.js, PHP, Java, and others, and there are integration guides for tools like Cypress when you need to scrape JavaScript-heavy pages or run end-to-end tests against dynamic content.

**Can I keep the same IP across requests?** Yes, via `session_number`, at no extra credit cost. Sessions last on the order of an hour, and you can't request a specific IP or ASN.

**Do Hobby and Startup support country targeting?** No — those two plans geotarget only at the US-and-EU level. Country-level targeting begins with Business at $299/month.

## Which setup fits your situation

- Monthly volume under ~200,000 pages against ordinary e-commerce and public-data sites → Hobby is the sensible entry point, and the 7-day trial will tell you if your targets clear on the default pool before you spend a dollar.
- Seven figures of monthly requests, or you need 50+ threads → Startup, stepping up to Business if you need country-level targeting or unrestricted analytics history.
- Amazon/Google/LinkedIn-heavy workloads → model your per-page credit cost with the Domain Cost Estimator first; if the multiplier math comes out ugly, bandwidth-billed providers (IPRoyal from roughly $1.75/GB, Decodo entry tiers around $2.20–3.75/GB) will likely cost less at scale.
- Long-lived sessions for accounts or ad verification → rotating residential pools of any kind are the wrong tool; look at dedicated static ISP or mobile options instead.

One last step before you pay for anything: run your three or four most important target URLs through the cost estimator and the free trial, multiply out the credits, and compare that against the plan's headline number. If they match, you're safe to commit. 👉 [start the free 5,000-credit trial and test your own targets](https://www.scraperapi.com/pricing?fp_ref=coupons)
