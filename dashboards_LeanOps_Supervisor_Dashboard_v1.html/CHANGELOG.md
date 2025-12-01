# LeanOps MES — Changelog

A running history of major changes to the LeanOps Manufacturing Execution System
(front-end dashboards + cloud backend).

The most recent changes are listed first.

---

## [2025-12-01] - Supabase Write-Back (Stations + Jobs)

### Added
- **Cloud-backed job completion flow** in `dashboards_LeanOps_Supervisor_Dashboard_v1.html`:
  - `simulateCompleteJob(stationId)` now:
    - Marks the job record in `public.jobs` as `status = 'completed'`.
    - Updates `public.station_assignments.qty_done` to the job’s `qty_total`.
    - Sets the station record in `public.stations.status` to `idle`.
  - Keeps local in-memory state in sync so the UI updates immediately after the cloud write.

### Improved
- **Station status write-backs**:
  - `simulateBlocked(stationId)` now updates `public.stations.status = 'blocked'` in Supabase
    before updating local state.
  - `simulateIdle(stationId)` now updates `public.stations.status = 'idle'` in Supabase
    before updating local state.
- Error handling and logging:
  - Added console logging and user-friendly alerts when Supabase updates fail.
  - UI still updates local state carefully so the dashboard doesn’t feel “frozen”
    even if a cloud write has issues.

### Verified
- Confirmed that:
  - Clicking **“Mark blocked”** updates `public.stations.status` to `blocked`.
  - Clicking **“Set idle”** updates `public.stations.status` to `idle`.
  - Clicking **“Mark complete”**:
    - Sets the job to `completed` in `public.jobs`.
    - Updates the matching row in `public.station_assignments`.
    - Clears the station’s `currentJob` and sets status to `idle` in both
      Supabase and the dashboard view.

---

## [2025-11-30] - Supabase Integration & Cloud Data Load

### Added
- **Supabase project `leanops-mes`** created with NANO plan.
- **Initial Postgres schema v1** defined in `/database/schema_v1.md.txt` and applied via SQL:
  - Tables:
    - `public.stations` (id, name, line, status, is_active, timestamps).
    - `public.jobs` (id, qty_total, due_date, priority, status, timestamps).
    - `public.station_assignments` (station_id, job_id, qty_done, started_at, completed_at).
    - `public.performance_logs` (id, station_id, job_id, metric fields, timestamps).
- **Seed data** inserted for:
  - 4 stations (ST-01 … ST-04) across Line A / Line B.
  - Multiple jobs with different priorities (`normal`, `high`) and statuses
    (`waiting`, `running`, `completed`).
  - Matching station assignment rows to simulate a live floor.
- **Realtime enabled** in Supabase (project-level setting) to support future live updates.

### Front-end changes
- Wired the Supervisor Dashboard to **load data from Supabase** using `@supabase/supabase-js`:
  - Replaced hard-coded demo arrays (`stations`, `globalQueue`) with data loaded from:
    - `public.stations`
    - `public.jobs`
    - `public.station_assignments`
    - `public.performance_logs`
  - Build-up logic:
    - Joined `stations` with `station_assignments` and `jobs` in JavaScript to construct
      `currentJob` objects for each station.
    - Derived the **global queue** from jobs with `status = 'waiting'`.
- Implemented `loadInitialData()` to:
  - Call Supabase in parallel (`Promise.all`) for all four tables.
  - Map rows into the existing front-end data model.
  - Recalculate:
    - `metrics.completedToday`
    - `metrics.completedOnTimeRate` (placeholder demo value for now)
    - `metrics.stationsRunning` / `metrics.stationCount`.

### Verified
- Confirmed that:
  - Refreshing the dashboard pulls live data from Supabase (no more local-only seed arrays).
  - Updating table data in Supabase and clicking **Quick refresh** reflects changes
    in the Supervisor dashboard.

---

## [2025-11-29] - LeanOps MES Repo Setup & Dashboard Refinement

### Added
- **New GitHub repository** dedicated to the MES:
  - `leanops-mes` created to separate the factory/MES logic from the public marketing site (`leanops-site`).
- Committed the first version of:
  - `dashboards_LeanOps_Supervisor_Dashboard_v1.html`
    - Dark, premium LeanOps UI.
    - Stations panel + Global queue panel.
    - Top metrics bar (“Today” section).
    - View controls (compact / expanded, toggle details, light/dark, hide empty stations).

### Improved
- Refined the visual style:
  - Consistent typography with display + system fonts.
  - Clean “pill” buttons for filters and actions.
  - Better spacing, card shadows, and hover states to feel like a SaaS control center.
- Implemented **local-only simulation logic**:
  - `simulateCompleteJob`, `simulateBlocked`, `simulateIdle`,
    `simulateAssignFromQueue`, `simulateDropFromQueue`,
    `simulateNextHour`.
  - Metrics update locally:
    - `completedToday`
    - `completedOnTimeRate` (demo)
    - `stationsRunning` / `stationCount`.

### Verified
- Local HTML opened in browser behaves as a realistic supervisor view,
  even before cloud integration:
  - Can mark jobs complete.
  - Can move jobs between queue and stations.
  - Can simulate next hour and see metrics respond.

---

## [2025-11-26] - Initial Supervisor Dashboard Concept (Local Prototype)

### Added
- First **generic, non-employer-branded Supervisor dashboard** built in a single HTML file:
  - Designed for LeanOps as a reusable template for multiple factories.
  - UI sections:
    - Header with LeanOps logo and “Supervisor — Live Stations”.
    - “Today” metrics row (Jobs completed, On-time completion, Stations running).
    - Filters for Line / Status / Sort.
    - Stations grid with:
      - Station name + ID
      - Status badge (running / idle / blocked)
      - Current job + Next job blocks
      - Action buttons.
    - Global job queue with priority badges and assign/remove actions.
- Implemented **in-memory data model**:
  - `stations` array containing station-level state.
  - `globalQueue` array containing pending jobs.
  - Helper functions for sorting, filtering, and rendering DOM nodes.

### Purpose
- Served as the first concrete visual & functional prototype of the LeanOps MES supervisor layer.
- Gave a realistic “control room” experience before any backend or database existed.

---

## Notes for Future Entries

- Add a new section **on top** for each meaningful change:
  - Use date format: `[YYYY-MM-DD] - Short title`
  - Capture:
    - What you added (`### Added`)
    - What you improved or refactored (`### Improved`)
    - Anything you tested or confirmed (`### Verified`)
- When you later add:
  - Realtime subscriptions,
  - Worker Tablet / TV dashboards,
  - Role-based access,
  - Multi-plant support,

  log each as its own dated entry here.
