# CI/CD Architecture Design — Kilter Route Selector

**Date:** 2026-03-01
**Status:** Approved
**Stack:** GitHub Actions + GHCR + Kamal v2 + VPS self-hosted

---

## Context

Le projet Kilter Route Selector a une architecture hybride :
- Backend FastAPI + PostgreSQL (prod) / SQLite (CI) + Redis
- Client Flutter (Android + Desktop)
- Équipe : 2-5 développeurs
- Déploiement : VPS self-hosted (Hetzner/OVH)
- 4 environnements : dev-local → integration → staging → production

Ce document définit la stratégie CI/CD complète pour ce projet.

---

## Git Flow

```
feature/*
    │ PR + CI gate (lint + tests SQLite + flutter test)
    ▼
 develop ────────────────────────► INTEGRATION VPS
    │                               (auto, merge sur develop)
    │ branch release/v1.x quand develop est stable
    ▼
release/v1.x ───────────────────► STAGING VPS
    │  bugfixes ciblés sur release/* seulement
    │  hotfixes cherry-pick vers develop
    │
    │ merge + tag v1.x.0
    ▼
  main ───────────────────────────► PROD VPS      + GitHub Releases
                                    (approval gate)  (APK + desktop)
```

### Règles de branche (branch protection)

| Branche | Protection | Prérequis |
|---------|-----------|-----------|
| `feature/*` | Libre | — |
| `develop` | PR obligatoire | 1 review + CI vert |
| `release/v*` | Branchée depuis develop | 1 review par fix |
| `main` | PR depuis `release/*` uniquement | CI vert + staging healthy |

---

## Environnements

| Env | VPS spec | Déclencheur | URL exemple |
|-----|----------|-------------|-------------|
| dev-local | Machine développeur | `docker compose up` | http://localhost:8000 |
| integration | 2 vCPU / 4 GB | push `develop` | https://integration.kilter.example |
| staging | 4 vCPU / 8 GB | push `release/*` | https://staging.kilter.example |
| production | 4 vCPU / 8 GB | tag `v*.*.*` + approval | https://api.kilter.example |

---

## GitHub Actions — Les 4 Workflows

### 1. `ci.yml` — PR gate

**Déclencheur :** `pull_request` vers `develop` ou `release/*`

```yaml
jobs:
  lint-backend:   # ruff check . && mypy backend/
  lint-flutter:   # dart analyze && dart format --set-exit-if-changed .
  test-backend:   # pytest avec SQLite en mémoire + coverage ≥ 80%
  test-flutter:   # flutter test --coverage
  quality-gate:   # bloque si coverage < 80% ou lint error
```

### 2. `deploy-integration.yml` — Integration

**Déclencheur :** push sur `develop`

```yaml
jobs:
  test-integration:   # pytest + postgres:15-alpine service container
  build-push:         # docker buildx build → push GHCR (tag: develop-<SHA>)
  deploy:             # kamal deploy -d integration
  smoke-test:         # curl https://integration.kilter.example/health
```

### 3. `deploy-staging.yml` — Staging

**Déclencheur :** push sur `release/*`

```yaml
jobs:
  test-staging:   # pytest + postgres:15-alpine service container
  security:       # trivy image scan (CRITICAL bloque) + pip-audit
  build-push:     # push GHCR (tag: <branch-SHA> + latest-staging)
  deploy:         # kamal deploy -d staging
  smoke-test:     # battery de tests HTTP contre staging
  notify:         # Sentry release create (staging)
```

### 4. `release.yml` — Production + Releases

**Déclencheur :** push tag `v*.*.*`

```yaml
jobs:
  # Parallèles :
  test-final:     # full test suite + integration tests
  build-backend:  # push GHCR (tag: v1.2.3 + latest)
  build-flutter:
    strategy.matrix:
      - os: ubuntu-latest, target: android   → APK + AAB
      - os: ubuntu-latest, target: linux     → Linux AppImage
      - os: windows-latest, target: windows  → Windows installer

  # Séquentiels :
  github-release:    # crée la release + attache artefacts Flutter
  approval-gate:     # GitHub Environment "production" (review obligatoire)
  deploy-prod:       # kamal deploy
  post-deploy:       # health check + Sentry release prod + tag "stable" GHCR
```

---

## Docker & Registry

### Multi-stage Dockerfile (backend)

```dockerfile
# Stage 1 : builder
FROM python:3.11-slim AS builder
WORKDIR /app
COPY backend/requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# Stage 2 : runtime (~150 MB)
FROM python:3.11-slim AS runtime
WORKDIR /app
COPY --from=builder /install /usr/local
COPY backend/ .
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=5s \
  CMD curl -f http://localhost:8000/health || exit 1
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Politique de tags GHCR

| Tag | Quand |
|-----|-------|
| `develop-<SHA>` | Push sur develop |
| `release-<SHA>` | Push sur release/* |
| `latest-staging` | Dernier push sur release/* |
| `v1.2.3` | Release tag |
| `latest` | Dernier tag prod |
| `stable` | Post-deploy prod validé |

---

## Kamal v2 — Configuration multi-environnements

```yaml
# config/deploy.yml (production)
service: kilter-route-selector
image: ghcr.io/yourorg/kilter-route-selector

servers:
  web:
    hosts: ["prod.kilter.example"]
    labels:
      traefik.http.routers.kilter.rule: "Host(`api.kilter.example`)"
      traefik.http.routers.kilter-secure.tls.certresolver: letsencrypt

accessories:
  db:
    image: postgres:15-alpine
    host: "prod.kilter.example"
    volumes: ["postgres_data:/var/lib/postgresql/data"]
    env:
      secret: [POSTGRES_DB, POSTGRES_USER, POSTGRES_PASSWORD]
  cache:
    image: redis:7-alpine
    host: "prod.kilter.example"

env:
  secret:
    - DATABASE_URL
    - REDIS_URL
    - KILTERBOARD_USERNAME
    - KILTERBOARD_PASSWORD
    - SENTRY_DSN
```

```yaml
# config/deploy.staging.yml (surcharge)
servers:
  web:
    hosts: ["staging.kilter.example"]
```

```yaml
# config/deploy.integration.yml (surcharge)
servers:
  web:
    hosts: ["integration.kilter.example"]
```

### Commandes Kamal essentielles

```bash
kamal deploy -d integration    # déployer sur integration
kamal deploy -d staging        # déployer sur staging
kamal deploy                   # déployer en production
kamal rollback                 # rollback vers l'image précédente
kamal app logs                 # logs live du conteneur
kamal app exec "python -c 'import app'"  # debug rapide
```

---

## Secrets Management

```
┌─────────────────────────────────────────────────────┐
│                  GitHub Secrets (CI)                 │
│                                                      │
│  CI_REGISTRY_TOKEN      → push GHCR                 │
│  SSH_PRIVATE_KEY_INT    → integration VPS            │
│  SSH_PRIVATE_KEY_STG    → staging VPS                │
│  SSH_PRIVATE_KEY_PRD    → production VPS             │
│  KILTERBOARD_USERNAME                               │
│  KILTERBOARD_PASSWORD                               │
│  SENTRY_DSN                                         │
│  FLUTTER_KEYSTORE_B64   → APK signing (base64)      │
│  FLUTTER_KEYSTORE_PASS                              │
└─────────────────────────────────────────────────────┘
              │ injectés par Kamal au runtime
              ▼
┌─────────────────────────────────────────────────────┐
│              VPS .env (géré par Kamal)               │
│   DATABASE_URL, REDIS_URL, POSTGRES_PASSWORD, ...   │
│   Jamais dans l'image Docker                        │
└─────────────────────────────────────────────────────┘
```

**Règle d'or :** aucun secret ne transite en clair dans les logs CI.

---

## Flutter Builds Matrix

```yaml
# Dans release.yml
build-flutter:
  strategy:
    matrix:
      include:
        - os: ubuntu-latest
          target: android
          artifact: "build/app/outputs/**/*.apk,build/app/outputs/**/*.aab"
        - os: ubuntu-latest
          target: linux
          artifact: "build/linux/x64/release/bundle/**"
        - os: windows-latest
          target: windows
          artifact: "build/windows/x64/runner/Release/**"

  steps:
    - uses: subosito/flutter-action@v2
      with: { flutter-version: '3.x', channel: 'stable' }
    - run: flutter pub get && flutter test
    - name: Build Android
      if: matrix.target == 'android'
      run: |
        echo "$FLUTTER_KEYSTORE_B64" | base64 -d > android/app/keystore.jks
        flutter build apk --release
        flutter build appbundle --release
    - name: Build Linux
      if: matrix.target == 'linux'
      run: flutter build linux --release
    - name: Build Windows
      if: matrix.target == 'windows'
      run: flutter build windows --release
    - uses: actions/upload-artifact@v4
      with:
        name: flutter-${{ matrix.target }}-${{ github.ref_name }}
        path: ${{ matrix.artifact }}
```

---

## Stratégie de tests par environnement

| Env | DB | Tests | Gate |
|-----|-----|-------|------|
| CI gate (PR) | SQLite en mémoire | unit + lint + flutter | Coverage ≥ 80% |
| Integration | postgres service container | unit + integration | CI vert |
| Staging | PostgreSQL VPS | unit + integration + smoke | Trivy CRITICAL = 0 |
| Production | PostgreSQL VPS | smoke post-deploy | Approval manuelle |

---

## Observabilité

| Outil | Rôle | Intégration CI/CD |
|-------|------|------------------|
| Sentry | Erreurs runtime FastAPI + Flutter | `sentry-cli releases new` sur chaque deploy |
| Prometheus | Métriques API (latence, rps, erreurs) | Accessory Kamal sur prod VPS |
| Grafana | Dashboards + alertes | Annotation sur chaque déploiement |
| Loki + Promtail | Logs centralisés | Accessory Kamal sur prod VPS |
| Uptime Kuma | Alertes uptime | Webhook Discord si `/health` tombe |

---

## Récapitulatif Tooling

| Rôle | Outil |
|------|-------|
| CI | GitHub Actions |
| Container registry | GHCR (gratuit) |
| CD | Kamal v2 |
| Reverse proxy + TLS | Traefik (intégré Kamal) |
| Secrets CI | GitHub Secrets |
| Secrets runtime | Kamal `.env` sur VPS |
| Sécurité images | Trivy |
| Sécurité dépendances Python | pip-audit |
| Coverage backend | pytest-cov |
| Coverage Flutter | flutter test --coverage |
| Erreurs runtime | Sentry |
| Métriques | Prometheus + Grafana |
| Logs | Loki + Promtail |
| Uptime | Uptime Kuma |
