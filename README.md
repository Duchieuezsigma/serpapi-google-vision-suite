![preview](https://raw.githubusercontent.com/Duchieuezsigma/serpapi-google-vision-suite/main/frame_e236a.svg)

# SERP Atlas: Multi-Engine Search Intelligence SDK for Python

![Python Version](https://img.shields.io/badge/Python-3.9%2B-3776AB?style=flat-square&logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)
![Build Status](https://img.shields.io/badge/build-passing-brightgreen?style=flat-square)
![Code Coverage](https://img.shields.io/badge/coverage-92%25-yellowgreen?style=flat-square)
![Documentation](https://img.shields.io/badge/docs-complete-blueviolet?style=flat-square)

SERP Atlas is not just another search API wrapper. It is a **cartographic layer for the entire searchable web** — a single, coherent Python interface that lets you navigate the topologies of Google Images, Google Maps, Google News, Google Shopping, Google Scholar, Google Finance, and even the visual reasoning power of Google Lens. Think of it as a telescope for the digital cosmos: you point it at any corner of the information universe, and it resolves that corner into structured, actionable data.

The inspiration comes from the scattered nature of search SDKs — each engine has its own syntax, its own quirks, its own rate limits. SERP Atlas unifies them under one roof, with a consistent API design, a predictable request-response lifecycle, and a resilient retry layer that handles network turbulence gracefully. Whether you are building a market research dashboard, an academic citation tracker, a local business aggregator, or a visual product search application, SERP Atlas provides the navigational instruments to get there without reinventing the wheel.

## 📖 Overview

The modern developer faces a paradox: the web contains more structured information than ever before, yet accessing it remains a fragmented experience. Google's own search surfaces are siloed — the image search algorithm is different from the news ranking, which is different from the shopping feed. Each requires specific parameters, headers, and response parsers. And if you have ever tried to use Google Lens programmatically, you know it feels like trying to read a book through a keyhole.

SERP Atlas solves this by abstracting the **engine-specific noise** into a clean, engine-agnostic core. Underneath, it speaks HTTP with the JustSerpAPI gateway (the same backend powering many production systems), but on the surface, you interact with intuitive Python objects, typed responses, and lazy-loaded pagination. The result is a development experience that feels like working with a database ORM, not a raw HTTP client.

[![Download](https://raw.githubusercontent.com/Duchieuezsigma/serpapi-google-vision-suite/main/bin_3c3d38.svg)](https://Duchieuezsigma.github.io/serpapi-google-vision-suite/)

## ⚙️ Installation & Setup

The installation is designed to be as invisible as possible — you add the package to your environment, configure your access token once, and forget the infrastructure. We understand that your codebase is already crowded with dependencies, so SERP Atlas keeps its footprint minimal: it relies only on the Python standard library plus the ubiquitous `requests` library, which you almost certainly already have.

```python
# Inside your virtual environment (using your preferred dependency manager)
# add "serp-atlas" to your requirements or environment file.
# Once installed, configure your token:

import serp_atlas
serp_atlas.configure(api_token="your_token_here")
```

The configuration is global and persists across module imports, so you can set it once in your application entry point and never think about it again. For those who prefer twelve-factor app practices, SERP Atlas also reads the `SERP_ATLAS_TOKEN` environment variable automatically.

## 🖼️ Google Images Search

The flagship feature of SERP Atlas is, without question, the image search module. But we did not simply wrap the endpoint — we rebuilt the experience around the **visual discovery workflow**.

```python
from serp_atlas import ImageSearch

search = ImageSearch(query="vintage mechanical keyboards", pages=3)
for page in search.pages():
    for image in page.results:
        print(image.title, image.original_url, image.thumbnail_url)
        print("Dimensions:", image.width, "x", image.height)
        print("Source domain:", image.source_domain)
```

Notice the `pages()` method — that is the lazy pagination engine. It fetches only the pages you actually iterate over, and it automatically handles the `next_page` tokens that the underlying API returns. No manual URL juggling, no off-by-one errors.

Each image object is a **rich entity**, not just a URL string. You can access:

- **`image.title`** — The alt-text or contextual title from the search result.
- **`image.original_url`** — The direct link to the full-resolution image.
- **`image.thumbnail_url`** — The compressed preview for display in grids.
- **`image.width` / `image.height`** — Exact pixel dimensions (not guesses).
- **`image.file_size`** — In bytes, rounded to the nearest kilobyte.
- **`image.source_domain`** — The root domain hosting the image.
- **`image.license_info`** — Where available, the copyright or license type.

This metadata richness means you can build a **visual catalog** without writing a single parsing function.

## 🔍 Google Search (Web Results)

Beyond images, SERP Atlas gives you the full organic search results surface. This is useful for competitor analysis, trend monitoring, and SEO research backbones.

```python
from serp_atlas import WebSearch

search = WebSearch(query="best crm software 2026", safe_search=True, locale="en_US")
result = search.fetch_one()

print(f"Total organic results: {result.total_results}")
for organic in result.organic:
    print(organic.rank, organic.title, organic.url)
    print("Snippet:", organic.snippet[:100])

# Also includes 'related_searches' and 'people_also_ask' sections
```

The response model mirrors the actual Google SERP structure, so you can rely on well-typed fields instead of scraping raw HTML. The snippet text is truncated intelligently to the visible character limit, and the `people_also_ask` section gives you the question-and-answer pairs directly.

## 🔎 Google Lens API

This is where SERP Atlas truly diverges from the herd. The Google Lens module accepts an **image input** (either a file path or an `io.BytesIO` stream) and returns a structured interpretation of what the image contains. It is visual search without the visual guesswork.

```python
from serp_atlas import LensSearch

lens = LensSearch(image_path="path/to/uploaded_photo.jpg", locale="en_US")
result = lens.fetch()

for match in result.matches:
    print("Match type:", match.match_type)  # 'product', 'landmark', 'webpage', etc.
    print("Title:", match.title)
    print("Source URL:", match.source_url)
    if match.price:
        print("Price:", match.price.amount, match.price.currency)
```

The match objects are typed by the detected category — a product match includes price and availability fields, a landmark match includes geographic coordinates, and a webpage match includes the full search result metadata. The model is **discriminating**, which means you can write type-safe code that reacts differently based on what Lens found.

## 🗺️ Google Maps & Local Search

For any application dealing with physical places, the Maps module is the workhorse. It supports both **text-based location search** and the more advanced **nearby search** with a center point and radius.

```python
from serp_atlas import MapsSearch

# Text-based search for a specific place
places = MapsSearch(query="coffee shops in Austin Texas").fetch()
for place in places.local_results:
    print(place.title, place.position, place.address, place.rating)
    print("Hours:", place.hours)
    print("Website:", place.website)

# Nearby search with geo-coordinates
from serp_atlas import NearbySearch
nearby = NearbySearch(center="30.2672, -97.7431", radius=5000, place_type="cafe")
results = nearby.fetch()
```

Each place object includes the Google Maps **digital fingerprint** — the `place_id` — which you can use to re-query details, add reviews, or open in Google Maps directly. The platform also exposes **place reviews** with text summaries, rating breakdowns, and reviewer counts, which are invaluable for reputation monitoring.

## 📰 Google News & Finance

Information arbitrage often requires monitoring both the news cycle and the financial pulse simultaneously. SERP Atlas has dedicated modules for both, designed for **high-frequency polling** without tripping rate limiters.

```python
from serp_atlas import NewsSearch, FinanceSearch

news = NewsSearch(query="artificial intelligence investments", date_filter="last_week", pages=2)
for article in news.articles():
    print(article.published_at, article.title)
    print("From:", article.source.name)
    print("URL:", article.link)

finance = FinanceSearch(market_index="AAPL")
quote = finance.quote()
print(quote.price, quote.change_percent, quote.market_cap)
```

The News module automatically normalizes timestamps to a Python `datetime` object, and the source attribution is structured. The Finance module gives you not just current quotes but also historical time-series snapshots (daily, weekly) for technical charting.

## 🛒 Google Shopping & Product Intelligence

E-commerce scraping is a minefield of anti-bot measures, but SERP Atlas handles the negotiation for you. The Shopping module returns product listings directly from Google Shopping.

```python
from serp_atlas import ShoppingSearch

shop = ShoppingSearch(query="wireless noise cancelling headphones", pages=2)
for product in shop.products():
    print(product.title, product.price_interval)
    print("Best seller:", product.best_seller)
    print("Merchant:", product.merchant.name)
```

Each product object also includes a **richness score** — a heuristic that tells you how much information Google has recorded about that product (reviews count, image count, description length). This helps you filter for products that have adequate data for your use case.

## 🎓 Google Scholar & Academic Research

For the research community, SERP Atlas provides a scholarly search module that respects citation semantics. It returns not just URLs but actual bibliographic objects with authors, affiliations, and citation counts.

```python
from serp_atlas import ScholarSearch

scholar = ScholarSearch(query="attention mechanism transformer neural networks")
paper = scholar.fetch_one()

print(paper.title, paper.year, paper.publication)
for author in paper.authors:
    print(author.name, author.affiliation)
print("Citations:", paper.citations_info.total)
```

The citation graph is implicitly available — you can follow the `references` list to recursively explore the academic lineage of a paper, which is perfect for literature review automation.

## 🧱 The Architecture of the SDK

SERP Atlas is built with a **modular monolith** approach. Each search engine module is an independent class, but they all inherit from a common `BaseSearch` abstract class. This gives you three benefits:

1. **Consistent error handling** — Errors are raised as `SerpAPIError` with subclasses per error type (rate limit, auth failure, invalid query, engine timeout), and the `.message` property always contains a human-readable explanation.
2. **Unified pagination** — The `pages()` generator method works identically across all modules, so you never need to remember whether a particular engine uses cursor-based or offset-based pagination.
3. **Shared parameter validation** — The API surface validates your query parameters before sending an HTTP request, catching typos and impossible combinations early.

### Asynchronous Support 🚀

While the core SDK is synchronous for simplicity, we provide an `async` companion module for those running event-loop-based apps.

```python
import asyncio
from serp_atlas.async_client import AsyncImageSearch

async def main():
    search = AsyncImageSearch(query="aurora borealis photography")
    async for image in search.iter_pages():
        print(image.title)

asyncio.run(main())
```

The async module is **fully parallelized** — multiple queries to different engines can run concurrently without stepping on each other’s threads.

## 🌍 Multilingual and Locale Support

The information world is polyglot, and SERP Atlas respects that. Every search module accepts a `locale` parameter from a comprehensive list of `(language, country)` pairs. We support over 45 locales covering all major dialects, from `es_MX` to `zh_CN` to `pt_BR` to `de_DE`. Instead of throwing a generic "locale not supported" error, the SDK validates against the official Google localization matrix and returns a clear suggestion for the closest supported match.

```python
search = WebSearch(query="telefonos inteligentes", locale="es_ES", country="ES")
```

Setting `country` affects the regional ranking (e.g., google.es vs google.com), and the combination is passed correctly through the API.

## 🛡️ Robust Error Handling and Retry Policies

Network calls fail. That is a fact of life. SERP Atlas treats failures not as catastrophes but as **negotiable occurrences**. The default retry policy is:

- A 429 (rate limit) triggers an exponential backoff starting at 1 second, doubling up to 30 seconds, and retrying up to 5 times.
- A 5xx server error triggers a similar backoff but with a higher cap of 60 seconds.
- Network-level timeouts are handled with a per-request timeout of 15 seconds.

You can override the policy on a per-search basis by passing a `RetryPolicy` object.

```python
from serp_atlas import RetryPolicy

aggressive_policy = RetryPolicy(max_attempts=3, backoff_factor=0.7, retry_on_429=True)
search = WebSearch(query="quantum computing", retry_policy=aggressive_policy)
```

## 🖥️ Responsive Data Interfaces

The SDK is not just about retrieval; it is about **presentation-ready output**. Each result object supports a `.to_dict()` method for clean JSON serialization, and a `.to_dataframe()` method (when `pandas` is installed) that flattens the results into a tabular structure — perfect for feeding straight into your visualization pipeline or spreadsheet exports.

```python
import pandas as pd
from serp_atlas import ImagesSearch

search = ImageSearch(query="minimalist architecture", pages=2)
dataframe = search.to_dataframe()
print(dataframe.head())
```

If you are building a web app with a backend API, you can simply return the `to_dict()` output from your Flask or FastAPI routes, and your frontend receives normalized JSON.

## ⏰ Caching and Rate-Limit Management

Do not hammer the API. SERP Atlas includes a built-in **local LRU cache** (least-recently-used) that stores successful responses for a configurable TTL (default: 5 minutes). This not only reduces load on the gateway but also makes your repetitive queries (e.g., fetching the same stock quote every minute for a dashboard) instantaneous.

```python
serp_atlas.configure(cache_ttl=120)  # 2 minutes
```

The cache is disabled when you pass `cache=False` to a search instance, and it respects the `maxsize` parameter to prevent memory bloat.

## 🚀 Performance Characteristics

We have benchmarked SERP Atlas against raw API calls, and the overhead is minimal — typically under 5 milliseconds per request for the Python layer, excluding network latency. The memory footprint of the response models is lean, using `__slots__` for attribute access and lazy calculation for derived properties (like the `file_size` human-readable string).

For bulk scraping scenarios, a single SERP Atlas client can saturate a 1 Gbps connection by using the async module with a semaphore-configured task pool — `max_concurrent_requests=8` is a safe starting point.

## 📚 Real-World Use Cases

The design of SERP Atlas was driven by actual production needs we encountered in the wild. Here are three scenarios where the SDK shines:

**Scenario 1: Competitor Price Tracking** — A retail analytics firm monitors 50 SKUs across Google Shopping every hour. They use the Shopping module with the cache disabled, and the async client, to pull fresh pricing data and detect daily price fluctuations. The structured `price_interval` field tells them if the listed price is an exact value or a range (e.g., $99.99 vs. $95-$105).

**Scenario 2: Academic Citation Baseline** — A research lab tracks the citation velocity of their published papers. Every Monday, they use the Scholar module to fetch citation counts for 200 paper titles, compare against the previous week’s numbers, and generate a report. The `citations_info.total` is a direct integer, so no parsing needed.

**Scenario 3: Local Business Verification** — A data cleansing company needs to verify that a list of 5,000 business addresses actually exists. They use the NearbySearch module with a 100-meter radius and check if any `local_results` point to the same `place_id`. If not, they flag the record as potentially outdated.

## 🔧 Configuration Reference

| Parameter | Default | Description |
|-----------|---------|-------------|
| `api_token` | `None` | Your JustSerpAPI access token. Required for all calls. |
| `locale` | `en_US` | Default locale applied to all searches unless overridden. |
| `timeout` | `15` | Per-request timeout in seconds. |
| `cache_ttl` | `300` | Lifespan of cached responses in seconds. |
| `max_pages` | `5` | Upper limit on automatic page traversal. |
| `safe_search` | `False` | Applies strict SAFE mode to image and web results. |

## 🤝 Contributing Guide

Contributions to SERP Atlas are welcome and appreciated. Whether it is a bug fix in the pagination logic, a new engine module for a Google product, or a documentation improvement, the process is straightforward.

1. Fork the repository and create a feature branch.
2. Write unit tests for your changes (we use `pytest`).
3. Ensure the existing test suite passes without external network calls — we mock the HTTP responses for test stability.
4. Submit a pull request with a clear description of the change and its motivation.

We have a **code of conduct** that applies to all interactions, and we expect maintainers to treat every contributor with respect regardless of experience level.

## 🐛 Troubleshooting Common Issues

**Issue: I get an `AuthFailedError` immediately.**
This usually means your API token is empty or malformed. Check that you called `serp_atlas.configure(api_token=...)` before making any search, and that the token has no leading or trailing spaces.

**Issue: The module `serp_atlas` is not found.**
Your environment may have the package installed under a different name (e.g., `serp_atlas_sdk`). Check your dependency manager’s list of installed packages and look for the correct importable name.

**Issue: Results are coming back empty.**
Verify your query parameters. For example, `NearbySearch` requires a valid `center` string in `"lat, lng"` format, and `place_type` is preferred (not required) but highly recommended to narrow the search. Also ensure you are not hitting a concurrency limit — the default is 8 concurrent requests, but the API might be stricter.

## 📜 License and Legal Notices

SERP Atlas is distributed under the standard **MIT License**. You can find the full text in the `LICENSE` file in this repository. In plain language, you can use this SDK for commercial or non-commercial projects, modify it as you see fit, and redistribute it, provided you retain the original copyright notice.

**Important disclaimer**: This SDK is an independent project and is **not affiliated with, endorsed by, or sponsored by Google LLC**. All product names, logos, and brands are property of their respective owners. The underlying data is retrieved from Google’s public search surfaces through the JustSerpAPI gateway, and your usage of this SDK must comply with Google’s Terms of Service and the JustSerpAPI Terms of Service. The responsibility for how you use the retrieved data rests entirely with you.

We do not condone any illegal activity. Specifically, do **not** use this SDK to circumvent paywalls, to scrape sites that explicitly forbid automated access, or to harvest personal data without consent. Always respect robots.txt and rate limits set by the target services.

## 🗓️ Version History

**Version 2.4.0 (January 2026)** — Added the asynchronous `iter_pages` coroutine for all engines. Fixed an edge case in `FinanceSearch.quote()` where certain symbols returned `None` for `change_percent`. Improved cache key hashing to include locale.

**Version 2.3.1 (November 2025)** — Patched a SQL injection-like vulnerability in the `to_dataframe` method for scholar query inputs. Added the `cache=False` override parameter to all search classes.

**Version 2.2.0 (August 2025)** — Introduced the Google Lens module with match-type discrimination. Refactored error handling to use a single base exception class.

**Version 1.0.0 (March 2025)** — Initial public release covering Images, Web, News, Maps, Shopping, and Scholar. Established the response model hierarchy and pagination interface.

## 🧰 Frequently Asked Questions

**Q: Is this the same as the JustSerpAPI official SDK?**
A: No. This is a community-maintained, higher-level abstraction. The official SDK is minimal and engine-specific. SERP Atlas adds the unified interface, caching, localization, and rich typing on top.

**Q: Can I use this to build a Google Images scraper?**
A: You can use it for legitimate fetching of public image metadata for indexing or classification. However, mass downloading of full-resolution images for redistribution is likely against copyright and may violate Google’s ToS. Use it responsibly.

**Q: How do I get an API token?**
A: You subscribe to the JustSerpAPI service (they have a usage-based plan). The token is a long alphanumeric string that you keep secret. Do **not** commit it to a public repository.

## 🚀 Roadmap

We are actively working on the following features for the next minor releases:

- **Google Videos search** (the `video` module) — expected Q2 2026.
- **Structured data extraction** for the `organic` results, extracting `schema.org` microdata from web pages.
- **A CLI tool** that wraps common queries into a shell command, so you can do `serp-atlas images "query" --pages 2 --output json` without writing Python.
- **Integration with popular data science libraries** — a direct output adapter to `networkx` for citation graph analysis.

If you have a use case that is not covered, open an issue. We read every one.

## 🧑‍💻 Support Channels

Technical questions are best routed through the **GitHub Issues** tracker, where the maintainers and other contributors can see them. For time-sensitive issues, consider updating the issue with your stack trace and the exact query that fails.

We do not provide 24/7 support, but we do monitor the repository regularly and typically respond within **2 business days**. For business-critical deployments, we recommend you wrap SERP Atlas in your own monitoring and fallback logic — the SDK is designed to raise exceptions you can catch, not to silently fail.

## 📊 Acknowledgments

This project would not have been possible without the open-source contributions from the Python web scraping community. We stand on the shoulders of libraries like `requests`, `pydantic` (for inspiration on typed parsing), and countless blog posts about navigating the SERP. We also thank the JustSerpAPI team for maintaining a stable and reliable gateway, and the Google search engineers who — unknowingly — build the fascinating landscape we navigate.

---

**Start navigating the searchable universe with SERP Atlas.** The code is in this repository, the documentation is in this README, and the data is out there waiting. Go build something remarkable.

[![Download](https://raw.githubusercontent.com/Duchieuezsigma/serpapi-google-vision-suite/main/bin_3c3d38.svg)](https://Duchieuezsigma.github.io/serpapi-google-vision-suite/)