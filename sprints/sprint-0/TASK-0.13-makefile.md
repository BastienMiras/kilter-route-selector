# TASK-0.13 - Makefile

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : TASK-0.6, TASK-0.7, TASK-0.11, TASK-0.12

## Objectif

Créer le Makefile à la racine du repo exposant des raccourcis développeur pour les opérations courantes : démarrage local, build, tests, provisioning VPS et déploiement.

## Checklist

- [ ] Créer `Makefile` à la racine du repo
- [ ] Ajouter la cible `help` avec auto-documentation via `grep`
- [ ] Ajouter les cibles `up`, `down`, `build`, `logs`, `test`
- [ ] Ajouter les cibles `provision` et `deploy-dev`
- [ ] Vérifier que `make help` affiche correctement toutes les cibles
- [ ] Vérifier que `make up` démarre le stack local

## Structure attendue

```
Makefile  (à la racine du repo)
```

## Exemple de contenu

**Makefile**
```makefile
# Kilter Route Selector — Developer shortcuts
# All compose commands run from the infra/ directory

COMPOSE_DEV     = podman-compose -f infra/compose.dev.yml
ANSIBLE_PLAYBOOK = ansible-playbook
ANSIBLE_DIR      = infra/ansible
ENV              ?= staging

.PHONY: help up down build logs test provision deploy-dev

help: ## Show this help
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2}'

up: ## Start local dev stack (hot-reload)
	$(COMPOSE_DEV) up

down: ## Stop local dev stack
	$(COMPOSE_DEV) down

build: ## Build images locally (no push)
	podman build -f infra/Containerfile.backend -t kilter-backend:local .
	podman build -f infra/Containerfile.frontend -t kilter-frontend:local .

logs: ## Tail logs from local dev stack
	$(COMPOSE_DEV) logs -f

test: ## Run all tests (backend + frontend when available)
	pytest backend/tests/ -v
	# Frontend tests added in Sprint 3: npm run test --prefix frontend

provision: ## Bootstrap a fresh VPS (run once)
	$(ANSIBLE_PLAYBOOK) $(ANSIBLE_DIR)/playbook-provision.yml

deploy-dev: ## Deploy dev environment on VPS
	$(ANSIBLE_PLAYBOOK) $(ANSIBLE_DIR)/playbook-deploy.yml -e env=dev
```

**Vérification**
```bash
make help
# Expected: formatted list of targets with descriptions

make up &
sleep 4
curl http://localhost:8000/health
# Expected: {"status":"ok"}
make down
```

Note : les indentations dans le Makefile doivent être des tabulations (`\t`), pas des espaces.

## Critères de validation

- `make help` affiche la liste formatée de toutes les cibles avec leur description
- `make up` démarre le stack local (backend :8000, frontend :5173)
- `make down` arrête le stack
- `make build` construit les deux images localement
- `make provision` appelle `ansible-playbook playbook-provision.yml`
- `make deploy-dev` appelle `ansible-playbook playbook-deploy.yml -e env=dev`

## Ressources

- [GNU Make documentation](https://www.gnu.org/software/make/manual/make.html)
- [Self-documenting Makefile pattern](https://marmelab.com/blog/2016/02/29/auto-documented-makefile.html)
