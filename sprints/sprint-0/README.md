# Sprint 0 — Infrastructure as Code

**Durée estimée :** ~16h
**Statut :** Planifié

## Objectif

Mettre en place toute l'infrastructure avant d'écrire le moindre code applicatif. L'infrastructure est traitée comme du code : versionnée, reproductible, déployable depuis le repo seul.

## Stack

- **Podman** — conteneurs rootless
- **podman-compose** — orchestration déclarative
- **Ansible** — provisioning VPS idempotent
- **GitHub Actions** — CI/CD automatisé
- **GHCR** — registry d'images

## Architecture déployée

```
VPS unique
├── /opt/kilter-prod/     → nginx :80    → fastapi :8000 (prod)
├── /opt/kilter-staging/  → nginx :8081  → fastapi :8001 (staging)
└── /opt/kilter-dev/      → nginx :8082  → fastapi :8002 (dev-vps)

Local dev : compose.dev.yml (volumes montés, hot-reload)
```

## Tâches

| ID | Tâche | Estimé |
|----|-------|--------|
| 0.1 | Backend skeleton (main.py + requirements.txt) | 30min |
| 0.2 | Containerfile.backend (multi-stage) | 1h |
| 0.3 | Frontend placeholder + nginx.conf | 30min |
| 0.4 | Containerfile.frontend | 1h |
| 0.5 | .env.example + .gitignore | 30min |
| 0.6 | compose.dev.yml (local hot-reload) | 1h |
| 0.7 | compose.vps-dev/staging/prod.yml | 30min |
| 0.8 | Ansible role: podman | 1h |
| 0.9 | Ansible role: firewall | 1h |
| 0.10 | Ansible role: app (deploy + health check) | 2h |
| 0.11 | Ansible inventory + group_vars + playbook-provision.yml | 1h |
| 0.12 | Ansible playbook-deploy.yml | 1h |
| 0.13 | Makefile | 30min |
| 0.14 | GitHub Actions : ci.yml (lint + test + build) | 2h |
| 0.15 | GitHub Actions : deploy-staging.yml | 1h |
| 0.16 | GitHub Actions : deploy-prod.yml (approbation manuelle) | 1h |
| 0.17 | Documentation (sprint-0 README + TASKS-INDEX) | 30min |

**Total : 17 tâches, ~16h**

## Résultats attendus

- `make up` → stack locale complète (backend :8000, frontend :5173)
- `make provision` → VPS vierge configuré en 1 commande
- `make deploy-dev` → déploiement env dev sur VPS
- CI bloque toute PR qui casse le build ou les tests

## Références

- **Spec IaC :** `docs/plans/2026-03-11-iac-podman-design.md`
- **Plan d'implémentation :** `docs/superpowers/plans/2026-03-11-sprint-0-iac.md`
