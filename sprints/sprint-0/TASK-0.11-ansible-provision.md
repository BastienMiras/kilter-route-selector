# TASK-0.11 - Inventory + group_vars + playbook-provision.yml

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.8, TASK-0.9, TASK-0.10

## Description

Les rôles Ansible créés dans les tâches précédentes sont des briques réutilisables, mais elles ne
font rien par elles-mêmes. Un **playbook** est le fichier qui orchestre ces rôles : il dit « exécute
ces rôles sur ces serveurs ». Un **inventaire** est la liste des serveurs sur lesquels Ansible doit
opérer. Les **group_vars** sont les variables associées à chaque groupe de serveurs.

Le **playbook de provisioning** ne s'exécute qu'une seule fois, lors de la création initiale du VPS
(ou pour remettre un serveur à plat). Il installe Podman, configure le pare-feu, crée l'utilisateur
`deploy` (un utilisateur dédié au déploiement, sans droits sudo inutiles) et crée les répertoires
de données pour les trois environnements.

Les **group_vars** permettent de définir des variables différentes selon l'environnement :
- `group_vars/all.yml` : variables communes à tous les environnements.
- `group_vars/dev.yml` : `env=dev`, `app_port=8082`, `image_tag=dev-latest`, `log_level=debug`.
- `group_vars/staging.yml` : `env=staging`, `app_port=8081`, `log_level=info`.
- `group_vars/prod.yml` : `env=prod`, `app_port=80`, `log_level=warning`.

`ansible.cfg` configure les paramètres par défaut d'Ansible pour ce projet : chemin de l'inventaire,
utilisateur SSH, clé privée, désactivation de la vérification d'empreinte SSH (pratique en dev).
Ce fichier est créé dans **TASK-0.8** (première tâche Ansible) pour être disponible dès que l'on
lance des commandes ansible-lint sur les rôles.

**Gestion des secrets** : les valeurs sensibles (`ghcr_token`, clés API) ne doivent **jamais** être
stockées dans `group_vars/` ni dans git. Le flux correct est :
- `ghcr_token` est injecté au moment de l'appel depuis GitHub Secrets : `-e "ghcr_token=${{ secrets.GHCR_TOKEN }}"`
- `kilter_api_token` suit le même principe : `-e "kilter_api_token=${{ secrets.KILTER_API_TOKEN }}"`
- `group_vars/all.yml` contient seulement `ghcr_username` (non sensible) et un commentaire rappelant
  que `ghcr_token` vient du CI. Ne jamais y mettre de valeur réelle.

**Comment réaliser cette tâche :**

1. Crée `inventory/vps.yml` avec un placeholder `VPS_IP` à remplacer par l'IP réelle du serveur.
2. Crée les fichiers `group_vars/` pour les quatre groupes (`all`, `dev`, `staging`, `prod`).
3. Crée `playbook-provision.yml` qui applique les rôles `podman` et `firewall`, puis crée
   l'utilisateur `deploy` et les répertoires `/opt/kilter-{dev,staging,prod}`.
4. Vérifie avec `ansible-playbook --syntax-check playbook-provision.yml` et `ansible-lint .`.

## Objectif

Créer l'inventaire Ansible, les variables par environnement et le playbook de provisioning qui bootstrap un VPS vierge (installation Podman, pare-feu, création de l'utilisateur deploy et des répertoires d'application).

## Checklist

- [ ] Vérifier que `infra/ansible/ansible.cfg` existe (créé dans TASK-0.8)
- [ ] Créer `infra/ansible/inventory/vps.yml`
- [ ] Créer `infra/ansible/group_vars/all.yml`
- [ ] Créer `infra/ansible/group_vars/dev.yml`
- [ ] Créer `infra/ansible/group_vars/staging.yml`
- [ ] Créer `infra/ansible/group_vars/prod.yml`
- [ ] Créer `infra/ansible/playbook-provision.yml`
- [ ] Installer les collections : `ansible-galaxy collection install -r requirements.yml`
- [ ] Vérifier la syntaxe du playbook avec `--syntax-check`
- [ ] Linter tout le répertoire ansible avec `ansible-lint`

## Structure attendue

```
infra/ansible/
├── ansible.cfg
├── inventory/
│   └── vps.yml
├── group_vars/
│   ├── all.yml
│   ├── dev.yml
│   ├── staging.yml
│   └── prod.yml
└── playbook-provision.yml
```

## Exemple de contenu

**infra/ansible/ansible.cfg**
```ini
[defaults]
inventory          = inventory/vps.yml
remote_user        = deploy
private_key_file   = ~/.ssh/kilter_vps
host_key_checking  = False
stdout_callback    = yaml
```

**infra/ansible/inventory/vps.yml**
```yaml
# Replace VPS_IP with your server's IP address
all:
  hosts:
    vps:
      ansible_host: VPS_IP          # TODO: replace with real IP
      ansible_user: deploy
      ansible_ssh_private_key_file: ~/.ssh/kilter_vps
```

**infra/ansible/group_vars/all.yml**
```yaml
# Common variables for all environments
deploy_user: deploy
ghcr_username: bastienmiras  # TODO: replace with your GitHub username if forking
# ghcr_token: NEVER store here — injected at deploy time via CI secret:
#   ansible-playbook ... -e "ghcr_token=${{ secrets.GHCR_TOKEN }}"

app_base_dir: /opt
```

**infra/ansible/group_vars/dev.yml**
```yaml
env: dev
compose_name: vps-dev   # → infra/compose.vps-dev.yml (≠ compose.dev.yml qui est le compose local)
app_port: 8082
image_tag: dev-latest
log_level: debug
```

**infra/ansible/group_vars/staging.yml**
```yaml
env: staging
compose_name: staging   # → infra/compose.staging.yml
app_port: 8081
image_tag: develop-latest
log_level: info
```

**infra/ansible/group_vars/prod.yml**
```yaml
env: prod
compose_name: prod      # → infra/compose.prod.yml
app_port: 80
# image_tag: set at deploy time (e.g. v1.2.3)
log_level: warning
```

**infra/ansible/playbook-provision.yml**
```yaml
---
# Bootstrap a fresh VPS: install Podman, configure firewall, create app directories
# Run once: ansible-playbook infra/ansible/playbook-provision.yml
- name: Provision VPS
  hosts: vps
  gather_facts: true

  roles:
    - podman
    - firewall

  post_tasks:
    - name: Create deploy user
      ansible.builtin.user:
        name: "{{ deploy_user }}"
        groups: sudo
        shell: /bin/bash
        create_home: true
      become: true

    - name: Create app directories for all environments
      ansible.builtin.file:
        path: "/opt/kilter-{{ item }}"
        state: directory
        owner: "{{ deploy_user }}"
        mode: "0750"
      loop:
        - dev
        - staging
        - prod
      become: true
```

**Vérification syntaxe**
```bash
cd infra/ansible
ansible-playbook --syntax-check playbook-provision.yml
# Expected: playbook: playbook-provision.yml (no errors)
```

**Lint**
```bash
ansible-lint .
# Expected: no errors (warnings about vault are acceptable)
```

## Critères de validation

- `ansible-playbook --syntax-check playbook-provision.yml` passe sans erreur
- `ansible-lint .` ne retourne pas d'erreurs bloquantes
- L'inventaire contient un placeholder `VPS_IP` (à remplacer manuellement)
- Les `group_vars` couvrent les trois environnements (dev, staging, prod)
- Le playbook crée l'utilisateur `deploy` et les répertoires `/opt/kilter-{dev,staging,prod}`

## Ressources

- [Ansible inventory documentation](https://docs.ansible.com/ansible/latest/inventory_guide/index.html)
- [Ansible group_vars](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#organizing-host-and-group-variables)
- [ansible-lint](https://ansible-lint.readthedocs.io/)
