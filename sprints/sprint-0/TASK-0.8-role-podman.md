# TASK-0.8 - Ansible role: podman

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : Aucune

## Description

**Ansible** est un outil d'automatisation de configuration de serveurs. Il permet d'écrire des
instructions (en YAML) qui décrivent l'état désiré du serveur, et Ansible se charge de rendre cet
état réel. Si Podman est déjà installé, Ansible ne fait rien (c'est l'**idempotence**). Si Podman
n'est pas installé, Ansible l'installe. On peut relancer les playbooks autant de fois qu'on veut
sans risque de casser quoi que ce soit.

Un **rôle** Ansible est une unité réutilisable qui regroupe les tasks, templates et variables liés
à une responsabilité précise. Le rôle `podman` n'a qu'une seule mission : s'assurer que Podman et
podman-compose sont installés sur le serveur.

`become: true` signifie qu'Ansible exécute la commande avec `sudo` — l'installation de paquets
nécessite des droits administrateur. Les modules `ansible.builtin.apt` et `ansible.builtin.pip`
sont les modules Ansible standards pour installer des paquets via apt (gestionnaire de paquets
Ubuntu/Debian) et pip (gestionnaire de paquets Python).

L'utilisation du **FQCN** (Fully Qualified Collection Name) comme `ansible.builtin.apt` au lieu de
simplement `apt` est une bonne pratique : cela rend les playbooks explicites sur la provenance de
chaque module et évite les ambiguïtés entre modules de même nom dans différentes collections.

`changed_when: false` sur les tasks de vérification indique à Ansible que ces commandes ne
modifient jamais l'état du système — elles ne font que lire — et donc elles ne doivent pas être
comptées comme des changements dans le rapport d'exécution.

**Comment réaliser cette tâche :**

1. Crée le répertoire `infra/ansible/` et les sous-répertoires des trois rôles : `roles/podman/`,
   `roles/firewall/`, `roles/app/`, chacun avec `meta/` et `tasks/` (et `templates/` pour `app`).
2. **Crée `infra/ansible/ansible.cfg` dès maintenant** — c'est le fichier de configuration Ansible
   local au projet. Il dit où trouver l'inventaire, quel utilisateur SSH utiliser, et désactive la
   vérification d'empreinte SSH (utile en dev). Sans ce fichier, toutes les commandes `ansible-*`
   doivent recevoir des paramètres manuellement à chaque appel. L'exemple est dans TASK-0.11.
3. Écris `roles/podman/meta/main.yml` avec les métadonnées du rôle (nom, description, version Ansible).
4. Écris `roles/podman/tasks/main.yml` avec les tasks d'installation de `podman` via apt, de `python3-pip`
   via apt, puis de `podman-compose` via pip. Ajoute des tasks de vérification.
5. Lance `ansible-lint roles/podman/` pour vérifier la conformité du code.

## Objectif

Créer le rôle Ansible `podman` qui installe Podman et podman-compose sur un VPS Ubuntu/Debian via apt et pip, de manière idempotente.

## Checklist

- [ ] Créer le répertoire `infra/ansible/` et la structure des trois rôles
- [ ] Créer `infra/ansible/ansible.cfg` (voir exemple dans TASK-0.11)
- [ ] Créer `infra/ansible/roles/podman/meta/main.yml`
- [ ] Créer `infra/ansible/roles/podman/tasks/main.yml`
- [ ] Linter le rôle avec `ansible-lint`

## Structure attendue

```
infra/ansible/
├── ansible.cfg          ← à créer dans cette tâche
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
