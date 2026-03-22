# TASK-0.15 - deploy-staging.yml

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.14, TASK-0.12

## Description

Le déploiement en staging est **automatique** : à chaque push sur la branche `develop` (ce qui
correspond à chaque PR mergée), les nouvelles images sont buildées, publiées sur GHCR et déployées
sur le VPS sans intervention humaine. C'est le principe du **Continuous Deployment** (CD).

Ce workflow a deux jobs qui s'exécutent séquentiellement :

**Job 1 — build-and-push :**
- Calcule un tag d'image unique basé sur le SHA git court (`develop-abc1234`). Cela permet de
  savoir exactement quelle version du code est déployée, et de revenir en arrière facilement.
- Se connecte à GHCR avec le secret `GHCR_TOKEN` (un Personal Access Token GitHub).
- Build et push les deux images avec **deux tags** : le tag précis (`develop-abc1234`) pour la
  traçabilité, et `develop-latest` pour les déploiements sans tag précis.

**Job 2 — deploy :**
- Attend que le job 1 soit terminé (`needs: build-and-push`).
- Récupère le tag d'image calculé par le job 1 via `outputs` (mécanisme de communication entre jobs).
- Installe Ansible et écrit la clé SSH privée (stockée en secret GitHub) sur le filesystem du runner.
- Remplace le placeholder `VPS_IP` dans l'inventaire par l'IP réelle du VPS (aussi en secret).
- Lance le playbook de déploiement pour l'environnement staging.

**Comment réaliser cette tâche :**

1. Crée `.github/workflows/deploy-staging.yml` avec le déclencheur `push` sur `develop`.
2. Configure le job `build-and-push` avec le calcul du tag, le login GHCR, le build et le push des
   deux images. Expose le tag en `output` du job.
3. Configure le job `deploy` qui dépend du premier, installe Ansible, écrit la clé SSH (permissions
   `chmod 600` obligatoires), patche l'inventaire et lance le playbook.
4. Documente les trois secrets GitHub requis : `GHCR_TOKEN`, `VPS_HOST`, `VPS_SSH_KEY`.

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
