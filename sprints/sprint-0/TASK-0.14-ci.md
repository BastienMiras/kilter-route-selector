# TASK-0.14 - ci.yml (lint + tests + image build gate)

**Sprint** : 0
**Estimation** : 2h
**Priorité** : Haute
**Dépendances** : TASK-0.2, TASK-0.4

## Objectif

Créer le workflow GitHub Actions de CI qui bloque toute PR vers `develop` ou `main` en cas d'échec du lint backend (ruff), des tests pytest ou du build des images Podman.

## Checklist

- [ ] Créer `.github/workflows/ci.yml`
- [ ] Ajouter le job `lint-backend` (ruff, Python 3.12)
- [ ] Ajouter le job `test-backend` (pytest, vacuous en Sprint 0)
- [ ] Ajouter le job `build-images` (build Containerfile.backend + Containerfile.frontend + health-check)
- [ ] Valider la syntaxe du workflow avec actionlint si disponible

## Structure attendue

```
.github/workflows/
└── ci.yml
```

## Exemple de contenu

**.github/workflows/ci.yml**
```yaml
name: CI

on:
  pull_request:
    branches: [develop, main]

jobs:
  lint-backend:
    name: Lint backend (ruff)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install ruff
      - run: ruff check backend/

  test-backend:
    name: Test backend (pytest)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -r backend/requirements.txt pytest pytest-asyncio httpx
      - run: pytest backend/tests/ -v --tb=short
        # Sprint 0: no tests yet — this will pass vacuously until Sprint 1 adds tests

  build-images:
    name: Build container images (dry-run)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install Podman
        run: sudo apt-get update && sudo apt-get install -y podman
      - name: Build backend image
        run: podman build -f infra/Containerfile.backend -t kilter-backend:ci .
      - name: Build frontend image
        run: podman build -f infra/Containerfile.frontend -t kilter-frontend:ci .
      - name: Health-check backend image
        run: |
          podman run --rm -d --name ci-backend -p 8000:8000 kilter-backend:ci
          sleep 3
          curl --fail http://localhost:8000/health
          podman stop ci-backend
```

Notes importantes :
- Les jobs lint et test frontend (eslint, vitest) sont intentionnellement absents — aucun code React n'existe en Sprint 0. Ils seront ajoutés en Sprint 3.
- Les secrets `ENV_STAGING` et `ENV_PROD` sont consommés par Ansible via `env.j2` et `group_vars`, non injectés dans ce workflow.

**Validation syntaxe locale**
```bash
which actionlint && actionlint .github/workflows/ci.yml || echo "actionlint not installed — skip local lint"
```

## Critères de validation

- Le workflow se déclenche sur les PRs vers `develop` et `main`
- Le job `lint-backend` utilise ruff sur `backend/`
- Le job `test-backend` installe les dépendances depuis `backend/requirements.txt`
- Le job `build-images` build les deux Containerfiles et health-check le backend
- `curl --fail` est utilisé (retourne un exit code non-zéro si la réponse n'est pas 2xx)
- Le workflow bloque la PR si l'un des jobs échoue

## Ressources

- [GitHub Actions documentation](https://docs.github.com/en/actions)
- [ruff linter](https://docs.astral.sh/ruff/)
- [actionlint](https://github.com/rhysd/actionlint)
