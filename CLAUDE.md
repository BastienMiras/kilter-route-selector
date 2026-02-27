# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

This repository is in **planning phase** — no code exists yet. All content is documentation and task specs. The first code to be written is the FastAPI backend (Sprint 1).

## Project Overview

**Kilter Route Selector** aggregates climbing routes from the Kilter Board community API, calculates movement metrics, and generates personalized training sessions. It scrapes the unofficial Kilter Board API (`api.kilterboardapp.com`), parses hold layouts, computes style scores, and serves a Flutter client.

## Planned Architecture

```
Kilter Board API (unofficial)
    ↓ async scraping worker (rate limited: 1 req/sec)
Backend: FastAPI + PostgreSQL + Redis
    ↓ REST API
Client: Flutter (Android + Desktop)
```

### Backend Stack (Sprint 1-2)
- **Python 3.11+**, FastAPI, SQLAlchemy or asyncpg
- **PostgreSQL 15+** with tables: `climbs`, `holds`, `climb_metrics`
- **Redis** for caching
- **pytest + pytest-asyncio** for tests
- Swagger docs at `/docs`, health check at `/health`

### Client Stack (Sprint 3-4)
- **Flutter 3.x** — cross-platform (Android, Windows, Linux, macOS)
- **Riverpod** — state management
- **Dio** — HTTP client
- **Hive** — local cache
- **go_router** — navigation
- **fl_chart** — charts

## Task Navigation

Tasks are tracked in `TASKS-INDEX.md` (103 tasks, ~240h). Detailed specs are in `sprints/sprint-N/TASK-X.Y-name.md`.

Sprint milestones:
- **Sprint 1**: FastAPI + PostgreSQL + scraping + metrics (28 tasks)
- **Sprint 2**: Session builder algorithm + `/api/sessions/generate` (17 tasks)
- **Sprint 3**: Flutter MVP — Home, Session, Detail screens (19 tasks)
- **Sprint 4**: Heatmap canvas, charts, zone map (15 tasks)
- **Sprint 5+**: Auth, ML recommendations, community stats (24 tasks)

## Key Algorithms

### Layout Parsing
Kilter layout format: `p1083r15p1117r15...`
```python
x = position & 0xff           # low 8 bits
y = (position & 0xff00) >> 8  # high 8 bits
```

### Metrics Calculated per Climb
- `avg_distance` — mean Euclidean distance between successive holds
- `max_reach` — max distance between consecutive holds (dyno detection)
- `move_count` — `len(holds) - 1`
- `vertical_range` — `max(y) - min(y)`
- `symmetry_score` — left/right hold ratio
- `hold_density` — holds per surface unit

### Style Scoring
```python
score_dynamic    = avg_distance * 2 + max_reach * 3 - density * 1
score_technical  = density * 3 - avg_distance * 1 + move_count * 2
score_endurance  = move_count * 3 + vertical_range * 2
```

### Session Builder
Selects `count` routes using style score + popularity bonus, enforcing zone and setter diversity. See `BRAINSTORM.md` for the full algorithm.

## Data Schema

```sql
-- climbs: kilter_id, name, setter, grade, angle, layout (raw string), ascents, quality_avg
-- holds: climb_id, position, x, y, radius, is_start, is_finish, is_foot_only
-- climb_metrics: climb_id (FK), all computed metrics above
```

## Kilter Board API (Unofficial)

```http
POST https://api.kilterboardapp.com/v1/logins
{"username": "...", "password": "...", "tou": "accepted", "pp": "accepted"}
# Returns: {"login": {"token": "Bearer eyJ..."}}

GET https://api.kilterboardapp.com/v1/climbs/{id}
Authorization: Bearer {token}
```

Rate limit: max 1 req/sec to avoid bans. Reference implementations: [BoardLib](https://github.com/lemeryfertitta/BoardLib), [blog post](https://bazun.me/blog/kiterboard).

## Planned Backend Directory Structure

```
backend/
├── main.py
├── config.py
├── requirements.txt
├── api/routes/          # FastAPI routers
├── services/            # Scraper, metrics calculator, session builder
├── models/              # SQLAlchemy models
└── tests/
```

## Development Commands (Once Code Exists)

```bash
# Backend
python -m venv venv && source venv/bin/activate
pip install -r backend/requirements.txt
uvicorn main:app --reload --port 8000

# Tests
pytest backend/tests/ -v
pytest backend/tests/test_specific.py::test_name  # single test

# Flutter client
flutter pub get
flutter run -d android   # or linux/windows
flutter test
```
