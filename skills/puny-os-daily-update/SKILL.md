---
name: puny-os-daily-update
description: Use when the user asks to update, write, or review their daily update in puny-os. Triggers on "update my daily", "daily update", "puny-os update", or "standup update".
---

# Puny OS Daily Update

## Overview

Syncs Stein's daily update in the puny-os repo with the latest activity from the Obsidian vault (`~/My Notes`). The daily update is a standup-style file that tracks what was done, what's planned, blockers, and notes.

## File Locations

- **Daily update:** `/home/stein/Projects/puny-os/01_Daily/YYYY-MM-DD/stein.md`
- **Template:** `/home/stein/Projects/puny-os/templates/daily_update.md`
- **Vault root:** `/home/stein/My Notes/`

## Daily Update Format

```markdown
# Daily Update – Stein

## What I did yesterday
-

## What I'm doing today
-

## Blockers
-

## Attachments
-

## Notes
-
```

## Process

### 1. Gather Context (parallel)

Scan these vault locations for recent changes:

| Location | What to look for |
|----------|-----------------|
| `Daily Notes/` | Today's and recent daily notes — tasks, schedule, reflections |
| `Tasks/This Week.md` | Current weekly task list across all clients/projects |
| `Projects/` | Project status updates (especially `Puny.bz.md`) |
| `Puny Platform/` | Docs updates (Desk, Features, Roadmap, Overview, Ops) |
| `CRM/` | New clients, deals, meetings, people, onboarding progress |
| `Ideas/` | New product/feature ideas |
| `Founder Dashboard.md` | Priority changes |

Also read the **current daily update file** to see what's already captured.

### 2. Cross-Reference

Compare vault activity against what's already in the daily update. Identify:
- Work done today not yet listed
- New blockers or resolved blockers
- Client/CRM updates (new leads, meetings, deal changes)
- Documentation or spec work
- Task list changes worth noting

### 3. Update the File

Edit the daily update — don't overwrite, **add to existing content**:

- **What I did yesterday** — only update if the file was just created or is clearly wrong
- **What I'm doing today** — add new items found in the vault
- **Blockers** — add new blockers, keep existing ones that are still active
- **Notes** — add context, decisions, or anything worth remembering

### 4. Flag Gaps

After updating, review for missing items:
- Are there tasks in `This Week.md` that should be reflected?
- Are there client sections with no tasks that should have some?
- Any blockers mentioned in daily notes not captured here?
- Any meetings scheduled this week not noted?

Tell the user what might be missing so they can decide.

## Common Mistakes

- **Overwriting existing content** — always add, don't replace what's already there
- **Missing CRM activity** — new clients, deals, and meetings are easy to miss; always check `CRM/`
- **Ignoring documentation work** — spec/docs updates are real work and belong in the daily update
- **Stale blockers** — check if existing blockers have been resolved before keeping them
