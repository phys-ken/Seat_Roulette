# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Seat_Roulette (座席ルーレット) is a static, client-side web application suite for randomizing classroom seating arrangements and group assignments in Japanese classrooms. It is designed for projection on a blackboard and requires no installation, server, or build process.

## Development

**Run a local dev server (optional):**
```bash
python _dev_server.py
# Serves at http://localhost:8765/
```

All `.html` files can also be opened directly in a browser — no server needed.

There is no build step, bundler, linter, or test framework. Testing is manual in-browser.

## Architecture

The project is four standalone HTML files, each self-contained with inline CSS and JavaScript:

| File | Purpose | Capacity |
|------|---------|----------|
| `layout_7by6.html` | Fixed 7-column × 6-row seating shuffler | 42 seats |
| `layout_7by7.html` | Fixed 7-column × 7-row seating shuffler | 49 seats |
| `layout_custom.html` | Configurable grid (1–12 rows, 1–15 cols) | 180 seats |
| `group_roulette.html` | Group assignment roulette | 200 students, 30 groups |

**No framework.** Pure ES6+ JavaScript with direct DOM manipulation (`getElementById`, `classList`, `innerHTML`).

**External dependencies via CDN only:**
- [SheetJS (xlsx)](https://cdnjs.cloudflare.com/ajax/libs/xlsx/) — Excel export
- [jsPDF](https://cdnjs.cloudflare.com/ajax/libs/jspdf/) — PDF generation
- [html2canvas](https://cdnjs.cloudflare.com/ajax/libs/html2canvas/) — HTML-to-image for PDF pages

## Key Patterns

### Seating layouts (`layout_*.html`)

State is held in two parallel arrays:
- `seatActive[]` — boolean, whether a seat is enabled
- `seatNumbers[]` — student number currently assigned to that seat

The roulette animation runs `setInterval` at 150ms, calling `render()` repeatedly until stopped. Fisher-Yates shuffle (`shuffle(arr)`) produces the random permutation.

Excel export writes two perspectives to one sheet: student-facing view on top, teacher-facing view (180° rotation) below. File naming: `zaseki_YYYYMMDDhhmm.xlsx` (custom: `zaseki_custom_RxC_...`).

### Group roulette (`group_roulette.html`)

Two-page UI driven by visibility toggling: a setup page and a results page. Absent students are tracked in a `Set<number>`.

Results have three tab views:
1. Student view — student number → group
2. Group view — group headers with member columns
3. Circle view — SVG polygon visualizations of groups

PDF export is 3 landscape A4 pages (one per view), rendered via html2canvas + jsPDF with JPEG quality 0.97.

## Important Constraints

- The UI language is Japanese. Keep all user-facing strings in Japanese.
- The dark background theme (CSS variable `--bg`) is intentional for classroom projection.
- Viewport-relative units (`vw`, `vh`) are used throughout for projection scaling — avoid pixel-based overrides.
- Files prefixed with `_` (`_dev_server.py`, `_fix_*.py`, etc.) are developer utilities and are git-ignored. Do not commit them.
- The `movs/` directory is also git-ignored.
