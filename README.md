# ENES100 Tool Check

**A touch-friendly web app for tracking hand tools in the ENES100 engineering labs at the University of Maryland.** Staff tap through photos of each lab table's tool chest, mark which tools are missing, and submit. Each submission is recorded in a timestamped history in Firebase Realtime Database.

> **Archived for my portfolio.** This repository is a frozen snapshot of the app as of September 2025. The original repository is [umdenes100/enes100-toolcheck](https://github.com/umdenes100/enes100-toolcheck). The app was used inside the ENES100 lab management platform (Velma) and was embedded in the [course website](https://github.com/camwharff/enes100-website).

**Timeline:** Built April 2025 · Responsive redesign September 2025

**Role:** Sole designer and developer

**Stack:** JavaScript (ES modules) · HTML/SVG · CSS · Vite · Firebase Realtime Database · GitHub Actions · GitHub Pages

---

## Overview

The ENES100 labs have 10 student work tables. Each has a tool chest with 10 drawers holding a fixed set of 73 tools: screwdrivers and wrenches, measuring tools, clamps, hammers and saws, electrical tools, PPE, and more. That makes **730 tracked tools** in total. Tools were organized in a physical kanban/shadow-board system, where each tool has its own outlined spot so a missing one is obvious. But checking it was manual, and nothing recorded what went missing or how often.

I built the tool check app as a digital version of that system. It mirrors the physical drawers so lab staff can check a table in seconds, and every check leaves a history in the database.

The tool check system was presented as part of the poster "Strategies for Managing a High-Throughput Academic Makerspace" at ISAM 2025 (International Symposium on Academic Makerspaces). See the [VELMA README](https://github.com/camwharff/enes100-velma#conference-presentation) for the poster.

## How it works

1. **Pick a table.** Staff select one of the 10 lab tables.
2. **Open a drawer.** A photo of the tool chest is overlaid with an invisible SVG hotspot on each drawer. Tapping a drawer opens a photo of that drawer's contents.
3. **Mark missing tools.** Each tool in the drawer photo is a traced SVG outline that sits exactly over the real tool. Tapping a tool toggles it between present and missing, and missing tools are highlighted. Each tap writes the new status to Firebase immediately, so the current state survives a page reload or a switch between devices.
4. **Submit.** Submitting logs the status of every tool at that table under the same timestamp. Each tool's present or missing count is incremented with a Firebase transaction.

When a drawer is opened, the app reads the current status for that drawer from Firebase. Tools already marked missing appear highlighted, so the next person sees where the last check left off.

## Data model

Firebase Realtime Database is organized as a tree. Every tool's record is reachable by table, drawer, and tool name:

```
tables/
└── Table 1/
    └── hms/                      # drawer (hammers, mallets & saws)
        └── hammer/
            ├── present: "true" | "false"
            ├── presentLog/       # { <timestamp>: ISO time, ... }
            ├── missingLog/       # { <timestamp>: ISO time, ... }
            ├── presentOccurences: <count>
            └── missingOccurences: <count>
```

With this structure, it's simple to query any single tool's history, a whole drawer, or a whole table. It's also easy to spot tools that go missing repeatedly by comparing the occurrence counters.

## Design decisions

- **Photos over lists.** Staff recognize a drawer by how it looks, not by a list of tool names. Putting traced SVG outlines on real drawer photos makes the digital check match the physical shadow-board system staff already used.
- **Instant writes plus snapshot logs.** Toggles are saved right away, so no work is lost if a tablet is closed mid-check. History is only logged on Submit, so each log entry represents one complete table check, not a stream of individual taps.
- **Transactions for counters.** The present and missing counters use `runTransaction`, so two staff checking at the same time can't overwrite each other's increments.
- **Built for lab tablets.** The September 2025 redesign added separate layouts for phones and tablets in portrait and landscape. It also locked pinch-zoom so taps register reliably on the touch targets.
- **Lightweight build.** The app uses plain JavaScript modules and SVG with no front-end framework, so it loads quickly on the lab's tablets.

## Project structure

```
index.html              # All views: table select, chest overview, 10 drawer views (SVG hotspots)
style.css               # Layout and responsive rules (phone/tablet × portrait/landscape)
src/
├── main.js             # Navigation, Firebase reads/writes, toggle and submit logic
├── firebaseConfig.js   # Firebase app and Realtime Database setup
└── showhide.js         # View show/hide helpers
images/                 # Photos of the tool chest and each drawer
public/                 # Interstate font files
.github/workflows/      # Build and deploy to GitHub Pages on push to main
```

## Running locally

```bash
git clone https://github.com/camwharff/enes100-toolcheck.git
cd enes100-toolcheck
npm install
npm run dev
```

> **Note:** `src/firebaseConfig.js` points to the ENES100 production database. To experiment, swap in the config for your own Firebase project so you don't change real lab data.

## What I'd improve

- **Authentication.** The app writes to the database without signing users in. Adding Firebase Auth for lab staff and tightening the database rules would stop anyone with the URL from changing tool statuses.
- **Await the submit.** The Submit handler returns to the home screen before all of its log writes finish. Collecting the writes and awaiting them together would let the app confirm the check was saved.
