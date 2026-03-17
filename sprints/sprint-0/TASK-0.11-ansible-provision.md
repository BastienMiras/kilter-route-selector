# TASK-0.11 - Inventory + group_vars + playbook-provision.yml

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.8, TASK-0.9, TASK-0.10

## Objectif

Créer l'inventaire Ansible, les variables par environnement et le playbook de provisioning qui bootstrap un VPS vierge (installation Podman, pare-feu, création de l'utilisateur deploy et des répertoires d'application).

## Checklist

- [ ] Créer `infra/ansible/ansible.cfg`
- [ ] Créer `infra/ansible/inventory/vps.yml`
- [ ] Créer `infra/ansible/group_vars/all.yml`
- [ ] Créer `infra/ansible/group_vars/dev.yml`
- [ ] Créer `infra/ansible/group_vars/staging.yml`
- [ ] Créer `infra/ansible/group_vars/prod.yml`
- [ ] Créer `infra/ansible/playbook-provision.yml`
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
ghcr_username: bastienmiras
# ghcr_token: provided via CI secret or ansible-vault

app_base_dir: /opt
```

**infra/ansible/group_vars/dev.yml**
```yaml
env: dev
app_port: 8082
image_tag: dev-latest
log_level: debug
```

**infra/ansible/group_vars/staging.yml**
```yaml
env: staging
app_port: 8081
image_tag: develop-latest
log_level: info
```

**infra/ansible/group_vars/prod.yml**
```yaml
env: prod
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
