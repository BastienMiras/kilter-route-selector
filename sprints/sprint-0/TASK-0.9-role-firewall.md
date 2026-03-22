# TASK-0.9 - Ansible role: firewall

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : Aucune

## Description

Un serveur VPS exposé sur internet reçoit en permanence des tentatives de connexion automatiques :
scanners de ports, bots qui cherchent des services mal configurés, tentatives d'intrusion. Sans
pare-feu, tous les ports du serveur sont accessibles depuis n'importe où dans le monde. C'est
extrêmement dangereux.

**ufw** (Uncomplicated Firewall) est l'outil de configuration du pare-feu sur Ubuntu. Il s'appuie
sur `iptables` (le pare-feu Linux bas niveau) mais offre une interface beaucoup plus simple. La
philosophie est simple : on part d'une politique **« deny all »** (bloquer tout trafic entrant par
défaut), puis on ouvre uniquement les ports nécessaires.

Les ports que nous ouvrons ont chacun une raison précise :
- **22 (SSH)** : pour que les administrateurs puissent se connecter et exécuter des commandes.
- **80 (HTTP)** : pour l'application en production (accessible au public).
- **443 (HTTPS)** : pour la future version HTTPS de la production.
- **8081 (staging)** : pour l'environnement de staging (accès restreint en pratique).
- **8082 (vps-dev)** : pour l'environnement de développement VPS.

Le module `community.general.ufw` est une collection Ansible tierce (non incluse par défaut) qui
apporte le support d'ufw. Elle doit être installée via `ansible-galaxy collection install -r
requirements.yml` avant d'utiliser le playbook. Le fichier `requirements.yml` déclare cette
dépendance.

**Comment réaliser cette tâche :**

1. Crée `infra/ansible/requirements.yml` déclarant la collection `community.general`.
2. Crée `roles/firewall/meta/main.yml` avec les métadonnées.
3. Crée `roles/firewall/tasks/main.yml` : installe ufw, ajoute les règles `allow` pour chaque port,
   puis active ufw avec `state: enabled` et `policy: deny` sur `incoming`. Termine par une task
   de vérification avec `ufw status verbose`.
4. Installe la collection avec `ansible-galaxy collection install -r requirements.yml` et lance
   `ansible-lint roles/firewall/` pour vérifier.

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
