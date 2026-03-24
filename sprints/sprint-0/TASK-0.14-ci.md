# TASK-0.14 - ci.yml (lint + tests + image build gate)

**Sprint** : 0
**Estimation** : 2h
**Priorité** : Haute
**Dépendances** : TASK-0.2, TASK-0.4

## Description

La **CI** (Continuous Integration, intégration continue) est un ensemble de vérifications automatiques
qui s'exécutent à chaque modification du code. L'objectif est de détecter les problèmes le plus tôt
possible — avant qu'un code cassé ne soit fusionné dans la branche principale et ne bloque toute
l'équipe.

**GitHub Actions** est le service d'automatisation intégré à GitHub. Un fichier YAML dans
`.github/workflows/` décrit ce qui doit s'exécuter, quand et comment. GitHub met à disposition des
machines virtuelles (runners) pour exécuter ces workflows gratuitement pour les repos publics.

Notre workflow CI comporte trois jobs indépendants qui s'exécutent en parallèle :

1. **lint-backend** : vérifie que le code Python respecte les conventions de style avec **ruff**.
   Ruff est un linter Python ultra-rapide (écrit en Rust). Un code mal formaté ou avec des imports
   inutilisés est détecté ici.

2. **test-backend** : exécute les tests avec **pytest**. En Sprint 0, il n'y a pas encore de tests,
   donc ce job passe « vacuously » (sans rien faire). Il sera rempli en Sprint 1 et suivants.

3. **build-images** : build les deux images Containerfile avec Podman sur le runner CI, puis lance
   le backend pour vérifier que `/health` répond. `curl --fail` retourne un code d'erreur non-zéro
   si la réponse HTTP n'est pas 2xx, ce qui fait échouer le workflow.

Si l'un de ces trois jobs échoue, GitHub bloque la fusion de la PR. C'est le filet de sécurité.

**Comment réaliser cette tâche :**

1. Crée `.github/workflows/ci.yml` avec le déclencheur `pull_request` sur les branches `develop` et
   `main`.
2. Ajoute les trois jobs avec leurs steps : checkout, setup-python, installation des dépendances,
   exécution des commandes. Pour `build-images`, installe Podman sur le runner Ubuntu avec `apt-get`.
3. Ajoute le health-check avec `curl --fail` après avoir démarré le conteneur backend en arrière-plan.
4. Optionnellement, valide la syntaxe du workflow avec `actionlint` en local.

## Objectif

Créer le workflow GitHub Actions de CI qui bloque toute PR vers `develop` ou `main` en cas d'échec du lint backend (ruff), des tests pytest ou du build des images Podman.

## Checklist

- [ ] Créer `.github/workflows/ci.yml`
- [ ] Ajouter le job `lint-backend` (ruff, Python 3.12)
- [ ] Ajouter le job `test-backend` (pytest, vacuous en Sprint 0)
- [ ] Ajouter le job `build-images` (build Containerfile.backend + Containerfile.frontend + health-check)
- [ ] Ajouter le job `validate-env` (vérifier cohérence `.env.example` ↔ `env.j2`)
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

**Validation cohérence `.env.example` ↔ `env.j2` (job optionnel, recommandé) :**
```yaml
  validate-env:
    name: Validate .env.example matches env.j2
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check variable names are in sync
        run: |
          # Extrait les clés de .env.example (lignes KEY=value, sans commentaires)
          env_keys=$(grep -v '^#' infra/.env.example | grep '=' | cut -d= -f1 | sort)
          # Extrait les clés de env.j2 (idem)
          j2_keys=$(grep -v '^#' infra/ansible/roles/app/templates/env.j2 | grep '=' | cut -d= -f1 | sort)
          if [ "$env_keys" != "$j2_keys" ]; then
            echo "Drift détecté entre .env.example et env.j2 !"
            diff <(echo "$env_keys") <(echo "$j2_keys")
            exit 1
          fi
          echo "OK — clés identiques dans .env.example et env.j2"
```

Ce job prévient la dérive entre le fichier de documentation des variables (`.env.example`) et le
template Ansible qui génère le `.env` sur le serveur (`env.j2`). Sans ce contrôle, un développeur
peut ajouter une variable à `.env.example` sans l'ajouter à `env.j2`, et le serveur ne déploiera
jamais cette variable.

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
