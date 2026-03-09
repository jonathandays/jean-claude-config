---
name: puny-playwright
description: Use when running Playwright browser automation for Puny.bz — testing pages, taking screenshots, verifying UI. Always use headless mode by default.
triggers:
  - "test with playwright"
  - "take a screenshot"
  - "verify the page"
  - "open the browser"
  - "playwright"
---

# Puny.bz Playwright Usage

## Default: Always Headless

**Always run Playwright in headless mode** unless you have a clear reason not to (e.g., debugging a visual interaction that requires seeing the browser). Never open a visible browser window without a specific reason.

When using the MCP Playwright tools (`mcp__plugin_playwright_playwright__*`), they run headless by default — do not change this behavior.

When writing Playwright scripts or code:
```typescript
const browser = await chromium.launch({ headless: true }); // always headless
```

## When Non-Headless Is Acceptable
- Debugging a visual glitch that can't be reproduced or described in a screenshot
- Demonstrating UI to the user in real time (user explicitly asks for it)

## Local Development URLs
- Portal: `http://localhost:3000/p/`
- Public cards: `http://cards.localhost:3000/{slug}`
- Django API: `http://localhost:8000`

## Common Tasks

### Take a screenshot
Use `mcp__plugin_playwright_playwright__browser_take_screenshot` (headless by default via MCP).

### Navigate and snapshot
1. `mcp__plugin_playwright_playwright__browser_navigate` to the URL
2. `mcp__plugin_playwright_playwright__browser_snapshot` to get accessibility tree
3. Use snapshot refs to interact with elements

### Verify a page loaded correctly
1. Navigate to URL
2. Take snapshot
3. Check for expected text/elements in snapshot output
