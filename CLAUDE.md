# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A data repository for ETC (Electronic Toll Collection) highway billing records from the "e高速" (yigaosu) system in Guangdong, China. Data is automatically synced and committed by the Go program `yigaosusync`.

## Architecture

The repo serves as both a data store (JSONP files) and a static web app (HTML files that load the JSONP data via script tags).

### Data Files (JSONP format)

All data files use a JSONP-like wrapper pattern: `__callbackName([...data, null])`.

- `cards.js` — ETC card list, callback `__yigaosuCards`. Contains `{CardNo}` objects.
- `{CardNo}.js` (e.g. `37012104230246454685.js`) — Billing records for a specific card, callback `__yigaosuBills`. Each record has `{Amount, BeginAt, EndAt, From, To}`.
- `routes.js` — Route metadata between toll stations, callback `__yigaosuRoutes`. Each entry has `{From, To, FromCoords, ToCoords, DrivingDistance}`.

These files are written by yigaosusync's `main.go` using `writeJS()` and read back using `readJS()`. The trailing `null` in each array is intentional (the parent program's write format).

### Web UI

- `index.html` — Main billing viewer. React SPA using Arco Design, dayjs, big.js. All dependencies loaded via `esm.sh` CDN (no build step). Features: card selection, date range filtering, route filtering, monthly summary, cost calculations with driving distance from `routes.js`.
- `routes.html` — Route editor for maintaining `routes.js`. Uses AMap (高德地图) API for geocoding and driving distance lookup. Outputs updated JSONP to paste back into `routes.js`.

### How Data Flows

1. `yigaosusync` (Go) fetches billing data from yigaosu API
2. Writes/updates `cards.js` and `{CardNo}.js` in this directory
3. Commits and pushes changes to this repo via go-git with SSH auth
4. `index.html` loads the JSONP files as `<script>` tags, which call global `window.__yigaosuCards` / `window.__yigaosuBills` / `window.__yigaosuRoutes` callbacks

## Key Conventions

- The `To` field in bill records includes a province prefix (e.g. "广东省广东勒流站") while `From` does not — the `fix()` function in `index.html` handles deduplication of repeated province names.
- Amounts are strings (not numbers) to preserve precision; `big.js` is used for arithmetic in the UI.
- `useLocalStorage` hook persists UI state (selected tab, date range, route filter, hideOne toggle).
- No build tools, package manager, or dev server — open HTML files directly in a browser.
