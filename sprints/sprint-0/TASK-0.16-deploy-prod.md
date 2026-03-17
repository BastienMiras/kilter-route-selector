# TASK-0.16 - deploy-prod.yml

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.15

## Objectif

Créer le workflow GitHub Actions de déploiement production déclenché sur les tags `v*.*.*`, avec approbation manuelle obligatoire via GitHub Environments avant le déploiement sur le VPS.

## Checklist

- [ ] Créer `.github/workflows/deploy-prod.yml`
- [ ] Configurer le déclencheur sur `push` vers les tags `v*.*.*`
- [ ] Ajouter le job `build-and-push` (extraction du tag de version, re-tag des images staging vers prod)
- [ ] Ajouter le job `deploy` avec `environment: production` (approbation manuelle)
- [ ] Configurer l'environnement GitHub `production` dans les settings du repo

## Structure attendue

```
.github/workflows/
└── deploy-prod.yml
```

## Exemple de contenu

**.github/workflows/deploy-prod.yml**
```yaml
name: Deploy Production

on:
  push:
    tags:
      - "v*.*.*"

jobs:
  build-and-push:
    name: Tag and push production images
    runs-on: ubuntu-latest
    outputs:
      image_tag: ${{ steps.tag.outputs.tag }}
    steps:
      - uses: actions/checkout@v4

      - name: Extract version tag
        id: tag
        run: echo "tag=${GITHUB_REF#refs/tags/}" >> $GITHUB_OUTPUT

      - name: Install Podman
        run: sudo apt-get update && sudo apt-get install -y podman

      - name: Log in to GHCR
        run: echo "${{ secrets.GHCR_TOKEN }}" | podman login ghcr.io -u bastienmiras --password-stdin

      - name: Pull staging images and re-tag for prod
        run: |
          podman pull ghcr.io/bastienmiras/kilter-route-selector-backend:develop-latest
          podman tag \
            ghcr.io/bastienmiras/kilter-route-selector-backend:develop-latest \
            ghcr.io/bastienmiras/kilter-route-selector-backend:${{ steps.tag.outputs.tag }}
          podman tag \
            ghcr.io/bastienmiras/kilter-route-selector-backend:develop-latest \
            ghcr.io/bastienmiras/kilter-route-selector-backend:latest
          podman push ghcr.io/bastienmiras/kilter-route-selector-backend:${{ steps.tag.outputs.tag }}
          podman push ghcr.io/bastienmiras/kilter-route-selector-backend:latest

          podman pull ghcr.io/bastienmiras/kilter-route-selector-frontend:develop-latest
          podman tag \
            ghcr.io/bastienmiras/kilter-route-selector-frontend:develop-latest \
            ghcr.io/bastienmiras/kilter-route-selector-frontend:${{ steps.tag.outputs.tag }}
          podman tag \
            ghcr.io/bastienmiras/kilter-route-selector-frontend:develop-latest \
            ghcr.io/bastienmiras/kilter-route-selector-frontend:latest
          podman push ghcr.io/bastienmiras/kilter-route-selector-frontend:${{ steps.tag.outputs.tag }}
          podman push ghcr.io/bastienmiras/kilter-route-selector-frontend:latest

  deploy:
    name: Deploy to production via Ansible
    runs-on: ubuntu-latest
    needs: build-and-push
    environment: production    # ← GitHub Environments: requires manual approval
    steps:
      - uses: actions/checkout@v4

      - name: Install Ansible
        run: pip install ansible

      - name: Install community.general collection
        run: ansible-galaxy collection install community.general

      - name: Write SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.VPS_SSH_KEY }}" > ~/.ssh/kilter_vps
          chmod 600 ~/.ssh/kilter_vps

      - name: Update VPS IP in inventory
        run: sed -i "s/VPS_IP/${{ secrets.VPS_HOST }}/g" infra/ansible/inventory/vps.yml

      - name: Deploy production
        run: |
          ansible-playbook infra/ansible/playbook-deploy.yml \
            -e env=prod \
            -e image_tag=${{ needs.build-and-push.outputs.image_tag }} \
            -e ghcr_token=${{ secrets.GHCR_TOKEN }}
```

**Setup GitHub Environments (manuel, une fois) :**
```
GitHub → repo Settings → Environments → New environment → "production"
→ Add required reviewers → Save
```

**Secrets GitHub requis :**
- `GHCR_TOKEN` — Personal Access Token avec permission `write:packages`
- `VPS_HOST` — IP du VPS
- `VPS_SSH_KEY` — Clé SSH privée (sans passphrase)

## Critères de validation

- Le workflow se déclenche uniquement sur des tags correspondant à `v*.*.*`
- Le tag d'image est extrait depuis `GITHUB_REF` (ex: `v1.0.0`)
- Les images prod sont re-taguées depuis `develop-latest` (pas rebuildées)
- Le job `deploy` possède `environment: production` (approbation manuelle GitHub)
- Les images sont poussées avec deux tags : le tag de version et `latest`
- Le déploiement s'effectue avec `env=prod` dans le playbook

## Ressources

- [GitHub Environments documentation](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [GitHub Actions tag triggers](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow#triggering-a-workflow-from-a-workflow)
- [Semantic versioning](https://semver.org/)
