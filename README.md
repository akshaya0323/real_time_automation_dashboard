# NEXUS RPA — Enterprise Control Terminal 2026

> A high-performance, real-time RPA (Robotic Process Automation) stream dashboard with frozen-state analytics, virtual grid rendering, and Chart.js-powered data visualization.

![NEXUS RPA Terminal](https://img.shields.io/badge/NEXUS_RPA-Enterprise_Terminal-00d4ff?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48cG9seWdvbiBwb2ludHM9IjEyLDIgMjIsNyAyMiwxNyAxMiwyMiAyLDE3IDIsNyIgc3Ryb2tlPSIjMDBkNGZmIiBzdHJva2Utd2lkdGg9IjEuNSIgZmlsbD0ibm9uZSIvPjwvc3ZnPg==)
![Chart.js](https://img.shields.io/badge/Chart.js-4.4.1-FF6384?style=for-the-badge&logo=chartdotjs)
![Vanilla JS](https://img.shields.io/badge/Vanilla_JS-ES2022-F7DF1E?style=for-the-badge&logo=javascript)
![No Dependencies](https://img.shields.io/badge/Build_Tool-None-brightgreen?style=for-the-badge)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [How It Works](#how-it-works)
- [Analytics Dashboard](#analytics-dashboard)
- [Bug Fixes Applied](#bug-fixes-applied)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Technical Architecture](#technical-architecture)
- [Data Schema](#data-schema)
- [Configuration](#configuration)
- [Browser Support](#browser-support)
- [License](#license)

---

## Overview

NEXUS RPA is a **zero-dependency, single-HTML-file enterprise dashboard** that ingests a live RPA project telemetry stream, renders a high-performance virtual grid of thousands of rows at 60 fps, and — when the stream is **frozen/paused** — surfaces a full **Chart.js analytics overlay** with 5 chart types and 4 aggregate KPIs, all computed from the frozen snapshot in real time.

The application simulates a production-grade stream pipeline: data arrives in batches every 200ms, is upserted into a live memory pool, filtered and sorted on the fly, and rendered using virtual DOM techniques so even 50,000+ row datasets remain buttery smooth.

---

## Features

### Core Stream Engine
- **Live telemetry firehose** — batches of 5–50 rows injected every 200ms from `dataStream.js`
- **Upsert memory pool** — rows keyed by `internal_uid`, updated in place rather than appended, preventing pool bloat
- **Anomaly detection** — rows with annual savings swings > $1M trigger a visual flash animation and increment the anomaly KPI counter
- **Pause queue** — incoming batches are buffered while the stream is frozen; on resume, all queued rows are flushed in a single batch

### Virtual Grid
- **Full virtual scrolling** — only visible rows (± 5 overscan) are rendered to the DOM, regardless of total row count
- **Multi-column sort** — click any column header to sort; click again to reverse; click a third time to remove. Multiple sort keys stack with Shift-click semantics
- **Fuzzy multi-token search** — space-separated tokens are all matched independently across project name, company ID, partner, and country
- **4 filter dropdowns** — Type, Department, Industry, Status — all populated dynamically from the live CSV data
- **Row inspector** — click any row (even while paused) to open a full relational attribute map in a modal overlay

### KPI Strip
Real-time counters across 6 metrics, updated on every tick:
- Rows Processed
- Active Robots (cumulative deployed)
- Global Savings (cumulative USD)
- Anomalies (critical macro shifts)
- Avg ROI (live pool average)
- Hours Saved (employee hours, cumulative)

### Analytics View (Frozen State Only)
Triggered by the **📊 Analytics View** button — only available when the stream is paused. Renders 5 Chart.js visualizations:

| Chart | Type | Data |
|-------|------|------|
| Status Distribution | Doughnut | Project count by status |
| Top Industries by Savings | Bar | Top 8 industries by cumulative savings |
| Automation Types by Count | Pie | Top 8 automation types by project count |
| ROI Distribution | Bar (histogram) | Projects bucketed into 6 ROI ranges |
| Dept: Avg Savings vs Avg ROI | Dual-axis Bar+Line | Top 10 departments |

### Panel Toggles
Three panels togglable via the widget bar:
- **Data Grid** — the main virtual scroll table
- **Dept Chart** — a mini bar chart of savings by department (bottom of grid)
- **Infrastructure** — real-time render FPS, queue depth, pool size, filtered row count, sort key count, tick count, AI-enabled %

Panel states persist to `localStorage`.

### Snapshot Export
Download the current filtered + sorted view pool as a `.csv` file with a timestamped filename.

---

## Project Structure

```
nexus-rpa-terminal/
├── index.html              # Complete application (HTML + CSS + JS)
├── dataStream.js           # Telemetry pipeline engine
├── automation_projects.csv # RPA project dataset (50k+ rows)
└── README.md               # This file
```

> **Everything is self-contained.** The entire app logic, styles, and markup live in `index.html`. No build step, no bundler, no framework.

---

## Getting Started

### Prerequisites
- Any modern web browser (Chrome 90+, Firefox 88+, Edge 90+, Safari 15+)
- A local web server (required for `fetch()` to load the CSV — `file://` protocol will be blocked by CORS)

### Option 1 — VS Code Live Server (Recommended)
1. Install the [Live Server extension](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer)
2. Open the project folder in VS Code
3. Right-click `index.html` → **Open with Live Server**
4. Browser opens at `http://127.0.0.1:5500`

### Option 2 — Python HTTP Server
```bash
# Python 3
python -m http.server 8080

# Then open http://localhost:8080
```

### Option 3 — Node.js
```bash
npx serve .
# Then open http://localhost:3000
```

### Option 4 — Deploy to GitHub Pages
1. Push the repo to GitHub
2. Go to **Settings → Pages → Source → Deploy from branch → `main` / `root`**
3. GitHub Pages will serve `index.html` directly

> **Important:** `automation_projects.csv` must be in the same directory as `index.html`. The stream engine fetches it via a relative URL (`./automation_projects.csv`).

---

## How It Works

### Stream Pipeline

```
automation_projects.csv
        │
        ▼
dataStream.js (parseCSV)
        │
   memoryPool[]        ← All rows held in RAM
        │
   setInterval 200ms
        │
   randomize batch (5–50 rows)
   mutate values (anomaly 5% chance)
        │
        ▼
   callback(incomingBatch)
        │
        ▼
   processBatch()
   ├── if PAUSED → push to pauseQueue[]
   └── else → applyBatch()
                   │
              upsert into STATE.pool[]
              update KPIs
              recomputeView() (filter + sort)
              scheduleRender() (rAF)
              updateDeptChart()
```

### Virtual Grid Rendering

The grid uses a classic **windowed rendering** pattern:

```
Total rows: N  (could be 50,000+)
Visible viewport height: ~600px
Row height: 38px
Visible rows: ~16

Only rows [scrollTop/38 - 5 ... scrollTop/38 + visible + 5] are in the DOM.
All others exist only in STATE.viewPool[].
```

A single `requestAnimationFrame` is scheduled per data tick; scroll events also schedule renders. This achieves consistent 60fps even with large datasets.

---

## Analytics Dashboard

The analytics overlay is only accessible when the stream is **paused** (frozen). This is intentional: the charts represent a **point-in-time snapshot** of the frozen pool, not a live view.

### Opening
1. Click **⏸ Pause Stream** — stream freezes, incoming rows queue up
2. Click **📊 Analytics View** — overlay opens with 5 charts computed from the current `STATE.viewPool`

### Closing
- Click the **✕** button in the overlay header
- Click the **backdrop** (outside the panel)
- Press **Escape**
- Click **📊 Analytics View** again (toggle)

### Re-opening
After closing, clicking **📊 Analytics View** again fully rebuilds all charts from the current frozen state. All previous Chart.js instances are properly destroyed before new ones are created — no "Canvas already in use" errors.

---

## Bug Fixes Applied

This version resolves **6 bugs** present in the original codebase:

### Bug 1 — Dept Chart Re-open Failure *(Primary Issue)*
**Symptom:** Closing the Dept Chart panel via the toggle chip and re-opening it caused a blank chart or silent error.

**Root cause:** `applyWidgetStates()` had a broken ternary:
```js
// BROKEN — 'visible' is always truthy, condition never reached
document.getElementById('dept-panel').className = 'visible' ? ... : ...
```
The panel was never actually hidden, so the old chart instance remained registered on the canvas. Re-toggling tried to create a new `Chart` on an already-registered canvas → Chart.js silently failed.

**Fix:** Rewrote `applyWidgetStates()` with clean `if/else` logic. When toggling the dept panel **off**, the chart instance is explicitly `destroy()`ed and the reference is nulled.

---

### Bug 2 — Orphaned Chart.js Instances on Canvas
**Symptom:** After rapid toggling or edge-case close sequences, `deptChartInst` could become out of sync with what Chart.js had internally registered.

**Root cause:** No check was performed against Chart.js's own registry before creating a new instance.

**Fix:** Added `Chart.getChart(canvas)` check before every chart creation. This uses the authoritative Chart.js API to detect any orphan instance and destroys it before proceeding.

---

### Bug 3 — Analytics Charts Not Destroyed on Close
**Symptom:** Closing the analytics overlay via Escape or backdrop click, then reopening caused charts to sometimes not render, or render incorrectly.

**Root cause:** `closeAnalytics()` only removed the `.active` CSS class. It never destroyed the Chart.js instances. The `STATE.analyticsCharts` array was then empty on the next open (because `forEach destroy` ran first in `openAnalytics`), but Chart.js still held the old canvas registrations internally.

**Fix:** `closeAnalytics()` now explicitly destroys all chart instances and clears the array before hiding the overlay.

---

### Bug 4 — Analytics Canvas ID Orphan Race
**Symptom:** Re-opening analytics after a prior session could throw Chart.js warnings and render blank charts.

**Root cause:** Between `destroy()` calls and `innerHTML = ''` + re-injection, stale Chart.js canvas registrations could persist.

**Fix:** Added a pre-scan of all 5 analytics canvas element IDs using `Chart.getChart()` immediately before rebuilding the DOM, destroying any orphans found.

---

### Bug 5 — Duplicate `deptChartInst` References
**Symptom:** Confusing state — `STATE.deptChartInst` (in STATE object) and `let deptChartInst` (standalone) both existed, but only the standalone was ever read or written.

**Root cause:** Dead code in STATE object.

**Fix:** Removed `deptChartInst` from the STATE object. One canonical variable, zero ambiguity.

---

### Bug 6 — Analytics Button Not a Toggle
**Symptom:** Clicking **📊 Analytics View** while the overlay was already open had no effect (rebuild ran silently under the visible overlay).

**Root cause:** No guard check at the entry of `openAnalytics()`.

**Fix:** Added `if (overlay.classList.contains('active')) { closeAnalytics(); return; }` at the top of `openAnalytics()`.

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Escape` | Close any open overlay (Inspector or Analytics) |
| `Ctrl/Cmd + P` | Toggle Pause / Resume stream |
| `Ctrl/Cmd + E` | Trigger Snapshot Export |

---

## Technical Architecture

```
┌─────────────────────────────────────────────────────┐
│                    index.html                        │
│                                                      │
│  ┌──────────────┐   ┌──────────────────────────┐    │
│  │  dataStream  │   │        STATE ENGINE        │    │
│  │  .js         │──▶│  pool[], viewPool[]        │    │
│  │  200ms ticks │   │  sortKeys[], widgetStates  │    │
│  └──────────────┘   └────────────┬───────────────┘   │
│                                  │                    │
│         ┌────────────────────────┼──────────────┐    │
│         ▼                        ▼              ▼    │
│  ┌─────────────┐   ┌──────────────────┐  ┌─────────┐│
│  │ Virtual     │   │  Chart.js        │  │  KPI    ││
│  │ Grid (rAF)  │   │  Analytics Panel │  │  Strip  ││
│  │ 60fps       │   │  (paused only)   │  │         ││
│  └─────────────┘   └──────────────────┘  └─────────┘│
│         │                                            │
│  ┌──────▼────────────────────────────────────────┐  │
│  │  Filter Engine (search + 4 dropdowns + sort)  │  │
│  └───────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### Key Design Decisions

| Decision | Rationale |
|----------|-----------|
| Single HTML file | Zero build complexity; works from any static host |
| Virtual scrolling | Handles 50k+ rows without DOM bloat |
| Chart.js (CDN) | Only charting library; specified requirement |
| `Chart.getChart()` for cleanup | Authoritative Chart.js API to prevent canvas re-use errors |
| Upsert pattern for pool | Same row updated in-place; prevents unbounded pool growth |
| `requestAnimationFrame` scheduling | Decouples data ingestion from render cycle |
| Pause queue | No data loss during inspection; flush on resume |
| `localStorage` for widget state | Panel visibility persists across refreshes |

---

## Data Schema

The CSV (`automation_projects.csv`) must have these columns:

| Column | Type | Description |
|--------|------|-------------|
| `project_id` | string | Unique project identifier |
| `company_id` | string | Parent company |
| `project_name` | string | Human-readable name |
| `start_date` | string | ISO date |
| `completion_date` | string | ISO date or blank |
| `project_status` | string | `Active` \| `Completed` \| `Failed` \| `Paused` |
| `automation_type` | string | RPA category |
| `robots_deployed` | number | Count of deployed bots |
| `budget_usd` | number | Project budget |
| `annual_savings_usd` | number | Yearly savings generated |
| `roi_percent` | number | Return on investment % |
| `department` | string | Owning department |
| `implementation_partner` | string | Vendor/partner name |
| `country` | string | Country of operation |
| `industry` | string | Industry vertical |
| `employee_hours_saved` | number | Annual hours freed |
| `ai_enabled` | string | `Yes` \| `No` |
| `cloud_deployment` | string | `Yes` \| `No` |

---

## Configuration

All tunable constants are at the top of the `<script>` block in `index.html`:

```js
const ROW_H = 38;          // Row height in pixels (must match CSS --row-h)
```

In `dataStream.js`:
```js
setInterval(() => { ... }, 200);          // Tick rate (ms)
const batchSize = randomRange(5, 50);     // Rows per tick
const isAnomaly = Math.random() > 0.95;  // Anomaly probability (5%)
```

---

## Browser Support

| Browser | Version | Status |
|---------|---------|--------|
| Chrome | 90+ | ✅ Full support |
| Edge | 90+ | ✅ Full support |
| Firefox | 88+ | ✅ Full support |
| Safari | 15+ | ✅ Full support |
| Opera | 76+ | ✅ Full support |

> Requires: `fetch`, `requestAnimationFrame`, `CSS Custom Properties`, `ES2020` (`?.`, `??`, `BigInt` not used — ES2018 syntax only)

---
