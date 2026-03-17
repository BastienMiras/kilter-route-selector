# IaC Podman Design — Kilter Route Selector

**Date:** 2026-03-11
**Status:** Approved

## Context

Infrastructure is treated as code. All environment configuration, container definitions, and deployment procedures are versioned in the repository. No manual server state outside of what Ansible provisions.

This design supersedes the Kamal v2 deployment strategy from `2026-03-01-cicd-design.md`. The CI/CD pipeline (GitHub Actions) and git flow remain unchanged.

---

## Constraints & Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Orchestration | podman-compose | Declarative, no daemon, rootless-friendly |
| VPS provisioning | Ansible | Idempotent, declarative, no state file |
| Environments | dev-local + dev-vps + staging + prod | All on same VPS except dev-local |
| Secrets | GitHub Actions Secrets + `.env` on VPS | Simple, sufficient for solo/small team |
| Sprint structure | Sprint 0 dedicated to infra | Infra-first before any app code |

---

## Multi-Stack VPS Layout

Three stacks run on a single VPS, isolated by Podman networks and directories:

```
VPS
├── /opt/kilter-prod/          :80 / :443  → backend internal :8000
├── /opt/kilter-staging/       :8081       → backend internal :8001
└── /opt/kilter-dev/           :8082       → backend internal :8002
```

Each stack has:
- Its own Podman network (`kilter-prod-net`, `kilter-staging-net`, `kilter-dev-net`)
- Its own data directory (`data/kilter.db`)
- Its own `.env` file

---

## Repository Structure

```
infra/
├── ansible/
│   ├── inventory/
│   │   ├── staging.yml              # VPS staging host
│   │   └── prod.yml                 # VPS prod host (same IP, different vars)
│   ├── group_vars/
│   │   ├── all.yml                  # common vars (deploy_user, app_base_dir)
│   │   ├── dev.yml                  # port 8082, image tag from branch
│   │   ├── staging.yml              # port 8081, image tag develop-<SHA>
│   │   └── prod.yml                 # port 80/443, image tag v*.*.*
│   ├── playbook-provision.yml       # one-time VPS bootstrap
│   ├── playbook-deploy.yml          # deploy (pull image + compose up)
│   └── roles/
│       ├── podman/                  # install Podman + podman-compose
│       ├── firewall/                # ufw rules
│       └── app/                     # create dirs, place .env, run compose
│
├── Containerfile.backend            # FastAPI multi-stage build
├── Containerfile.frontend           # React + Nginx multi-stage build
│
├── compose.dev.yml                  # local dev (volumes mounted, hot-reload)
├── compose.vps-dev.yml              # dev on VPS (built images, port 8082)
├── compose.staging.yml              # staging (GHCR develop-<SHA>, port 8081)
├── compose.prod.yml                 # prod (GHCR v*.*.*, port 80/443)
│
├── .env.example                     # committed template with all keys, empty values
└── Makefile                         # developer shortcuts

.github/
└── workflows/
    ├── ci.yml                       # PR gate: lint + tests + dry-run build
    ├── deploy-staging.yml           # push to develop → auto deploy staging
    └── deploy-prod.yml              # tag v*.*.* → manual approval → deploy prod
```

---

## Container Architecture

```
Internet
    │
    ▼ :80/:443 (prod) | :8081 (staging) | :8082 (dev)
┌─────────────────────┐
│   frontend          │  nginx:alpine — serves React build + proxies /api/*
└────────┬────────────┘
         │ internal network
         ▼ :8000 (prod) | :8001 (staging) | :8002 (dev)
┌─────────────────────┐
│   backend           │  python:3.12-slim — FastAPI + uvicorn
└────────┬────────────┘
         │ volume
         ▼
    data/kilter.db
```

### Containerfile.backend (multi-stage)

- **Stage `builder`**: `python:3.12` — installs pip deps
- **Stage `runtime`**: `python:3.12-slim` — copies venv + app code (~150MB)
- Entrypoint: `uvicorn main:app --host 0.0.0.0 --port 8000`

### Containerfile.frontend (multi-stage)

- **Stage `builder`**: `node:20-alpine` — `npm ci && npm run build`
- **Stage `runtime`**: `nginx:alpine` — serves `/dist`, proxies `/api/*` to backend

### Compose variants

| File | Images | Backend | Ports | Restart |
|---|---|---|---|---|
| `compose.dev.yml` | local build | volume + `--reload` | 8000 + 5173 | no |
| `compose.vps-dev.yml` | built images | built | 8082 | unless-stopped |
| `compose.staging.yml` | GHCR `develop-<SHA>` | built | 8081 | unless-stopped |
| `compose.prod.yml` | GHCR `v*.*.*` | built | 80/443 | always |

---

## CI/CD Pipeline

```
feature/* ──PR──▶ ci.yml
                    │ ruff lint (backend)
                    │ eslint (frontend)
                    │ pytest
                    │ vitest
                    │ podman build --dry-run
                    ✅ required gate before merge

develop ────push──▶ deploy-staging.yml
                    │ build backend + frontend images
                    │ push to GHCR: develop-<SHA> + develop-latest
                    │ ansible-playbook deploy.yml -e env=staging
                    │ health check GET /health → 200
                    ✅ automatic staging deploy

tag v*.*.* ─push──▶ deploy-prod.yml
                    │ [GitHub Environments: manual approval]
                    │ re-tag staging image → v*.*.* + latest
                    │ ansible-playbook deploy.yml -e env=prod
                    │ health check GET /health → 200
                    ✅ production deploy
```

### GitHub Actions Secrets

| Secret | Usage |
|---|---|
| `VPS_HOST` | IP of the VPS |
| `VPS_SSH_KEY` | Private SSH key for Ansible |
| `GHCR_TOKEN` | Push/pull images on GHCR |
| `ENV_STAGING` | Content of `/opt/kilter-staging/.env` |
| `ENV_PROD` | Content of `/opt/kilter-prod/.env` |

---

## Ansible Roles

### `podman` role
- Installs `podman` and `podman-compose` on Ubuntu/Debian
- Configures rootless Podman for the deploy user

### `firewall` role
- Configures ufw:
  - 22 → SSH
  - 80 → prod HTTP
  - 443 → prod HTTPS
  - 8081 → staging (open or IP-restricted)
  - 8082 → dev (open or IP-restricted)

### `app` role
- Creates `/opt/kilter-{env}/` directory structure
- Places `.env` file (from Ansible vault or GitHub Actions secret)
- Pulls images from GHCR
- Runs `podman-compose -f compose.{env}.yml up -d`
- Waits for health check `/health` to return 200

### Playbooks

**`playbook-provision.yml`** — Run once to bootstrap a fresh VPS:
1. Create deploy user
2. Install Podman (role: podman)
3. Configure firewall (role: firewall)
4. Create app directories
5. Place `.env` files

**`playbook-deploy.yml`** — Run on every deployment:
1. Pull latest images from GHCR
2. Run `podman-compose up -d --pull always`
3. Health check

---

## Secrets Management

```
Committed to repo:
  .env.example          ← all required keys, empty values, safe to commit

GitHub Actions Secrets:
  ENV_STAGING           ← full .env content for staging
  ENV_PROD              ← full .env content for prod
  VPS_HOST, VPS_SSH_KEY ← Ansible connection

On VPS (placed by Ansible):
  /opt/kilter-prod/.env
  /opt/kilter-staging/.env
  /opt/kilter-dev/.env  ← placed manually or via playbook-provision.yml
```

---

## Makefile (developer shortcuts)

```makefile
up:              ## Start local dev stack
	podman-compose -f compose.dev.yml up

down:            ## Stop local dev stack
	podman-compose -f compose.dev.yml down

build:           ## Build images locally
	podman build -f Containerfile.backend -t kilter-backend:local .
	podman build -f Containerfile.frontend -t kilter-frontend:local .

deploy-dev:      ## Deploy to dev env on VPS (manual)
	ansible-playbook infra/ansible/playbook-deploy.yml -i infra/ansible/inventory/staging.yml -e env=dev

provision:       ## Bootstrap a fresh VPS (run once)
	ansible-playbook infra/ansible/playbook-provision.yml -i infra/ansible/inventory/staging.yml

test:            ## Run all tests
	pytest backend/tests/ -v
	npm run test --prefix frontend
```

---

## Sprint 0 Deliverables

| # | Task | Output |
|---|---|---|
| 0.1 | Containerfile.backend | Working multi-stage image for FastAPI skeleton |
| 0.2 | Containerfile.frontend | Working multi-stage image for Nginx + React placeholder |
| 0.3 | compose.dev.yml | Local dev stack running with `make up` |
| 0.4 | compose.vps-dev.yml + compose.staging.yml + compose.prod.yml | Multi-env compose files |
| 0.5 | Ansible role: podman | Idempotent Podman install on Ubuntu/Debian |
| 0.6 | Ansible role: firewall | ufw configured for all 5 ports |
| 0.7 | Ansible role: app | Deploy + health check |
| 0.8 | playbook-provision.yml | VPS bootstrapped from scratch in 1 command |
| 0.9 | playbook-deploy.yml | Deploy any env in 1 command |
| 0.10 | .env.example | All required env vars documented |
| 0.11 | Makefile | `make up`, `make deploy-dev`, `make provision`, `make test` |
| 0.12 | ci.yml | PR gate: lint + tests + build check |
| 0.13 | deploy-staging.yml | Auto-deploy staging on push to develop |
| 0.14 | deploy-prod.yml | Manual-approval prod deploy on tag |
| 0.15 | Update TASKS-INDEX.md | Sprint 0 added before Sprint 1 |
| 0.16 | sprints/sprint-0/README.md | Sprint overview + task table |

---

## Relation to Existing Design Docs

- **`2026-03-01-cicd-design.md`**: Git flow and GitHub Actions structure are preserved. Kamal v2 is replaced by Ansible + podman-compose.
- **`2026-03-07-web-ui-design.md`**: React + Nginx + Podman direction is unchanged. Containerfile.frontend and compose files formalize what was sketched there.
