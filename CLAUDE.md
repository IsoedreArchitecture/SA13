# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SA13 Intranet is a single-file, zero-dependency HTML application built for the board of the **Syndicat des Architectes des Bouches-du-Rhône (SA13)** — a French architects' professional union. It runs entirely in the browser with no build step, no backend, and no persistence layer. The entire application lives in `SA13_intranet.html`.

## Running the App

Open `SA13_intranet.html` directly in a browser. There is no server, no build process, and no package manager. To serve it locally:

```bash
python3 -m http.server 8000
# then open http://localhost:8000/SA13_intranet.html
```

## Architecture

### Single-file SPA

The file is structured as: `<head>` CSS → `<body>` HTML → `<script>` JS, all inline.

**Navigation** is tab-based via `showPage(id)`, which toggles `.active` on `.page` divs and `.nav-item` elements. Pages: `dashboard`, `seance`, `partenariats`, `actions`, `annuaire`, `planning`.

### Data Layer

All data is hardcoded as `const` arrays at the top of the `<script>` block — there is no API or database:

| Constant | Purpose |
|---|---|
| `MEMBRES` | Board members (name, initials, role) |
| `PARTENARIATS` | Partnership pipeline entries |
| `ACTIONS` | Action items / decisions from board meetings |
| `AGENDA` | Upcoming events |
| `PLANNING_COMM` | Communication planning calendar |

**To add or modify data**, edit the relevant `const` array directly.

### State Management

In-memory JS variables hold runtime state:

- `presenceState` — object mapping member name → `'neutre' | 'present' | 'pouvoir' | 'absent'`
- `voteState` — `{pour, contre, abstention}` counts for the current vote
- `seanceActions` — array of decisions/votes recorded during the current session

**Nothing persists** across page reloads. There is no `localStorage` or cookie usage.

### Render Functions

Each page has a dedicated render function called on navigation:

- `renderDashboard()` — computes stats and fills action/agenda/pipeline previews
- `renderPartenariats()` — builds pipeline kanban + stats cards + table
- `renderActions()` — filtered table of ACTIONS with overdue highlighting
- `renderAnnuaire()` — grid of contact cards from MEMBRES
- `renderPlanning()` — timeline + communication table

### Design System

CSS custom properties (defined on `:root`) drive the full theme. Key tokens:

- Colors: `--orange` (#E8580A), `--navy` (#1A2B4A), `--green`, `--amber`, `--red`, `--grey`
- Semantic surfaces: `--bg`, `--surface`, `--text`, `--text-muted`, `--grey-border`
- Shapes: `--radius` (8px), `--radius-lg` (12px)

Reusable component classes: `.card`, `.stat-card` (with `.stat-ok/.stat-warn/.stat-danger` variants), `.badge` (with `-orange/-green/-amber/-red/-navy/-grey`), `.btn` / `.btn-primary`, `.pipeline`, `.timeline`, `.presence-item`, `.vote-btn`.

External CDN dependencies (require internet access to load):
- `fonts.googleapis.com` — Inter font family
- `cdn.jsdelivr.net/@tabler/icons-webfont` — icon font (`ti ti-*` classes)

### Business Logic

- **Quorum**: Requires `Math.ceil(15 / 2) + 1 = 9` voting members (présents + pouvoirs). The board has 15 voting seats; Anaïs is listed as non-voting.
- **Presence cycle**: clicking a member's tile cycles `neutre → present → pouvoir → absent → neutre`.
- **Vote recording**: `saveVote()` appends a formatted string to `seanceActions` and resets counters.
- **Late actions**: any action with `statut !== 'Fait'` and `echeance` date in the past is flagged in red.

## Conventions

- The application is entirely in **French** — all UI text, variable names for user-facing data, and new content should be in French.
- Keep everything in the single HTML file. Do not split into multiple files or introduce a bundler.
- Do not introduce JS frameworks or npm dependencies — vanilla JS only.
- When adding a new page/section, follow the pattern: add a `.page` div in HTML, a `showPage` case in the `titles` map, a nav item in the sidebar, and a `render*()` function called from `showPage()`.
