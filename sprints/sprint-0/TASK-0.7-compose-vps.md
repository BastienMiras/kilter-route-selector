# TASK-0.7 - compose.vps-dev/staging/prod.yml

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : TASK-0.6

## Objectif

Créer les trois variantes compose pour les environnements VPS (dev :8082, staging :8081, prod :80), chacune tirant ses images depuis GHCR avec un tag configurable.

## Checklist

- [ ] Créer `infra/compose.vps-dev.yml` (port 8082, tag `dev-latest`)
- [ ] Créer `infra/compose.staging.yml` (port 8081, tag `develop-latest`)
- [ ] Créer `infra/compose.prod.yml` (port 80, tag `latest`, restart `always`)
- [ ] Valider la syntaxe des trois fichiers avec `podman-compose config`

## Structure attendue

```
infra/
├── compose.vps-dev.yml
├── compose.staging.yml
└── compose.prod.yml
```

## Exemple de contenu

**infra/compose.vps-dev.yml**
```yaml
# Dev environment on VPS — port 8082
# Deployed manually via: make deploy-dev
# Images pulled from GHCR (tag: dev-latest or branch-<SHA>)

services:
  backend:
    image: ghcr.io/bastienmiras/kilter-route-selector-backend:${IMAGE_TAG:-dev-latest}
    volumes:
      - /opt/kilter-dev/data:/app/data
    env_file:
      - /opt/kilter-dev/.env
    restart: unless-stopped

  frontend:
    image: ghcr.io/bastienmiras/kilter-route-selector-frontend:${IMAGE_TAG:-dev-latest}
    ports:
      - "8082:80"
    depends_on:
      - backend
    restart: unless-stopped

networks:
  default:
    name: kilter-dev-net
```

**infra/compose.staging.yml**
```yaml
# Staging environment on VPS — port 8081
# Auto-deployed by CI on push to develop branch

services:
  backend:
    image: ghcr.io/bastienmiras/kilter-route-selector-backend:${IMAGE_TAG:-develop-latest}
    volumes:
      - /opt/kilter-staging/data:/app/data
    env_file:
      - /opt/kilter-staging/.env
    restart: unless-stopped

  frontend:
    image: ghcr.io/bastienmiras/kilter-route-selector-frontend:${IMAGE_TAG:-develop-latest}
    ports:
      - "8081:80"
    depends_on:
      - backend
    restart: unless-stopped

networks:
  default:
    name: kilter-staging-net
```

**infra/compose.prod.yml**
```yaml
# Production environment on VPS — port 80
# Deployed by CI on tag v*.*.* after manual approval

services:
  backend:
    image: ghcr.io/bastienmiras/kilter-route-selector-backend:${IMAGE_TAG:-latest}
    volumes:
      - /opt/kilter-prod/data:/app/data
    env_file:
      - /opt/kilter-prod/.env
    restart: always

  frontend:
    image: ghcr.io/bastienmiras/kilter-route-selector-frontend:${IMAGE_TAG:-latest}
    ports:
      - "80:80"
    depends_on:
      - backend
    restart: always

networks:
  default:
    name: kilter-prod-net
```

**Vérification syntaxe**
```bash
podman-compose -f infra/compose.vps-dev.yml config
podman-compose -f infra/compose.staging.yml config
podman-compose -f infra/compose.prod.yml config
# Expected: no errors, YAML printed
```

## Critères de validation

- Les trois fichiers passent `podman-compose config` sans erreur
- Chaque environnement utilise le bon port (8082 / 8081 / 80)
- Les images référencent `ghcr.io/bastienmiras/kilter-route-selector-{backend,frontend}`
- `IMAGE_TAG` est configurable via variable d'environnement avec fallback
- `compose.prod.yml` utilise `restart: always` (vs `unless-stopped` pour dev/staging)

## Ressources

- [podman-compose documentation](https://github.com/containers/podman-compose)
- [GHCR container registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
