# TASK-0.9 - Ansible role: firewall

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : Aucune

## Objectif

Créer le rôle Ansible `firewall` qui configure ufw sur le VPS pour autoriser SSH (22), HTTP/HTTPS (80/443), staging (8081) et dev-vps (8082), avec politique par défaut `deny incoming`.

## Checklist

- [ ] Créer `infra/ansible/requirements.yml` (collection `community.general`)
- [ ] Créer `infra/ansible/roles/firewall/meta/main.yml`
- [ ] Créer `infra/ansible/roles/firewall/tasks/main.yml` avec toutes les règles ufw
- [ ] Activer ufw avec `state: enabled` et `policy: deny` sur `incoming`
- [ ] Ajouter une task de vérification et affichage du statut
- [ ] Linter le rôle avec `ansible-lint`

## Structure attendue

```
infra/ansible/
├── requirements.yml
└── roles/firewall/
    ├── meta/
    │   └── main.yml
    └── tasks/
        └── main.yml
```

## Exemple de contenu

**infra/ansible/requirements.yml**
```yaml
collections:
  - name: community.general
```

**infra/ansible/roles/firewall/meta/main.yml**
```yaml
galaxy_info:
  role_name: firewall
  description: Configure ufw firewall for kilter VPS
  min_ansible_version: "2.14"
dependencies: []
```

**infra/ansible/roles/firewall/tasks/main.yml**
```yaml
---
- name: Install ufw
  ansible.builtin.apt:
    name: ufw
    state: present
  become: true

- name: Allow SSH
  community.general.ufw:
    rule: allow
    port: "22"
    proto: tcp
  become: true

- name: Allow prod HTTP (port 80)
  community.general.ufw:
    rule: allow
    port: "80"
    proto: tcp
  become: true

- name: Allow prod HTTPS (port 443)
  community.general.ufw:
    rule: allow
    port: "443"
    proto: tcp
  become: true

- name: Allow staging (port 8081)
  community.general.ufw:
    rule: allow
    port: "8081"
    proto: tcp
  become: true

- name: Allow dev-vps (port 8082)
  community.general.ufw:
    rule: allow
    port: "8082"
    proto: tcp
  become: true

- name: Enable ufw (default deny incoming)
  community.general.ufw:
    state: enabled
    policy: deny
    direction: incoming
  become: true

- name: Verify ufw status
  ansible.builtin.command: ufw status verbose
  changed_when: false
  become: true
  register: ufw_status

- name: Print ufw status
  ansible.builtin.debug:
    msg: "{{ ufw_status.stdout_lines }}"
```

**Installer la collection et linter**
```bash
cd infra/ansible
ansible-galaxy collection install -r requirements.yml
ansible-lint roles/firewall/
# Expected: no errors
```

## Critères de validation

- `infra/ansible/requirements.yml` déclare `community.general`
- `ansible-galaxy collection install -r requirements.yml` s'exécute sans erreur
- `ansible-lint roles/firewall/` ne retourne aucune erreur
- Le rôle ouvre exactement les ports : 22, 80, 443, 8081, 8082
- `community.general.ufw` est utilisé (FQCN)
- `ufw status verbose` est vérifié à la fin du rôle
- La politique par défaut `deny incoming` est activée

## Ressources

- [community.general.ufw module](https://docs.ansible.com/ansible/latest/collections/community/general/ufw_module.html)
- [ufw documentation](https://help.ubuntu.com/community/UFW)
