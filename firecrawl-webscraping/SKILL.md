---
name: firecrawl-webscraping
description: Firecrawl web scraping, crawling, searching, and AI-driven data extraction — CLI, agents, browser automation, sitemaps. Auto-activates for web scraping or crawling tasks.
origin: VoltAgent/awesome-agent-skills (firecrawl)
tools: Read, Write, Edit, Bash
---

# Firecrawl Web Scraping Skills

Official Firecrawl skills for scraping, crawling, and extracting structured data from the web.

## Setup

```bash
pip install firecrawl-py
# or
npm install @mendable/firecrawl-js
```

```python
from firecrawl import FirecrawlApp
app = FirecrawlApp(api_key=os.environ['FIRECRAWL_API_KEY'])
```

## Scrape a Single Page

```python
# Scrape and convert to markdown
result = app.scrape_url(
    'https://example.com/article',
    params={
        'formats': ['markdown', 'html'],
        'onlyMainContent': True,          # strip nav/footer/ads
        'waitFor': 2000,                  # wait 2s for JS
    }
)

print(result['markdown'])
print(result['metadata']['title'])
print(result['metadata']['description'])
```

## Crawl a Website

```python
# Crawl entire site — returns all pages as markdown
crawl_result = app.crawl_url(
    'https://docs.example.com',
    params={
        'limit': 100,
        'scrapeOptions': {
            'formats': ['markdown'],
            'onlyMainContent': True,
        },
        'includePaths': ['/docs/*'],
        'excludePaths': ['/blog/*', '/changelog/*'],
    }
)

for page in crawl_result['data']:
    print(page['url'], page['markdown'][:200])
```

### Async Crawl (large sites)

```python
# Start crawl job
job = app.async_crawl_url('https://example.com', params={'limit': 1000})
job_id = job['id']

# Poll for completion
import time
while True:
    status = app.check_crawl_status(job_id)
    print(f"Status: {status['status']}, Pages: {status['completed']}/{status['total']}")
    if status['status'] == 'completed':
        break
    time.sleep(5)

# Get results
results = app.get_crawl_data(job_id)
```

## Map Website Structure

```python
# Get all URLs from a site (fast, no content)
sitemap = app.map_url('https://example.com', params={
    'includeSubdomains': False,
    'limit': 5000,
})

print(f"Found {len(sitemap['links'])} URLs")
for url in sitemap['links'][:20]:
    print(url)
```

## Search the Web

```python
# Search and get markdown content of results
results = app.search(
    'Claude Code best practices 2024',
    params={
        'limit': 5,
        'scrapeOptions': {'formats': ['markdown']},
    }
)

for r in results['data']:
    print(r['url'])
    print(r['markdown'][:500])
```

## Structured Data Extraction

```python
from pydantic import BaseModel

class ProductInfo(BaseModel):
    name: str
    price: float
    rating: float
    review_count: int
    in_stock: bool

# Extract structured data from a page
result = app.scrape_url(
    'https://shop.example.com/product/123',
    params={
        'formats': ['extract'],
        'extract': {
            'schema': ProductInfo.model_json_schema(),
            'prompt': 'Extract product information from this page',
        },
    }
)

product = ProductInfo(**result['extract'])
print(f"{product.name}: ${product.price} ({product.rating}★)")
```

## AI Agent for Autonomous Scraping

```python
# firecrawl-agent: let AI decide what to scrape
result = app.agent(
    prompt="Find the top 10 Python web scraping libraries, their GitHub stars, and last release date",
    params={'maxSteps': 20}
)

print(result['data'])
```

## Browser Automation

```python
# firecrawl-browser: interact with JS-heavy pages
result = app.scrape_url(
    'https://spa-example.com',
    params={
        'actions': [
            {'type': 'click', 'selector': '#load-more'},
            {'type': 'wait', 'milliseconds': 1000},
            {'type': 'scroll', 'direction': 'down', 'amount': 500},
            {'type': 'screenshot'},
        ],
        'formats': ['markdown', 'screenshot'],
    }
)
```

## CLI Usage

```bash
# Install
npm install -g firecrawl-cli

# Set API key
export FIRECRAWL_API_KEY=fc-...

# Scrape
firecrawl scrape https://example.com --format markdown

# Crawl
firecrawl crawl https://docs.example.com --limit 50 --output ./docs

# Map
firecrawl map https://example.com

# Search
firecrawl search "python web scraping" --limit 5

# Download files
firecrawl download https://example.com/report.pdf
```

## Batch Scraping

```python
# Scrape multiple URLs in parallel
urls = [
    'https://example.com/page1',
    'https://example.com/page2',
    'https://example.com/page3',
]

results = app.batch_scrape_urls(
    urls,
    params={'formats': ['markdown'], 'onlyMainContent': True}
)

for url, result in zip(urls, results['data']):
    print(f"{url}: {len(result['markdown'])} chars")
```

## Webhooks (async notifications)

```python
# Get notified when crawl completes
job = app.async_crawl_url(
    'https://example.com',
    params={
        'limit': 500,
        'webhook': 'https://myapp.com/api/crawl-complete',
    }
)
```

## Best Practices

1. Use `onlyMainContent: True` to strip navigation/ads
2. Set `waitFor` for JavaScript-rendered pages
3. Use `includePaths`/`excludePaths` to scope crawls
4. Use async crawl for sites > 50 pages
5. Cache results — avoid re-scraping unchanged pages
6. Respect `robots.txt` — Firecrawl handles this automatically
7. Use `map_url` first to understand site structure before crawling

## Sources
- firecrawl/firecrawl-cli
- firecrawl/firecrawl-agent
- firecrawl/firecrawl-browser
- firecrawl/firecrawl-crawl
- firecrawl/firecrawl-download
- firecrawl/firecrawl-map
- firecrawl/firecrawl-scrape
- firecrawl/firecrawl-search
