# dynamic web scraping python: 3 Working Methods With Code, the Real Credit Math, and When a Scraping API Beats a Headless Browser

Scraping dynamic websites with Python comes down to three options: find and call the site's JSON endpoints directly with `requests`, render the page with a headless browser like Playwright or Selenium, or hand the whole job to a scraping API such as ScraperAPI. Which one fits depends on a single question — is the data you need sitting behind a clean AJAX call, or does it only exist after JavaScript runs inside a real browser?

Most tutorials stop at the code. This one doesn't, because the expensive part of dynamic scraping is rarely the parsing. It's IP rotation, CAPTCHAs, and the credit math behind "unlimited" looking plans. We'll cover the code first, then the numbers.

> **Quick pick:** For a few JavaScript-heavy sites scraped a few times a week, start with the trial — ScraperAPI gives **5,000 API credits free for 7 days, no credit card required**. If it works on your targets, the Hobby plan is **$49/month** (or $44.10/month billed annually) with 100,000 credits. 👉 [Start the 7-day free trial and test your target sites](https://www.scraperapi.com/signup?fp_ref=coupons)

## Why `requests` + BeautifulSoup fails on dynamic sites

A plain HTTP client reads the first HTML response and stops. Anything a page adds afterward with JavaScript — infinite scroll, live search results, prices fetched from an internal API — never shows up in what BeautifulSoup sees. You send a GET request to a product page, print the HTML, and the price or listings you need are nowhere in it.

That's the actual definition of "dynamic" here. It's also why your first debugging step costs nothing: open Chrome DevTools, switch to the Network tab, reload the page, and watch what the browser does. The data has to come from somewhere, and very often it arrives as a request you can copy.

## Method 1: Hit the JSON endpoint directly (fastest, cheapest)

If the Network tab shows a clean XHR/Fetch request returning JSON, you can usually skip browser rendering entirely and call that endpoint with `requests`:

python
import requests

url = "https://example.webscraping.com/ajax/search.json?page=0&page_size=10&search_term=a"
r = requests.get(url, headers={"User-Agent": "Mozilla/5.0"})
data = r.json()


That example comes straight from a common tutorial pattern — one GET call plus `.json()` and the data is in a Python dict. No browser, no waiting, no rendering fees. This is the fastest and lowest-cost of the three methods whenever the data comes from a discoverable endpoint.

The catch is that not every site cooperates. Some internal APIs require session cookies or signed tokens; others fingerprint the request and drop anything that doesn't look like a real browser. If the Network tab shows a maze of tokens and rotating parameters, that's your signal to move on.

## Method 2: Render the page with Playwright or Selenium

When data only appears after scrolling, clicking, or full client-side rendering, you need a browser engine to run the JavaScript for you. Playwright is the sensible default for new Python projects: it launches a real browser, auto-waits for elements, supports Chromium, Firefox, and WebKit through one API, and is faster than Selenium in most benchmarks.

python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://quotes.toscrape.com/js/", wait_until="networkidle")
    page.wait_for_selector(".quote")
    for quote in page.query_selector_all(".quote"):
        print(quote.query_selector(".text").inner_text())
    browser.close()


Two details separate working scripts from flaky ones:

- **Wait for elements, not for seconds.** `wait_for_selector(".product-card")` returns as soon as the element exists. `time.sleep(5)` either wastes time or breaks on slow networks. If you don't know which element to wait for, `wait_until="networkidle"` is a reasonable fallback.
- **For infinite scroll**, loop `page.mouse.wheel(0, 15000)` with a short `wait_for_timeout`, comparing `document.body.scrollHeight` each round. When the height stops growing, you've reached the bottom.

Selenium does the same job and remains worth picking if you need a specific browser version or are joining an existing Selenium test suite — it has the largest community and widest browser and language support. Its minimal setup is `--headless=new` plus `--no-sandbox`, with `WebDriverWait(driver, 10)` for waiting.

One proxy detail that trips people up in Selenium: Chrome's `--proxy-server` flag reads host and port cleanly but doesn't reliably accept a username and password embedded in the same string. IP-whitelisted proxies work directly; rotating residential proxies that authenticate by credentials need an extra workaround. Playwright's proxy dictionary handles both authentication methods natively — one more reason it's the simpler default.

The real cost of this method isn't money, it's infrastructure: headless Chrome eats RAM, and neither Playwright nor Selenium does anything about IP blocks or CAPTCHAs. If you're scraping protected sites at any volume, that becomes a second project of its own.

## Method 3: ScraperAPI — JavaScript rendering and IP rotation with one parameter

A scraping API replaces your HTTP client entirely. You send your target URL to ScraperAPI's endpoint, and its infrastructure renders JavaScript, rotates residential and mobile IPs, and handles CAPTCHAs and anti-bot detection before returning the final HTML.

python
import requests

payload = {
    "api_key": "YOUR_API_KEY",
    "url": "https://www.amazon.com/dp/B07PGL2N7J",
    "render": "true",      # renders JavaScript before returning HTML
    "country_code": "us",
}
r = requests.get("https://api.scraperapi.com/", params=payload)
print(r.status_code)


You can also use the official Python SDK with `ScraperAPIClient(API_KEY)` and `client.get(url, render=True)` — either way, the response comes back as plain rendered HTML, so your existing BeautifulSoup code works unchanged. Pagination is just a loop over constructed URLs, checking `status_code == 200` each time.

Two honest trade-offs. First, speed: rendering happens per request with a headless Chrome, which takes roughly 5–10 seconds per page. Second, credits: a rendered request through premium proxies burns far more than the 1 credit a standard page costs. More on that below, because it changes which plan you should actually buy.

👉 [See current plan prices and credit allowances](https://www.scraperapi.com/pricing?fp_ref=coupons)

## All ScraperAPI plans and prices

Every paid plan includes the same core toolkit: JavaScript rendering, premium residential and mobile proxies, CAPTCHA and anti-bot bypass, automatic retries, custom headers and sessions, unlimited bandwidth, and a 99.9% uptime guarantee. Plans differ in credits, concurrency, and geotargeting reach. Annual billing takes 10% off all of them.

| Plan | Credits / month | Concurrent threads | Geotargeting | Price (mo / annual) | Subscribe |
| --- | --- | --- | --- | --- | --- |
| Free | 1,000 | 5 | — | $0 | [ Get 1,000 free credits](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | 100,000 | 20 | US & EU only | $49 / $44.10 | [ Start with Hobby](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | US & EU | $149 / $134.10 | [ Start with Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Country-level | $299 / $269.10 | [ Start with Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | 5,000,000 | 200 | Country-level | $475 / $427.50 | [ Start with Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Global + priority support | $975 / $877.50 | [ Start with Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Global + priority routing | $1,975 / $1,777.50 | [ Start with Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Global | Custom quote | [ Contact sales](https://www.scraperapi.com/pricing?fp_ref=coupons) |

The 7-day trial (5,000 credits, up to 5 concurrent connections, no payment method) is the practical way to find out whether ScraperAPI actually succeeds on your specific targets before spending anything.

## The real credit math most reviews skip

A plan advertising 100,000 credits does **not** mean 100,000 pages. ScraperAPI prices domains differently because some are genuinely harder to scrape:

- Standard page: **1 credit**
- Amazon: **5 credits**
- Google and Bing, including subdomains: **25 credits**
- LinkedIn: **30 credits**
- Sites behind Cloudflare, Datadome, or PerimeterX: **+10 credits per request** once bypassed
- Rendered requests through premium proxies: tens of credits each (a third-party 2026 comparison measured JS-rendered residential requests at 30+ credits, and noted JS rendering adds roughly $5 per 1,000 requests)

So the Hobby plan's 100,000 credits translates to about 20,000 Amazon pages, 4,000 Google SERPs, or under 1,000 rendered requests against a protected site. Same plan, three different projects. This is the single most misunderstood part of scraping-API pricing, and none of the method-focused tutorials mention it.

ScraperAPI does give you two ways to keep the math under control. The dashboard's **Domain Cost Estimator** tells you the exact credit cost for any URL before you build around it, and the `max_cost` parameter caps what a single request may consume — so one unlucky URL can't drain your monthly balance mid-job.

Usage-edge behavior is also worth knowing before you commit:

1. Exceed your concurrent thread limit and you get a 429 — the request is turned away, not queued, with no extra charge.
2. Run out of credits and scraping pauses until renewal by default. Hobby through Professional can opt into auto-upgrade to the next plan; Scaling and above support pay-as-you-go at a fixed per-credit rate, up to a monthly cap you set.

👉 [Check what your target URLs cost per request](https://www.scraperapi.com/pricing?fp_ref=coupons)

## How ScraperAPI stacks up against the alternatives

For dynamic scraping, the managed-API market splits cleanly by target difficulty. A May 2026 third-party benchmark (50 popular sites, 1,000 requests each) measured all three:

|  | ScraperAPI | Zyte API | Bright Data Web Unlocker |
| --- | --- | --- | --- |
| Entry price | $49/mo, 100k requests | ~$50/mo, usage-based | ~$1.05 per 1,000 requests |
| Simple site cost | ~$0.49/1k | ~$0.40–3.00/1k | $1.05–3.00/1k |
| Bypass success (Cloudflare / DataDome / PerimeterX) | 92% / 78% / 73% | 95% / 91% / 88% | 99% / 97% / 96% |
| JS render speed | 5–10s | 3–7s | 2–5s |
| Trial | 5,000 requests, minutes to onboard | $5 credit, hours | 7 days, KYC required, 24–48h |

Success rates by difficulty (ScraperAPI / Zyte / Bright Data): easy sites 99.2% / 99.5% / 99.6%; medium 96.4% / 98.1% / 99.4%; hard (advanced Cloudflare, DataDome) 78.3% / 91.7% / 97.8%; extreme (PerimeterX, Kasada, Akamai) 71.5% / 88.2% / 96.4%. Per successful request, ScraperAPI's cost inflated from $0.0005 on easy sites to $0.0210 on extreme ones — because failed requests still consume premium credits — while Bright Data stayed flat at $1.05 per 1,000 regardless of difficulty.

There are also self-serve competitors like ScrapingBee and ZenRows running similar credit models, and the free option from earlier in this article still wins for static pages and discoverable endpoints. One genuine differentiator elsewhere: Zyte's auto-extraction returns structured product JSON without you writing parsers — in the benchmark's 100-product test, price accuracy hit 97.8% and name accuracy 99.4%. If structured data rather than raw HTML is your end goal, that matters.

The benchmark's own verdict: ScraperAPI is the cheapest for simple needs, Zyte is the most engineering-friendly, Bright Data wins on the hardest sites at premium prices.

## FAQ

**Do I still need a headless browser if I use ScraperAPI?**
No, not for data collection. `render=true` handles JavaScript rendering server-side. Keep Playwright for cases where you must interact with a page — logins, clicking through flows — before extracting.

**What happens when my credits run out mid-month?**
By default, scraping pauses until the next billing cycle. You can opt into auto-upgrade (Hobby through Professional) to move up a plan automatically, or pay-as-you-go (Scaling and above) to keep scraping at a fixed per-credit rate with a monthly cap.

**Is there a free way to test it on my actual sites?**
Yes. The 7-day trial includes 5,000 credits and up to 5 concurrent connections with no credit card required. Run your real targets through it and check each one in the Domain Cost Estimator first.

**Can I scrape Google and Amazon with it?**
Yes, but budget accordingly: Google and Bing cost 25 credits per request, Amazon 5. The Hobby plan covers roughly 4,000 Google searches or 20,000 Amazon product pages per month — but far fewer if you also enable rendering.

**Which method should a beginner start with?**
Open the target site's Network tab first. A clean JSON endpoint means `requests` will do. Anything requiring rendering, try the free trial of a scraping API before investing in headless-browser infrastructure — there's less code to maintain when the site changes.

## Bottom line: match the method to your target sites

Three rules cover most cases:

- **Clean JSON endpoint or mostly static pages** → plain `requests` + BeautifulSoup. Free, fast, done.
- **JS-rendered but lightly protected sites, moderate volume** → ScraperAPI's Hobby or Startup plan. One `render=true` parameter replaces proxy pools and CAPTCHA-solving code, and the 5,000-credit trial tells you within a day whether it holds up on your targets.
- **Hard anti-bot targets or structured-extraction needs** → ScraperAPI's cost per successful request climbs steeply on those sites; Zyte or Bright Data fit that tier better, at higher prices.

Before paying for anything, run two checks: paste your actual URLs into the Domain Cost Estimator to get real credit costs, and use the 7-day trial to measure success rates on the sites you care about. Those two numbers decide your plan far better than any comparison table. 👉 [Create your free ScraperAPI account and start testing](https://www.scraperapi.com/signup?fp_ref=coupons)
