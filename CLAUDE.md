# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

This repository is in **planning phase** — no code exists yet. All content is documentation and task specs.

**First code to be written:** Sprint 0 infrastructure (Containerfiles, Ansible, GitHub Actions).
**First app code:** Sprint 1 FastAPI backend.

## Project Overview

**Kilter Route Selector** aggregates climbing routes from the Kilter Board community via BoardLib, calculates movement metrics, and generates personalized training sessions. It serves a React web client via Nginx.

## Architecture

```
boardlib sync (cron, weekly)
    ↓
kilter.db (SQLite)
    ↓
FastAPI + aiosqlite (metrics: climb_metrics table)
    ↓
Nginx (reverse proxy /api/* → backend)
    ↓
React + TypeScript + Vite (SPA — responsive web, mobile-friendly)
```

### Infrastructure Stack (Sprint 0)
- **Podman** + **podman-compose** — rootless containers
- **Ansible** — idempotent VPS provisioning (roles: podman, firewall, app)
- **GitHub Actions** — CI/CD (lint + test + build gate, auto-deploy staging, manual prod)
- **GHCR** — container image registry
- Single VPS hosts 3 environments: dev (:8082), staging (:8081), prod (:80)

### Backend Stack (Sprint 1-2)
- **Python 3.12**, FastAPI, aiosqlite
- **SQLite** via BoardLib (`boardlib database kilter data/kilter.db`)
- **BoardLib** for data (no custom scraper needed — 200k+ routes)
- **pytest + pytest-asyncio** for tests
- Swagger docs at `/docs`, health check at `/health`

### Client Stack (Sprint 3-4)
- **React 18 + TypeScript + Vite** — SPA, responsive (mobile + desktop)
- **TanStack Query** — server state / caching
- **React Router** — navigation
- **Recharts** — charts
- **Nginx** — static serving + `/api/*` reverse proxy

## Task Navigation

Tasks are tracked in `TASKS-INDEX.md` (120 tasks, ~256h). Detailed specs are in `sprints/sprint-N/`.

Sprint milestones:
- **Sprint 0**: IaC — Containerfiles, podman-compose, Ansible, GitHub Actions (17 tasks)
- **Sprint 1**: FastAPI + SQLite/BoardLib + metrics (28 tasks)
- **Sprint 2**: Session builder algorithm + `/api/sessions/generate` (17 tasks)
- **Sprint 3**: React MVP — Home, Session, Detail screens + frontend containers (19 tasks)
- **Sprint 4**: Heatmap canvas, charts, zone map (15 tasks)
- **Sprint 5+**: Auth, ML recommendations, community stats (24 tasks)

## Key Algorithms

### Layout Parsing
Kilter layout format: `p1083r15p1117r15...`
```python
position = int(pos_str)
x = position & 0xff           # low 8 bits
y = (position & 0xff00) >> 8  # high 8 bits
role_code = int(role_str)     # NOT radius — LED type/role
# role codes: 12=start, 13=foot-only, 14=hand+foot, 15=finish
```

### Metrics Calculated per Climb
- `avg_distance` — mean Euclidean distance between successive holds
- `max_reach` — max distance between consecutive holds (dyno detection)
- `move_count` — `len(holds) - 1`
- `vertical_range` — `max(y) - min(y)`
- `symmetry_score` — left/right hold ratio
- `hold_density` — holds per surface unit

### Style Scoring (normalized 0-1)
```python
score_dynamic    = avg_distance * 2 + max_reach * 3 - density * 1
score_technical  = density * 3 - avg_distance * 1 + move_count * 2
score_endurance  = move_count * 3 + vertical_range * 2
```

### Session Builder
Selects `count` routes using style score + popularity bonus, enforcing zone and setter diversity. See `BRAINSTORM.md` for the full algorithm.

## Data Schema

```sql
-- From BoardLib (read-only):
-- climbs: uuid, name, setter_username, difficulty (int), frames (layout string),
--         ascensionist_count, quality_average, is_listed, angle

-- To be created:
-- climb_metrics: climb_uuid (FK), move_count, avg_distance, max_reach,
--                vertical_range, horizontal_range, symmetry_score, hold_density,
--                style_dynamic_score, style_technical_score, style_endurance_score,
--                computed_at
```

## Planned Directory Structure

```
backend/
├── main.py
├── requirements.txt
├── api/routes/
├── services/            # metrics calculator, session builder, boardlib sync
├── models/              # SQLAlchemy models
└── tests/

frontend/
├── index.html           # Sprint 0: placeholder; Sprint 3: Vite entry
├── src/                 # Sprint 3+
└── package.json         # Sprint 3+

infra/
├── Containerfile.backend
├── Containerfile.frontend
├── compose.dev.yml
├── compose.vps-dev.yml
├── compose.staging.yml
├── compose.prod.yml
├── nginx.conf
├── .env.example
├── ansible/
│   ├── playbook-provision.yml
│   ├── playbook-deploy.yml
│   ├── inventory/
│   ├── group_vars/
│   └── roles/ (podman, firewall, app)
└── (Makefile at repo root)

.github/workflows/
├── ci.yml
├── deploy-staging.yml
└── deploy-prod.yml
```

## Development Commands

```bash
# Local dev stack (once Sprint 0 is done)
make up              # start backend :8000 + frontend :5173
make down            # stop stack
make test            # pytest backend/tests/
make build           # build images locally

# VPS operations
make provision       # bootstrap fresh VPS (run once)
make deploy-dev      # deploy dev env on VPS

# Backend (manual)
python -m venv venv && source venv/bin/activate
pip install -r backend/requirements.txt
uvicorn main:app --reload --port 8000

# Tests
pytest backend/tests/ -v
pytest backend/tests/test_specific.py::test_name
```

## Key Design Decisions (superseded plans)

- `docs/plans/2026-03-01-cicd-design.md` — CI/CD design (git flow preserved; **Kamal v2 replaced by Ansible + podman-compose**)
- `docs/plans/2026-03-07-web-ui-design.md` — React + Nginx + Podman (still current)
- `docs/plans/2026-03-11-iac-podman-design.md` — IaC design (current, authoritative)
- `ARCHITECTURE-REVIEW.md` — critical findings (BoardLib, SQLite, role codes, grade_numeric)
