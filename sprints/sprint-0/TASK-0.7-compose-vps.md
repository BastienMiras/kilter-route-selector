# TASK-0.7 - compose.vps-dev/staging/prod.yml

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : TASK-0.6

## Description

Sur le VPS (le serveur distant), l'approche est différente du développement local. On ne veut pas
builder les images sur le serveur — c'est lent et le serveur n'a pas forcément les outils nécessaires.
À la place, les images sont **buildées en CI** (GitHub Actions) et **poussées vers GHCR** (GitHub
Container Registry, le registre d'images de GitHub), puis le serveur les **pull** (télécharge) et
les démarre. C'est le flux standard en production.

Les fichiers VPS compose utilisent donc `image:` au lieu de `build:` — ils pointent vers l'image
déjà construite sur GHCR avec un tag spécifique.

Le projet a **trois environnements sur le même VPS** :
- **vps-dev** (port 8082) : environnement de développement sur le VPS, déployé manuellement pour
  des tests ponctuels. Tag `dev-latest`.
- **staging** (port 8081) : déployé automatiquement à chaque push sur la branche `develop`. Sert
  à valider les nouvelles fonctionnalités avant la production. Tag `develop-latest`.
- **prod** (port 80) : l'environnement public, stable. Déployé uniquement sur des tags `v*.*.*`
  après approbation manuelle. Tag `latest`.

La variable `IMAGE_TAG` est configurable via l'environnement, avec une valeur par défaut. Cela
permet à Ansible de déployer un tag précis (`develop-abc1234`) tout en ayant un fallback.

**Différence clé avec `compose.dev.yml`** : en dev local, le backend est exposé directement sur
`:8000` pour faciliter les tests directs. Sur le VPS, le backend **n'expose pas** de port — il
est interne au réseau Podman et accessible uniquement via Nginx. Tout trafic passe par le
frontend Nginx (`app_port`), qui joue le rôle de reverse proxy. C'est pourquoi `curl localhost:8000`
ne fonctionne pas sur le VPS — il faut appeler `curl localhost:8081/api/health` (via Nginx).

**GHCR_ORG** : le namespace GHCR (`ghcr.io/bastienmiras/...`) est spécifique au compte GitHub du
projet. Si tu forkes le repo, remplace `bastienmiras` par ton nom d'utilisateur dans les trois
fichiers compose et dans `group_vars/all.yml`. Pour le CI, la valeur vient de `ghcr_username` dans
`group_vars/all.yml` — c'est la source de vérité unique.

**Comment réaliser cette tâche :**

1. Crée `infra/compose.vps-dev.yml` en vous basant sur le modèle : services `backend` et `frontend`
   avec `image: ghcr.io/bastienmiras/...`, `env_file` pointant vers `/opt/kilter-dev/.env`, port 8082.
2. Crée `infra/compose.staging.yml` de la même façon avec le port 8081 et le tag `develop-latest`.
   Ajoute des `deploy.resources` (limites mémoire/CPU) pour éviter qu'un bug en staging n'épuise
   les ressources du VPS et impacte les autres environnements.
3. Crée `infra/compose.prod.yml` avec le port 80, `restart: always` et des `deploy.resources`
   adaptées à la production. Le backend n'expose pas de port — seul le frontend Nginx est accessible.
4. Valide la syntaxe des trois fichiers avec `podman-compose -f <fichier> config`.

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
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: "256M"

  frontend:
    image: ghcr.io/bastienmiras/kilter-route-selector-frontend:${IMAGE_TAG:-develop-latest}
    ports:
      - "8081:80"
    depends_on:
      - backend
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: "0.25"
          memory: "64M"

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
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: "512M"

  frontend:
    image: ghcr.io/bastienmiras/kilter-route-selector-frontend:${IMAGE_TAG:-latest}
    ports:
      - "80:80"
    depends_on:
      - backend
    restart: always
    deploy:
      resources:
        limits:
          cpus: "0.5"
          memory: "128M"

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
