# TASK-0.16 - deploy-prod.yml

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.15

## Description

Le déploiement en production est beaucoup plus prudent que le staging. On ne déploie pas
automatiquement chaque commit — on déploie uniquement des versions explicitement taguées et
approuvées manuellement. Cette rigueur est normale : une erreur en production impacte les
utilisateurs réels.

**Le déclencheur par tag** : le workflow se déclenche uniquement quand on crée un tag git suivant
le format `v*.*.*` (semantic versioning : `v1.0.0`, `v1.2.3`, etc.). Créer un tag est un geste
délibéré qui signifie « cette version est prête pour la production ».

**Re-tagging au lieu de rebuild** : plutôt que de reconstruire les images depuis zéro, on réutilise
les images staging déjà testées (tag `develop-latest`). On leur applique simplement un nouveau tag
(`v1.0.0` et `latest`). C'est plus rapide et garantit qu'exactement les mêmes images qui ont tourné
en staging vont en production — pas une reconstruction potentiellement différente.

**GitHub Environments et approbation manuelle** : `environment: production` dans le job de déploiement
active le système d'approbation de GitHub. Quand le workflow arrive au job `deploy`, il se met en
pause et envoie une notification aux reviewers configurés. Un humain doit cliquer « Approve » dans
l'interface GitHub pour que le déploiement continue. C'est la barrière de sécurité finale.

**Comment réaliser cette tâche :**

1. Crée `.github/workflows/deploy-prod.yml` avec le déclencheur `push: tags: ["v*.*.*"]`.
2. Dans le job `build-and-push` : extrait le tag de version depuis `GITHUB_REF` (variable d'environnement
   GitHub qui contient `refs/tags/v1.0.0`), pull les images `develop-latest`, re-tag et push.
3. Dans le job `deploy` : ajoute `environment: production`. Le reste est identique au staging mais
   avec `env=prod`. Ajoute un step `--check` (dry-run) avant le déploiement réel pour détecter les
   erreurs Ansible sans toucher au serveur.
4. Configure l'environnement `production` dans les settings GitHub du repo (Settings → Environments
   → New environment) et ajoute les reviewers requis.

**Note TLS/HTTPS :** ce workflow déploie sur le port 80 (HTTP). Pour la production réelle, configurer
HTTPS via Let's Encrypt (Certbot ou l'action `certbot/certbot-github-action`). Le certificat SSL doit
être géré dans le rôle `firewall` ou dans un rôle `nginx` dédié. Cette tâche est prévue en **Sprint 3**
lors de la finalisation de nginx.conf (TASK-3.4). En Sprint 0, HTTP est accepté pour le VPS staging.

## Objectif

Créer le workflow GitHub Actions de déploiement production déclenché sur les tags `v*.*.*`, avec approbation manuelle obligatoire via GitHub Environments avant le déploiement sur le VPS.

## Checklist

- [ ] Créer `.github/workflows/deploy-prod.yml`
- [ ] Configurer le déclencheur sur `push` vers les tags `v*.*.*`
- [ ] Ajouter le job `build-and-push` (extraction du tag de version, re-tag des images staging vers prod)
- [ ] Ajouter le job `deploy` avec `environment: production` (approbation manuelle)
- [ ] Ajouter le step Ansible `--check` (dry-run) avant le déploiement réel
- [ ] Configurer l'environnement GitHub `production` dans les settings du repo
- [ ] Documenter la procédure de rollback (re-déployer un tag précédent via Ansible)

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

      - name: Dry-run deploy (check mode)
        run: |
          ansible-playbook infra/ansible/playbook-deploy.yml \
            --check \
            -e env=prod \
            -e "image_tag=${{ needs.build-and-push.outputs.image_tag }}" \
            -e "ghcr_token=${{ secrets.GHCR_TOKEN }}"
        # --check détecte les erreurs Ansible avant d'appliquer en prod

      - name: Deploy production
        run: |
          ansible-playbook infra/ansible/playbook-deploy.yml \
            -e env=prod \
            -e "image_tag=${{ needs.build-and-push.outputs.image_tag }}" \
            -e "ghcr_token=${{ secrets.GHCR_TOKEN }}"
```

**Procédure de rollback (si le déploiement prod casse quelque chose) :**
```bash
# 1. Identifier le dernier tag fonctionnel
git tag --sort=-creatordate | head -5

# 2. Re-déployer manuellement l'ancienne version via Ansible
ansible-playbook infra/ansible/playbook-deploy.yml \
  -e env=prod \
  -e "image_tag=v1.2.3" \   # ← tag fonctionnel précédent
  -e "ghcr_token=<token>"

# 3. Ou via make (si Makefile configuré)
make deploy-prod IMAGE_TAG=v1.2.3
```
Les images GHCR sont conservées avec leurs tags de version — un rollback ne nécessite pas de rebuild.

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
