# TASK-0.8 - Ansible role: podman

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : Aucune

## Objectif

Créer le rôle Ansible `podman` qui installe Podman et podman-compose sur un VPS Ubuntu/Debian via apt et pip, de manière idempotente.

## Checklist

- [ ] Créer la structure de répertoires des rôles Ansible (`roles/podman/`, `roles/firewall/`, `roles/app/`)
- [ ] Créer `infra/ansible/roles/podman/meta/main.yml`
- [ ] Créer `infra/ansible/roles/podman/tasks/main.yml`
- [ ] Linter le rôle avec `ansible-lint`

## Structure attendue

```
infra/ansible/
└── roles/
    ├── podman/
    │   ├── meta/
    │   │   └── main.yml
    │   └── tasks/
    │       └── main.yml
    ├── firewall/
    │   ├── meta/
    │   └── tasks/
    └── app/
        ├── meta/
        ├── tasks/
        └── templates/
```

## Exemple de contenu

**infra/ansible/roles/podman/meta/main.yml**
```yaml
galaxy_info:
  role_name: podman
  description: Install Podman and podman-compose on Ubuntu/Debian
  min_ansible_version: "2.14"
dependencies: []
```

**infra/ansible/roles/podman/tasks/main.yml**
```yaml
---
- name: Install podman package
  ansible.builtin.apt:
    name: podman
    state: present
    update_cache: yes
  become: true

- name: Install pip3
  ansible.builtin.apt:
    name: python3-pip
    state: present
  become: true

- name: Install podman-compose via pip
  ansible.builtin.pip:
    name: podman-compose
    state: present
    executable: pip3
  become: true

- name: Verify podman is installed
  ansible.builtin.command: podman --version
  changed_when: false
  register: podman_version

- name: Print podman version
  ansible.builtin.debug:
    msg: "{{ podman_version.stdout }}"

- name: Verify podman-compose is installed
  ansible.builtin.command: podman-compose --version
  changed_when: false
```

**Lint**
```bash
cd infra/ansible
ansible-lint roles/podman/
# Expected: no errors (install ansible-lint first: pip install ansible-lint)
```

## Critères de validation

- `ansible-lint roles/podman/` ne retourne aucune erreur
- Le rôle utilise `ansible.builtin.*` (FQCN) pour tous les modules
- `changed_when: false` est présent sur les tasks de vérification
- La structure `meta/main.yml` est valide

## Ressources

- [Ansible apt module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)
- [Ansible pip module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/pip_module.html)
- [ansible-lint](https://ansible-lint.readthedocs.io/)
