# IT Program Resource Planning Dashboard

A single-file, browser-based resource planning tool designed for managing IT program portfolios — cloud migrations, infrastructure projects, and software delivery programs. Fully brandable, it runs entirely in the browser with no server, framework, or build step required.

## Quick Start

1. Open `resource_planner.html` in a Chromium-based browser (Chrome or Edge recommended)
2. Create a portfolio or load an existing JSON data file
3. Define your roles, projects, and phases
4. Plan resource allocation across your program timeline

> **Note:** The File System Access API (used for direct JSON file saving) requires Chrome or Edge. Firefox and Safari can still use the tool but will fall back to browser-only storage with manual download/upload.

## Features

### Portfolio Management
- Create, rename, switch between, and delete multiple portfolios
- Each portfolio maintains its own set of projects, roles, resources, settings, and financial data
- Data is stored in browser `localStorage` with optional JSON file linking for persistent storage
- **Multi-session diff**: compare portfolio changes across saved sessions

### JSON File Linking
- Link a portfolio to a `.json` file on disk using the File System Access API
- Changes auto-save directly to the linked file (no manual export needed)
- File handles persist across page refreshes via IndexedDB
- Amber warning indicator when no file is linked — click to link one
- **Smart save debouncing**: UI renders after 600ms of inactivity, data persists after 5s, with a 60s auto-save interval — keeps typing responsive
- Manual import/export available as fallback

### Dashboard Tabs

#### Overview
- Portfolio-wide summary with KPI cards: total projects, active roles, current FTE demand, staffing gaps
- Week-by-week demand vs supply chart
- Role-level gap analysis highlighting shortfalls
- Filters by contract status (committed demand vs all)

#### Gantt Chart
- Visual timeline of all projects with colour-coded phase bars
- Phases rendered proportionally across the program timeline
- Phase offsets supported for non-sequential scheduling (overlapping, parallel, or gapped phases)

#### Weekly Resource Demand
- Heatmap grid showing FTE demand per role per week
- Colour intensity indicates demand level
- Totals per role and per week

#### Monthly Resource Demand
- Aggregated monthly view of the weekly demand data
- Useful for longer-range planning and reporting

#### Recruitment & Onboarding Planner
- Shows hiring needs based on gaps between demand (from committed projects) and current supply
- Configurable onboarding lead time per role
- Separate "Potential Roles" section for Pipeline and Prospect projects (not counted for active recruitment)
- Recruitment timeline aligned to project start dates
- **First Needed popover**: click to see detailed breakdown of which project and phase drives each role's earliest demand date

#### Resource Management
- Add and manage named resources with role, start date, end date, state, salary, and project assignment
- **Multi-role assignments**: resources can be assigned to multiple roles across different projects (or multiple roles on the same project)
- **Time-aware over-allocation detection**: warns only when assigned roles actually overlap in time, respecting shared-phase logic
- **No-planned-effort warnings**: highlights when a role is assigned to a project that has no FTE allocation for that role
- Assignment blocked for non-contracted projects (shows warning)
- Per-resource cost tracking with state-specific burden rates
- Supports full-time and part-time allocations

#### Financials
- Per-project revenue and cost breakdown
- Toggle between simple view and Contracted vs Forecast split
  - **Contracted**: projects with signed contracts (100% confidence)
  - **Forecast**: High Confidence, Pipeline, and Prospect projects shown separately
- KPI summary cards for revenue, cost, and margin
- Per-project detail table with phase-level breakdown
- **Role salary estimates**: configurable min/max salary range per role (set in Configuration), used for cost projections on unfilled positions
- **Estimate info popovers**: click to see the full calculation breakdown (salary range, burden rate, hourly rate) when estimates are used

### Project Configuration

#### Contract Status
Each project has a contract status that controls planning behaviour:

| Status | Probability | Can Assign Resources | Counted for Recruitment |
|--------|------------|---------------------|------------------------|
| **Contracted** | 100% | Yes | Yes |
| **High Confidence** | 90% (editable) | No | Yes |
| **Pipeline** | 50% (editable) | No | No |
| **Prospect** | 20% (editable) | No | No |

- FTE allocations are always editable regardless of status
- Contract status and probability are set per project in the configuration panel

#### Project Phases
- Default phases: Mobilisation, Discovery & Assessment, Execution, Hypercare (customisable)
- Each phase has a configurable duration (weeks) and offset from the previous phase
- **Phase offset** controls timing relative to the prior phase:
  - Offset = 0: starts immediately after the previous phase ends
  - Negative offset: overlaps with the previous phase
  - Positive offset: adds a gap between phases
- Per-phase activity lists (editable bullet points) for tracking tasks like "Project kick-off meeting" or "Team introductions & onboarding"
- Activities are stored per-project and persist to the JSON file

#### FTE Allocation
- Per-project, per-role, per-phase FTE values
- Automatically distributed across the phase's weeks in the timeline

### Branding & Customisation
- **Company name** and **logo URL** stored in JSON — no hardcoded branding in the HTML
- **Primary colour** and **accent colour** pickers override the default theme via CSS custom properties
- Default phases (names, durations, activities) are stored in `settings.defaultPhases` in the JSON, not as constants in the code
- New portfolios start with a clean slate — no sample projects or roles
- All configuration is portable: copy the JSON file to rebrand for a different organisation

### Program Settings
- **Program start date**: anchor for the entire timeline (defaults to start of current Australian financial year quarter)
- **Total weeks**: length of the planning horizon
- **Australian Financial Year**: Jul–Jun quarters used for date calculations
- **Public holidays**: fetched from nager.at API by Australian state
- **Burden rate components**: superannuation, payroll tax, WorkCover, leave loading — configurable per state
- **Role salary estimates**: min/max salary range per role used for financial projections on unfilled positions

### Print / Reports
- Each tab has a print button that generates a print-friendly view
- **Always prints in light mode** — dark theme is automatically overridden for print output
- Uses `@media print` CSS — no external dependencies
- Optimised page breaks prevent content from being cut across pages
- Header with portfolio name, report title, and generation date
- Footer with confidentiality notice
- Financials auto-switches to the full Contracted vs Forecast view when printing
- Use your browser's "Save as PDF" to create PDF reports

### Onboarding Guide
- First-time setup wizard guides new users through 3 steps: Configure Projects, Set Financial Data, Add & Assign Resources
- Auto-detects completion state and highlights remaining steps
- Dismissible permanently via "Don't show again" option (stored in localStorage)

### Responsive Design
- Adaptive layout with breakpoints at 80rem, 64rem, and 48rem
- Mobile-friendly with wrapped navigation bars and scaled-down controls
- Horizontal scrolling on data-heavy tables to prevent content clipping

### Theme
- Light and dark mode toggle
- Follows Atturra brand palette (teal, orange, gold)
- Theme preference persists across sessions

### Keyboard Shortcuts
- `Ctrl + S` — Save / export data
- `Ctrl + P` — Print current tab report
- Number keys `1–7` — Switch between tabs

## Data Structure

All portfolio data is stored as JSON with the following top-level structure:

```json
{
  "settings": {
    "startDate": "2025-07-07",
    "totalWeeks": 52,
    "portfolioName": "My Portfolio",
    "branding": { "companyName": "Acme Corp", "logoUrl": "https://...", "primaryColor": "#004C45", "accentColor": "#BE5400" },
    "defaultPhases": [ { "name": "Mobilisation", "short": "MOB", "weeks": 2, "activities": [...] }, ... ],
    "burdenConfig": { ... },
    "holidayStates": ["AU-NSW"]
  },
  "roles": [ ... ],
  "projects": { ... },
  "resources": [ ... ],
  "projectFinancials": { ... },
  "roleSalaryEstimates": { "Solution Architect": { "minSalary": 140000, "maxSalary": 220000 }, ... }
}
```

Each project contains:
- `contractStatus` — one of `contracted`, `highConfidence`, `pipeline`, `prospect`
- `contractProbability` — win percentage (informational, does not multiply revenue)
- `phases` — object with phase names as keys, each having `weeks`, `offset`, and `activities`
- `fte` — nested object of role/phase FTE allocations
- `revenue`, `startDate`, and other metadata

## Browser Compatibility

| Feature | Chrome/Edge | Firefox | Safari |
|---------|------------|---------|--------|
| Core functionality | Full | Full | Full |
| File System Access API (auto-save to disk) | Yes | No | No |
| IndexedDB handle persistence | Yes | N/A | N/A |
| Print reports | Full | Full | Full |

## Technology

- **Zero dependencies** — single HTML file with inline CSS and JavaScript
- **No build step** — open the file and it works
- **No server required** — runs entirely in the browser
- **~5,000 lines** of vanilla HTML/CSS/JS
- Uses Google Fonts (Poppins, JetBrains Mono) loaded via CDN

## File Structure

```
Resource Planner/
├── resource_planner.html    # The entire application
├── README.md                # This file
└── .gitignore               # Excludes *.json data files from version control
```

## Contributing

Found a bug or have a feature idea?

- **Bug reports**: [Open an issue](https://github.com/ChrisFryer/IT-Project-Resource-Planner/issues/new?labels=bug&template=bug_report.md)
- **Feature requests**: [Open an issue](https://github.com/ChrisFryer/IT-Project-Resource-Planner/issues/new?labels=enhancement&template=feature_request.md)
- **Pull requests** are welcome — please open an issue first to discuss the change

## License

This project is licensed under the [MIT License](LICENSE).
