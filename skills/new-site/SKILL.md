---
name: new-site
description: Use when creating a complete Astro website for a business folder that has business-info.md but no site yet. Trigger phrases: "build site for [folder]", "generate [folder]", "create site for X".
---

# new-site

## Overview

Generate a complete Astro + Tailwind + React website for a business folder by reading its `business-info.md`, detecting the business category, using the closest completed reference site as a structural example, then building and verifying.

## Flow

1. Identify target folder (from user message, or list folders with `business-info.md` but no `astro.config.mjs`)
2. Read `[folder]/business-info.md`
3. Detect category using the table below
4. Read 3-5 key files from the reference site as structure examples
5. Generate all site files adapted to this business's data
6. Run: `cd [folder] && npm install && npm run build`
7. Report success or surface errors with file:line references

## Category Detection → Reference Site

Scan `business-info.md` for these keywords:

| Keywords found | Category | Reference site |
|---|---|---|
| plomero, plomería, destapes, plumbing | Plomero | `munoz-plumbing` |
| electricista, electric, eléctrico | Electricista | `donca-electric-service` |
| AC, aire acondicionado, HVAC, refrigeración | HVAC | `quintana-air-conditioning` |
| fumigador, plagas, pest control, exterminador | Fumigador | `control-plagas-norte` |
| handyman, roofing, techos, sellado, techo | Handyman/Roofing | `quinones-handyman` |
| barbería, barber, corte, barbershop | Barbería | `herman-hair-design` |
| nail, nails, uñas, salon de uñas, beauty | Nail/Salon | `foxy-beauty` |
| landscaping, paisajismo, jardín, podadora, grama | Landscaping | `alex-cosme-landscaping` |
| CPA, contador, contabilidad, planillero, taxes | CPA/Contador | `cpa-karla` |
| limpieza, cleaning, cleaner, hogar limpio | Cleaning | `serene-cleaner` |
| fotógrafo, photography, foto, studio, bodas | Photography | `senor-fotografo-pr` |
| pintura, painting, painter, pintor | Painting | `alex-cosme-landscaping` |

If no keyword matches clearly, ask the user which category before proceeding.

## Files to Generate

**Always (all sites):**
- `package.json`
- `astro.config.mjs`
- `tailwind.config.cjs`
- `tsconfig.json`
- `src/layouts/BaseLayout.astro`
- `src/pages/index.astro`
- `src/pages/servicios.astro`
- `src/pages/contacto.astro`
- `src/styles/global.css`

**Category-specific additional pages:**
- Plomero / Electricista / HVAC / Fumigador → `src/pages/emergencias.astro`
- Landscaping / Handyman / Roofing / Painting → `src/pages/proyectos.astro`
- Photography → `src/pages/portafolio.astro`
- CPA → `src/pages/recursos.astro`

## Content Rules

- Spanish primary, English secondary
- Business phone number must appear in: hero CTA, footer, contact page (minimum 3 locations)
- Municipality and service area prominent on home page
- All `<img>` tags need descriptive `alt` attributes in Spanish
- `<html lang="es">` always set in BaseLayout
- Mobile-first — test viewport at 375px width mentally
- WCAG AA color contrast — use the reference site's color scheme as baseline

## Reference Reading Strategy

From the reference site, read in this order:
1. `src/layouts/BaseLayout.astro` — understand the base structure
2. `src/pages/index.astro` — understand home page pattern
3. `src/pages/contacto.astro` — understand contact pattern
4. `tailwind.config.cjs` — understand color scheme to adapt

Do NOT copy business-specific text. Copy only structure, components, and patterns.

## Success Criteria

- `npm run build` exits 0 with no errors
- All required pages present
- Phone number in >= 3 locations
- No placeholder text ("TBD", "TODO", "Lorem ipsum")
