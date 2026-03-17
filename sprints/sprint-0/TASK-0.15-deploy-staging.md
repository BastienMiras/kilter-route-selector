# TASK-0.15 - deploy-staging.yml

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.14, TASK-0.12

## Objectif

Créer le workflow GitHub Actions de déploiement staging déclenché automatiquement sur chaque push vers la branche `develop` : build + push des images vers GHCR, puis déploiement via Ansible sur le VPS.

## Checklist

- [ ] Créer `.github/workflows/deploy-staging.yml`
- [ ] Configurer le déclencheur sur `push` vers `develop`
- [ ] Ajouter le job `build-and-push` (calcul du tag, login GHCR, build + push backend et frontend)
- [ ] Ajouter le job `deploy` (installation Ansible, écriture clé SSH, déploiement via playbook-deploy.yml)
- [ ] Passer `image_tag` entre les jobs via `outputs`

## Structure attendue

```
.github/workflows/
└── deploy-staging.yml
```

## Exemple de contenu

**.github/workflows/deploy-staging.yml**
```yaml
name: Deploy Staging

on:
  push:
    branches: [develop]

jobs:
  build-and-push:
    name: Build and push images to GHCR
    runs-on: ubuntu-latest
    outputs:
      image_tag: ${{ steps.tag.outputs.tag }}
    steps:
      - uses: actions/checkout@v4

      - name: Compute image tag
        id: tag
        run: echo "tag=develop-$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

      - name: Install Podman
        run: sudo apt-get update && sudo apt-get install -y podman

      - name: Log in to GHCR
        run: echo "${{ secrets.GHCR_TOKEN }}" | podman login ghcr.io -u bastienmiras --password-stdin

      - name: Build and push backend
        run: |
          podman build -f infra/Containerfile.backend \
            -t ghcr.io/bastienmiras/kilter-route-selector-backend:${{ steps.tag.outputs.tag }} \
            -t ghcr.io/bastienmiras/kilter-route-selector-backend:develop-latest \
            .
          podman push ghcr.io/bastienmiras/kilter-route-selector-backend:${{ steps.tag.outputs.tag }}
          podman push ghcr.io/bastienmiras/kilter-route-selector-backend:develop-latest

      - name: Build and push frontend
        run: |
          podman build -f infra/Containerfile.frontend \
            -t ghcr.io/bastienmiras/kilter-route-selector-frontend:${{ steps.tag.outputs.tag }} \
            -t ghcr.io/bastienmiras/kilter-route-selector-frontend:develop-latest \
            .
          podman push ghcr.io/bastienmiras/kilter-route-selector-frontend:${{ steps.tag.outputs.tag }}
          podman push ghcr.io/bastienmiras/kilter-route-selector-frontend:develop-latest

  deploy:
    name: Deploy to staging via Ansible
    runs-on: ubuntu-latest
    needs: build-and-push
    steps:
      - uses: actions/checkout@v4

      - name: Install Ansible
        run: pip install ansible ansible-lint

      - name: Install community.general collection
        run: ansible-galaxy collection install community.general

      - name: Write SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.VPS_SSH_KEY }}" > ~/.ssh/kilter_vps
          chmod 600 ~/.ssh/kilter_vps

      - name: Update VPS IP in inventory
        run: sed -i "s/VPS_IP/${{ secrets.VPS_HOST }}/g" infra/ansible/inventory/vps.yml

      - name: Deploy staging
        run: |
          ansible-playbook infra/ansible/playbook-deploy.yml \
            -e env=staging \
            -e "image_tag=${{ needs.build-and-push.outputs.image_tag }}" \
            -e "ghcr_token=${{ secrets.GHCR_TOKEN }}"
        working-directory: .
```

**Secrets GitHub requis :**
- `GHCR_TOKEN` — Personal Access Token avec permission `write:packages`
- `VPS_HOST` — IP du VPS
- `VPS_SSH_KEY` — Clé SSH privée (sans passphrase)

## Critères de validation

- Le workflow se déclenche sur push vers `develop` uniquement
- Le tag d'image est de la forme `develop-<short-sha>`
- Les images sont poussées avec deux tags : le tag précis et `develop-latest`
- Le job `deploy` dépend de `build-and-push` (`needs: build-and-push`)
- `image_tag` est passé entre jobs via `outputs`
- La clé SSH est écrite avec les permissions `600`

## Ressources

- [GitHub Actions secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [GHCR container registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [GitHub Actions job outputs](https://docs.github.com/en/actions/using-jobs/defining-outputs-for-jobs)
