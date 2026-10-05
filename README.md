# browser use proxy: how to route Browser Use agents, Playwright and headless Chrome through residential IPs that hold up

Search "browser use proxy" and you land in two different conversations at once. One group means Browser Use, the AI agent framework, and wants to know where the proxy setting goes and why their runs keep dying on the fourth action. The other group just wants their browser to leave from a different country. Both end up asking the same question: which proxy, and how do you actually plug it in?

This is the setup guide for both, with the pricing you'll need to decide whether it's worth paying for.

## Why a browser agent is the loudest client you can run

A plain HTTP scraper sends a request, gets HTML, leaves. A browser pulls the whole page — scripts, fonts, images, tracking pixels — and then executes the JavaScript on top of it. From a target site's point of view that's a much richer fingerprint, and it repeats every session, usually from the same cloud IP address.

Datacenter IP ranges are the part that gets priced first. They announce themselves as servers rather than people, which is why a datacenter address under that load earns a Cloudflare or DataDome challenge almost immediately. Residential IPs route through ordinary home broadband connections, so they clear the reputation check that bounces a server range.

That's the whole reason the proxy conversation exists for browser automation. It isn't about anonymity, it's about not being scored as a bot on sight.

Worth stating plainly, because it's the mistake that wastes the most money: a browser-agent run produces two separate traffic streams. The reasoning calls (page state to a model, next action back) carry your API key and count against that key's quota no matter which IP they leave from. Only the browsing traffic — every navigation, asset and form post — goes through the proxy and benefits from it. Proxying the model call does nothing for your rate limits.

## Where the proxy setting actually lives

The answer depends on which browser you're driving, and getting it in the wrong layer is the second most common failure.

**Browser Use Cloud (managed):** every browser it provisions comes with a US residential proxy on by default. You pick a different country with `proxyCountryCode` — pass `"de"` for Germany, `null` to disable it for QA or internal sites. Managed residential runs at $5/GB across 195+ countries. Bring your own proxy instead and egress drops to $0.20/GB.

**Browser Use Cloud (custom proxy):** `customProxy` with `host`, `port`, `username`, `password` and an optional `ignoreCertErrors` flag, available on all accounts including free and pay-as-you-go. The field name depends on the endpoint — top-level `customProxy` for `POST /api/v4/browsers`, `browserSettings.customProxy` for `POST /api/v4/runs`. A custom proxy overrides `proxyCountryCode`.

**Local, open-source browser-use:** the proxy goes on the browser launch, not the agent. You build a `ProxySettings` object and attach it to a `BrowserProfile` or `BrowserSession`. Chromium accepts a `socks5://` server but will not do username/password authentication over SOCKS — for an authenticated exit you need an HTTP proxy URL, which is what most rotating gateways hand you anyway.

**Playwright, Puppeteer, Selenium:** same shape, different syntax. Playwright takes a `proxy` dict at launch; Puppeteer and Selenium connect over CDP to a browser you've created with the proxy already configured.

**Anti-detect browsers:** the proxy is set per profile, and the rule is one profile, one sticky IP, never shared. Two profiles on one IP links the accounts, which is the fastest way to get both flagged.

## Rotating or sticky: this is the decision that matters most

Rotating proxies hand you a fresh IP on every request. Sticky sessions hold one IP for a fixed window, then release it.

For browser work the split is straightforward:

- **Sticky** for anything with a login, a cart, a form flow, or a multi-step agent task where the site expects the same visitor to keep showing up. Expect to hold an IP for the full run.
- **Rotating** for sweeps where each page is independent — SERP checks, price monitoring across many product pages, ad verification.

DataImpulse runs rotating on port 823 for HTTP/HTTPS and 824 for SOCKS5. Sticky connections live in the 10000–20000 port range and hold for 1 to 120 minutes, defaulting to 30 if you don't specify. That's a reasonable ceiling for an agent run; if your task needs longer than two hours on one IP, plan around a re-login.

## What to check before you pay anyone

Five things separate a proxy that works with browser automation from one that technically connects:

1. **Is country targeting included or billed extra?** Cheaper providers quote a low per-GB rate and then charge for the geolocation you actually need. Country-level is usually free; state, city, ZIP and ASN filters are where the surcharge hides.
2. **How does authentication work?** Username/password or IP whitelisting. Whichever your stack supports without custom code.
3. **Does the traffic expire?** Monthly subscriptions that reset unused gigabytes are a bad fit for agent work, which comes in bursts.
4. **How many concurrent sessions?** Irrelevant at hobby scale, the limiting factor once you're running parallel agents.
5. **Does it fit your stack?** Integration blueprints for Playwright, Puppeteer, Selenium and Scrapy matter more than a feature list.

## DataImpulse plans and pricing

DataImpulse runs a pay-as-you-go, per-gigabyte model with no subscription and traffic that never expires — you top up a balance and spend it down. Four product lines, with volume discounts kicking in at 1 TB.

| Proxy type | What you get | Price | Billing / notes | Buy |
| --- | --- | --- | --- | --- |
| Residential (entry) | 5 GB, rotating + sticky sessions, HTTP(S)/SOCKS5, free country targeting, 90M+ IPs in 195 countries | $5 total — $1/GB | Pay-as-you-go, traffic never expires | [ Start with 5 GB residential](https://bit.ly/dataimPulse) |
| Residential (volume) | 1 TB | $800 — $0.80/GB | Volume tier; same network and targeting | [ See the 1 TB residential tier](https://bit.ly/dataimPulse) |
| Datacenter (entry) | 10 GB, 99.9% uptime, randomized datacenter subnets | $5 total — $0.50/GB | Cheapest option; best for undefended targets | [ Check datacenter pricing](https://bit.ly/dataimPulse) |
| Datacenter (volume) | 100 GB / 1 TB / 5 TB+ | $50 / $450 ($0.45/GB) / custom from $2,250 | State, city, ZIP and ASN targeting listed as included | [ See datacenter volume tiers](https://bit.ly/dataimPulse) |
| Mobile (entry) | 2.5 GB, 4G/5G/LTE IPs | $5 total — $2/GB | Most trusted IP class; CGNAT means carrier IPs are rarely blocked | [ Get mobile proxy access](https://bit.ly/dataimPulse) |
| Mobile (volume) | 25 GB / 1 TB / 5 TB+ | $50 / $1,600 ($1.60/GB) / custom from $8,000 | Volume pricing starts at the 1 TB tier | [ Check mobile volume pricing](https://bit.ly/dataimPulse) |
| Premium residential (entry) | 1 GB, low-latency pool, all targeting options included | $5 total — $5/GB | Dedicated account manager included | [ See premium residential](https://bit.ly/dataimPulse) |
| Premium residential (volume) | 10 GB / 5 TB+ | $50 / custom from $20,000 | Highest-trust residential traffic, no targeting surcharge | [ Get a premium residential quote](https://bit.ly/dataimPulse) |

Two billing details that decide your real cost. Country selection is free on residential; state, city, ZIP and ASN filters are billed at double the standard per-GB rate on standard residential plans, so a city-targeted crawl at $1/GB effectively costs $2/GB. And the mobile and premium residential volume discounts only start at the 1 TB tier — below that you're paying list price.

New users get a 7-day refund window, which makes the $5 entry package a genuine test rather than a commitment. Third-party assessments sit where you'd expect for a budget provider: a published 99.51% success rate, a 4.8/5 rating on G2, and a TechRadar review that notes consistently high scraping success on residential IPs.

## Setting up DataImpulse for browser automation

Once you have credentials, the endpoint is `gw.dataimpulse.com` and everything else is passed in the username string. The base form is your login plus parameters separated by semicolons:


LOGIN__cr.us
LOGIN__cr.us;city.newyork
LOGIN__cr.us;city.newyork;sessid.profile01


`__cr.us` sets the country, `city.newyork` narrows the location, and `sessid.` pins a sticky session — the same `sessid` always returns the same IP, which is what keeps one browser profile on one address.

For an open-source browser-use run, the proxy belongs on the browser object:

python
from browser_use import Agent, Browser, ProxySettings

browser = Browser(
    proxy=ProxySettings(
        server="http://gw.dataimpulse.com:823",
        username="LOGIN__cr.us;sessid.run01",
        password="YOUR_PASSWORD",
        bypass="localhost,127.0.0.1",
    ),
)

agent = Agent(task="Open the product page and report the current price", llm=llm, browser=browser)


The equivalent in Playwright is a launch argument:

python
proxy = {
    "server": "http://gw.dataimpulse.com:823",
    "username": "LOGIN__cr.us;sessid.run01",
    "password": "YOUR_PASSWORD",
}
browser = p.chromium.launch(proxy=proxy, headless=True)


Swap port 823 for 824 and `http://` for `socks5://` if you're on SOCKS5 — but remember the Chromium authentication caveat, so keep HTTP for credentialed exits.

**Verify before you run the task.** One curl tells you whether the exit IP and country are what you asked for:


curl -x "http://LOGIN__cr.us;sessid.run01:YOUR_PASSWORD@gw.dataimpulse.com:823" http://ip-api.com/json


If that returns the wrong country, your parameter syntax is off, and no amount of agent retry logic will fix it. Check the proxy, then run the agent.

## The cost math nobody does before their first agent run

Browser traffic is measured in gigabytes, and a browser session is expensive per page compared to a raw HTTP fetch, because every asset the page pulls counts. A single agent task that loads fifteen pages with images is not the same order of magnitude as fifteen `httpx` calls.

Three habits cut the bill:

- **Disable images** in the browser config when the task only needs text or numbers. Browser-use exposes that as a launch setting, and it's the single biggest lever on bandwidth per page.
- **Match the tier to the target.** If a site doesn't defend against server IPs, paying mobile rates is wasted budget. Datacenter at $0.50/GB handles a lot of internal and undefended work.
- **Use sticky sessions on multi-step tasks.** A new IP mid-flow triggers a re-challenge, and the re-challenge costs traffic you already paid for.

The $5 / 5 GB residential package is the sane starting point: it's a real test budget, it never expires if you don't use it this month, and it tells you your actual cost per successful request before you commit to a 1 TB tier. If you'd rather not guess, 👉 [grab a 5 GB test balance](https://bit.ly/dataimPulse) and point one agent run at a target you already know is defended.

## What DataImpulse is not

Being specific here saves you a support ticket later.

There's no managed scraping API. DataImpulse is a raw proxy layer — you write the request handling, parsing, retries and CAPTCHA logic yourself, though there are documented integration blueprints for Scrapy, Puppeteer, Selenium, Playwright, AdsPower, Multilogin and Zapier, plus code snippets for Python, Node.js, PHP, C#, Go, Ruby and cURL, and a Gateway API for programmatically generating proxy lists and tracking bandwidth.

It's also not for static ISP proxies, fully managed scraping pipelines, or bank and government sites — the provider says as much itself. Advanced targeting outside country level carries the 2× residential surcharge mentioned above. And if you need a guaranteed dedicated IP for a long-lived account, premium residential or sticky mobile is the tier to price, not standard residential.

## FAQ

**Does Browser Use include proxies?** Yes. Every cloud browser gets a US residential proxy by default, switchable by country code, at $5/GB. Bring your own and egress costs $0.20/GB.

**Can I use free proxies with browser-use?** Shared datacenter IPs from free pools get flagged and dropped quickly, and you have no idea what else is running through them. A residential gateway at $1/GB is cheaper than the debugging time.

**How many gigabytes does a browser agent need?** It depends on page weight and whether images are enabled. Measure one run on the 5 GB package rather than estimating — that's what the package is for.

**Do I need SOCKS5 for browser automation?** No. HTTP/HTTPS covers most browser traffic. SOCKS5 helps for raw TCP tunnelling, but Chromium won't authenticate over SOCKS, so use HTTP for any credentialed exit.

**Will a proxy fix my model rate limits?** No. Model quota follows your API key, not your IP.

**Do I need country targeting?** Usually yes, and it's free at country level. Sites score the exit IP and the locale together, so a German proxy paired with a US locale still looks wrong.

## The short version

If your search started with "browser use proxy", the answer has three parts. Put the proxy on the browser, not the model. Sticky for anything with a session, rotating for independent page sweeps. And pick a provider where country targeting is included and the traffic doesn't expire, because agent workloads arrive in bursts and subscriptions quietly punish that.

Standard residential at $1/GB covers most of it, datacenter at $0.50/GB covers the easy targets, and mobile at $2/GB is the tier you move to when a site keeps challenging you anyway. 👉 [Compare the current DataImpulse plans](https://bit.ly/dataimPulse) before you commit to a volume tier — the entry packages cost the same $5 whichever direction you lean, and the traffic stays yours either way.
