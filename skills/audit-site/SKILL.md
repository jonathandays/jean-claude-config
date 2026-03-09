---
name: audit-site
description: Use when verifying a generated site meets quality, accessibility, and content standards. Trigger phrases: "audit [folder]", "check [folder]", "check all sites", "verify [folder]".
---

# audit-site

## Overview

Run a structured quality check on a generated Impulso business site. Covers: build success, required pages, phone presence, accessibility basics, and content completeness.

## Flow

1. Identify target folder(s) — one folder, or all folders with `astro.config.mjs`
2. Run each check below
3. Output pass/fail per check with file:line references for failures
4. Summarize: N passed, N warnings, N failed

## Checks

Run ALL of these for every audit:

| # | Check | Pass Condition | Severity |
|---|---|---|---|
| 1 | Build | `npm run build` exits 0, no errors | FAIL |
| 2 | Required pages | `index.astro`, `servicios.astro`, `contacto.astro` all present | FAIL |
| 3 | Phone presence | Phone number string appears in ≥ 3 source files | FAIL |
| 4 | Address/location | Municipality or street address present in source | FAIL |
| 5 | Language attr | `<html lang="es">` in layout file | WARN |
| 6 | Image alt text | All `<img>` tags have non-empty `alt` attribute | WARN |
| 7 | No placeholders | No "Lorem ipsum", "TBD", "TODO", "PLACEHOLDER", "xxx" | FAIL |
| 8 | CTA button | At least one button/link with phone or "Contactar" on index | FAIL |
| 9 | Mobile viewport | `<meta name="viewport"...>` present in layout | WARN |

## Running Check #1 (Build)

```bash
cd [folder] && npm run build 2>&1
```

Look for: "error" lines, unclosed tags, undefined variables, missing imports.

## Running Checks #2-9 (Static Analysis)

Use Read and Grep tools — do NOT run a linter. Check manually:
- Grep for phone pattern: `\d{3}-\d{3}-\d{4}` across all `.astro` files
- Grep for `alt=""` or `alt=` without value on `<img>` tags
- Grep for placeholder strings
- Read `src/layouts/BaseLayout.astro` for lang and viewport

## Output Format

```
## Audit: [folder-name]

✅  Build: passes (0 errors)
✅  Required pages: index, servicios, contacto present
❌  Phone presence: found in 1 file only (contacto.astro:34) — need ≥ 3
✅  Address: "Bayamón, Puerto Rico" found
⚠️   Language attr: missing lang="es" on <html> (BaseLayout.astro:3)
⚠️   Image alt text: 2 images missing alt (index.astro:44, index.astro:88)
✅  No placeholders
✅  CTA button present
✅  Mobile viewport meta present

Result: 6 passed, 2 warnings, 1 failed
```

## Batch Auditing

To audit all sites with `astro.config.mjs`:

```bash
find /home/stein/Projects/impulso -maxdepth 2 -name "astro.config.mjs" | sed 's|/astro.config.mjs||'
```

Run the full check list on each folder. Print one result block per folder, then a final summary table.
