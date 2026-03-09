---
name: enrich-business
description: Use when a business-info.md file is missing critical fields (phone, hours, services, address, category) before generating a site. Trigger phrases: "enrich [folder]", "fill in missing data for X", "update business info for X".
---

# enrich-business

## Overview

Fill data gaps in `business-info.md` using web search before generating a site. Never generate a site without at least phone, category, and municipality.

## Critical Fields Checklist

Run through these in order — stop when all are found or sources exhausted:

| Field | Required? | Example |
|---|---|---|
| Phone | Yes | `787-555-1234` |
| Category | Yes | `Plomería`, `Barbería`, `CPA` |
| Municipality | Yes | `Bayamón`, `San Juan` |
| Hours | High | `Lun-Vie 8AM-5PM` |
| Services | High | List of 3-5 services |
| Service Area | Medium | `Norte de PR`, `Todo PR` |
| Rating / Reviews | Low | `4.8 ⭐ (24 reseñas)` |

## Flow

1. Read `[folder]/business-info.md`
2. Identify which critical fields are missing
3. If phone, category, and municipality are all present → skip search, proceed to `new-site`
4. Web search for each missing field using this priority order:
   - Yelp Puerto Rico: `site:yelp.com "[business name]" Puerto Rico`
   - Google Maps: `"[business name]" Puerto Rico phone hours`
   - Facebook: `site:facebook.com "[business name]" Puerto Rico`
   - Instagram: `site:instagram.com "[business name]"`
   - ClasificadosOnline: `site:clasificadosonline.com "[business name]"`
5. Present findings clearly — show what was found and what is still unknown
6. Ask user to confirm before writing any changes
7. Update `business-info.md` with confirmed data only

## Output Format

After searching, present:

```
## Enrichment Results: [folder-name]

Found:
- Phone: 787-XXX-XXXX (source: Yelp)
- Hours: Lun-Vie 8AM-5PM (source: Facebook)

Still unknown:
- Service Area (not found in any source)

Confirm to update business-info.md? [y/n]
```

## Rules

- Never write to `business-info.md` without explicit user confirmation
- Never guess or fabricate data — mark as "not found" if unavailable
- If phone cannot be found anywhere, tell user — do not generate site
- Keep existing data intact; only add missing fields
