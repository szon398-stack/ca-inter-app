# CA Inter · Remaining Lectures Planner — 3D Web App

The original single-file HTML planner (`ca-inter-schedule.html`) was upgraded into a
production-style Node.js web application. **Every existing function/tool is preserved** —
timers, locks, progress tracking, night verify, reminders, Sunday overflow, print sheet,
pomodoro, syllabus guide — the UI is layered with futuristic 3D/FX on top.

## Stack

| Layer    | Tech |
|----------|------|
| Backend  | Node.js, Express (REST API) |
| Realtime | WebSocket (`ws`) live state sync across tabs/devices |
| Storage  | MongoDB (Mongoose) with automatic JSON-file fallback |
| 3D      | Three.js (GPU starfield + 3D task-visualization card) |
| FX      | GSAP (entrance/scroll animations) + glowing cursor |
| Charts  | ApexCharts (animated) + hand-rolled SVG donuts/gauges |
| Frontend | Vanilla JS modules (same logic, modularised files) |

## Quick start

```bash
cd ca-inter-app
npm install        # installs deps + copies vendor libs (postinstall)
npm start          # http://localhost:3000
```

Open `http://localhost:3000` in Chrome/Edge.

### MongoDB (optional, recommended)
Install MongoDB locally or set a connection string:

```bash
# data keeps its own MongoDB if reachable; otherwise it silently uses ./data/*.json
copy .env.example .env   # then edit MONGO_URI if needed
set MONGO_URI=mongodb://127.0.0.1:27017/ca_inter
npm start
```

The app **never breaks** without MongoDB — it falls back to a file adapter
(`data/*.json`), prints a warning, and keeps syncing via WebSocket.

## Tests

```bash
# start server first, then:
npm test          # jsdom smoke test → 23 assertions (boot, rendering, tick, live sync)
```

## Architecture

```
ca-inter-app/
├─ server/
│  ├─ index.js            Express + static + WS + graceful db init
│  ├─ config.js           port, mongo uri, allowed storage keys
│  ├─ db/                 file.js (fallback) · mongo.js · index.js (adapter pick)
│  └─ routes/             api.js (REST) · analytics-shared.js (pace intelligence)
│  └─ ws.js               realtime broadcast hub
├─ public/
│  ├─ index.html          generated from the original (all DOM ids preserved)
│  ├─ css/style.css       original design + 3D/FX layer
│  ├─ vendor/             three.min.js, gsap.min.js (local → works offline)
│  └─ js/                 modular client: data · store · planner · progress ·
│                         svg · render-core · live · journey · log · reminders ·
│                         night · addons · main + fx/* (three-bg, task-3d,
│                         glow-cursor, animate)
└─ scripts/               copy-vendor.js · smoke-test.js
```

### Data flow (live sync)

```
[tick task / log task / move clock] → storeSet(key) → localStorage (instant)
                                       │
                                       └─ PUT /api/state/:key (debounced)
                                            │
                                            └─ server db.set → broadcast WS
                                                 │
                                                 └─ other tab: WS state:update
                                                      → localStorage → autoRefresh repaints
```

Newest-wins merge on timestamp; offline-safe (local always works, sync resumes).

### "Intelligent self-adjust"
- `/api/analytics` derives pace %, streak and a guidance tip from the daily log.
- The 3D core orb + task pillars re-colour by pace status (best…poor).
- Real-time progress: every change re-renders stats, graphs, water tank, speed line,
  week chart and the 3D visualisations instantly.

## Notes
- `public/index.html` was assembled from the original file; to regenerate after
  editing the source layout, see `index.html` (it is committed as static).
- ApexCharts loads from CDN; everything else is served locally. The app's core
  logic works fully offline — 3D/FX degrade gracefully if a library is missing.