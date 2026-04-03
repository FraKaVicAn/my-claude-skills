---
name: webapp-testing
description: Test local web applications with Playwright — screenshot capture, DOM inspection, UI automation, and browser log review via Python scripts
origin: ComposioHQ/awesome-claude-skills
tools: Bash, Read, Write
---

# Web App Testing Skill

Automate and validate local web application behavior using Playwright: capture screenshots, inspect the DOM, trigger interactions, and review browser console logs.

## When to Activate

- Testing a local frontend after changes
- Debugging UI behavior that's hard to inspect manually
- Capturing screenshots for documentation or regression comparison
- Validating form flows, navigation, or dynamic content

## Decision Tree

| Situation | Approach |
|-----------|----------|
| Static HTML file | Read file to find selectors, write Playwright script |
| Dynamic app, server not running | Use `scripts/with_server.py` to start server, then automate |
| Server already running | Screenshot → inspect DOM → identify selectors → interact |

## Critical Rule

**Always call `page.wait_for_load_state('networkidle')` before inspecting dynamic apps.** Skipping this is the #1 cause of empty or incomplete DOM reads.

## Core Script Pattern

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("http://localhost:3000")
    page.wait_for_load_state("networkidle")  # REQUIRED for SPAs

    # Screenshot
    page.screenshot(path="screenshot.png", full_page=True)

    # Find elements
    title = page.locator("h1").first.text_content()
    print(f"Title: {title}")

    # Interact
    page.fill("input[name='email']", "test@example.com")
    page.click("button[type='submit']")
    page.wait_for_load_state("networkidle")
    page.screenshot(path="after_submit.png")

    browser.close()
```

## Starting a Dev Server

```bash
# with_server.py manages server lifecycle
python scripts/with_server.py --server "npm run dev" --port 3000 -- python test_script.py

# Two servers (backend + frontend)
python scripts/with_server.py \
  --server "uvicorn app:app --port 8000" \
  --server "npm run dev --port 3000" \
  -- python test_e2e.py
```

## Console Log Capture

```python
logs = []
page.on("console", lambda msg: logs.append(f"[{msg.type}] {msg.text}"))
page.goto("http://localhost:3000")
page.wait_for_load_state("networkidle")
errors = [l for l in logs if "[error]" in l]
```

## Installation

```bash
pip install playwright
playwright install chromium
```

## Links

- [Playwright Python](https://playwright.dev/python/)
- [github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)
