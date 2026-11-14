# ARES – Advanced Rover Exploration System

A Mars mission-control prototype: 3D globe, terrain inspection, weather observations, route comparison, a discovery catalogue and a grounded AI assistant.

**Stack:** React + Vite + TypeScript + Tailwind + CesiumJS · Node + Express + MongoDB (Mongoose) · REST

## Run locally

Requires Node 20+.

```bash
npm install
cp server/.env.example server/.env   # optional: add NASA_API_KEY / ANTHROPIC_API_KEY
npm run dev                          # API on :4000, app on http://localhost:5173
```

- **MongoDB is optional.** If it isn't reachable, ARES uses an in-memory store (saved routes vanish on restart). `/api/health` and the UI report which one is active. To use MongoDB: `docker run -d -p 27017:27017 mongo` (default URI `mongodb://127.0.0.1:27017/ares`).
- **NASA_API_KEY**: get a free key at https://api.nasa.gov. `DEMO_KEY` works but is heavily rate-limited.
- **ANTHROPIC_API_KEY** (optional): turns the assistant LLM-backed (`ARES_AI_MODEL`, default `claude-sonnet-5-5`). Without it the assistant uses a built-in responder grounded in the same data.

Other scripts: `npm run typecheck`, `npm run build` (client bundle in `client/dist`).

## Pages

Landing · Mission Dashboard · Mars Explorer (3D) · Terrain Analysis · Environment · Route Planner · Discovery Explorer · AI Assistant · About / Data Sources

## Data honesty (read this)

Every value is tagged **LIVE**, **HISTORICAL**, **REFERENCE** or **SIMULATED** in the UI.

| Feature | What it really uses |
|---|---|
| Globe imagery | NASA Trek tiles (Viking MDIM 2.1, MOLA shaded) streamed by the browser. If tiles fail, the map says so and shows a plain globe. |
| Locations & discoveries | **Reference** catalogue curated from published mission results (see `server/src/data/reference.ts`). Elevations are approximate. |
| Weather | Tries the NASA InSight weather API → shown as **HISTORICAL** (mission ended 2022). If unreachable → clearly **SIMULATED** demo values. No live Mars weather or dust-storm feed is integrated. |
| Rover photos | NASA Mars Rover Photos API (live) or an error state. |
| Terrain (elevation/slope/hazard) | **SIMULATED.** A synthetic model anchored on approximate feature elevations; *not* MOLA/HiRISE measurements. |
| Routes | A* over that terrain model with three cost profiles. Regional only (≤ 600 km). Planning aid, **not safety-certified**. |

**Using real terrain:** implement the `TerrainProvider` interface in `server/src/services/terrain.ts` against a DEM (e.g. the USGS MOLA–HRSC blended DEM) and return it from `getTerrain()`. Everything downstream (inspector, grids, routes, assistant) picks it up.

## Structure

```
server/src
  index.ts            Express app + REST routes
  db.ts               Mongoose + in-memory fallback
  data/reference.ts   Curated locations, discoveries, data-source registry
  services/terrain.ts Terrain provider, slope/hazard, grid, profile
  services/router.ts  A* route planner (direct / balanced / cautious)
  services/nasa.ts    NASA API clients with provenance + attempt logs
  services/assistant.ts  Grounded assistant (LLM or offline)
client/src
  components/MarsMap.tsx  CesiumJS Mars globe (Mars ellipsoid)
  pages/              The nine pages
  lib/                API client, shared state, types
```

## API

`GET /api/health · /locations · /discoveries · /sources · /terrain/inspect · /terrain/grid · /terrain/profile · /environment · /rover-photos/:rover`
`POST /api/routes/plan · /assistant` · `GET/POST/DELETE /api/routes/saved`
# mar-app-challange-
