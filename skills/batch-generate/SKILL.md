---
name: batch-generate
description: Use when generating multiple Impulso business sites in parallel. Trigger phrases: "generate next N sites", "batch build", "batch generate", "build all remaining sites".
---

# batch-generate

## Overview

Orchestrate parallel generation of multiple Impulso business sites using sub-agents. One sub-agent per folder, each running the `new-site` workflow.

## Flow

1. Find all un-built folders (have `business-info.md`, no `astro.config.mjs`)
2. Show count and list to user
3. Confirm batch size (default: 5, max recommended: 10)
4. Invoke `superpowers:dispatching-parallel-agents` skill to dispatch one sub-agent per folder
5. Each sub-agent: reads `business-info.md`, runs `new-site` workflow, reports result
6. Collect all results, print summary table

## Detecting Un-Built Folders

```bash
for dir in /home/stein/Projects/impulso/*/; do
  if [ -f "$dir/business-info.md" ] && [ ! -f "$dir/astro.config.mjs" ]; then
    echo "$dir"
  fi
done
```

Present this list to user before proceeding.

## Sub-Agent Prompt Template

Each sub-agent gets this prompt:

```
You are building an Astro website for a Puerto Rico service business.
Folder: [folder-name]
Full path: /home/stein/Projects/impulso/[folder-name]/

Use the new-site skill to generate the complete site:
1. Read business-info.md
2. Detect category, select reference site
3. Generate all files
4. Run npm install && npm run build
5. Report: SUCCESS or FAILED with error details
```

## REQUIRED: Use dispatching-parallel-agents

**REQUIRED SUB-SKILL:** Use `superpowers:dispatching-parallel-agents` to dispatch the sub-agents. Do not run them sequentially.

## Summary Table Format

After all sub-agents complete:

```
## Batch Generate Results

| Folder | Category | Status | Notes |
|---|---|---|---|
| barberia-penolana | Barbería | ✅ Built | 4 pages |
| caribe-roofing | Handyman/Roofing | ✅ Built | 4 pages |
| bonilla-painting | Painting | ❌ Failed | npm install timeout |
| del-toro-rain-gutters | Roofing | ✅ Built | 4 pages |
| eco-clean-pr | Cleaning | ✅ Built | 5 pages |

Results: 4 built, 1 failed
Next: Run audit-site on built sites, investigate failed builds
```

## After Batch

Suggest running `audit-site` on the batch to catch any issues before moving on.
