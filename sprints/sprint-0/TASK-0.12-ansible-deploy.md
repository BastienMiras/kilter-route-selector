# TASK-0.12 - playbook-deploy.yml

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.11

## Description

Le playbook de déploiement (`playbook-deploy.yml`) est distinct du playbook de provisioning. Là où
le provisioning configure le serveur une fois pour toutes, le déploiement est exécuté à chaque mise
en production d'une nouvelle version — potentiellement plusieurs fois par jour en staging.

Ce playbook est **générique** : il déploie n'importe quel environnement selon les paramètres qu'on
lui passe. L'environnement (`env=dev`, `env=staging` ou `env=prod`) et le tag d'image (`image_tag`)
sont passés via `-e` (extra vars) au moment de l'appel. La variable `env` est marquée `mandatory`,
ce qui fait échouer le playbook avec un message d'erreur clair si on oublie de la passer — plutôt
qu'un comportement imprévisible.

`image_tag` a un fallback intelligent : si non fourni, il utilise `env + '-latest'` (par exemple
`staging-latest`). Cela permet de tester rapidement sans avoir à calculer un tag précis, tout en
permettant aux workflows CI de passer un tag de commit précis pour la traçabilité.

`gather_facts: false` désactive la collecte d'informations système au début du playbook. Le rôle
`app` n'a pas besoin de ces informations (contrairement au rôle `podman` qui pourrait en avoir
besoin pour adapter les commandes selon l'OS). Désactiver `gather_facts` accélère légèrement
l'exécution.

**Comment réaliser cette tâche :**

1. Crée `infra/ansible/playbook-deploy.yml` : un playbook minimaliste qui déclare les variables
   `env` (mandatory) et `image_tag` (avec default), et appelle uniquement le rôle `app`.
2. Assure-toi que `gather_facts: false` est présent — le rôle app n'en a pas besoin.
3. Teste la validation des paramètres : appelle le playbook sans `-e env=` et vérifie qu'Ansible
   produit bien une erreur explicite.
4. Vérifie la syntaxe avec `ansible-playbook --syntax-check playbook-deploy.yml` et lance
   `ansible-lint playbook-deploy.yml`.

## Objectif

Créer le playbook de déploiement Ansible qui appelle le rôle `app` pour un environnement donné, en recevant `env` et `image_tag` comme paramètres obligatoires au moment de l'exécution.

## Checklist

- [ ] Créer `infra/ansible/playbook-deploy.yml`
- [ ] Configurer `env` comme variable obligatoire (`| mandatory`)
- [ ] Configurer `image_tag` avec fallback sur `env + '-latest'`
- [ ] Vérifier la syntaxe du playbook avec `--syntax-check`
- [ ] Linter le playbook

## Structure attendue

```
infra/ansible/
└── playbook-deploy.yml
```

## Exemple de contenu

**infra/ansible/playbook-deploy.yml**
```yaml
---
# Deploy a specific environment
# Usage: ansible-playbook infra/ansible/playbook-deploy.yml -e env=staging -e image_tag=develop-abc1234
- name: Deploy application
  hosts: vps
  gather_facts: false

  vars:
    env: "{{ env | mandatory }}"
    image_tag: "{{ image_tag | default(env + '-latest') }}"

  roles:
    - app
```

**Exemples d'utilisation**
```bash
# Déployer staging avec un tag précis
ansible-playbook infra/ansible/playbook-deploy.yml \
  -e env=staging \
  -e image_tag=develop-abc1234

# Déployer dev avec le tag par défaut (dev-latest)
ansible-playbook infra/ansible/playbook-deploy.yml \
  -e env=dev

# Déployer prod avec un tag de release
ansible-playbook infra/ansible/playbook-deploy.yml \
  -e env=prod \
  -e image_tag=v1.0.0
```

**Vérification syntaxe**
```bash
cd infra/ansible
ansible-playbook --syntax-check playbook-deploy.yml
# Expected: playbook: playbook-deploy.yml (no errors)
```

## Critères de validation

- `ansible-playbook --syntax-check playbook-deploy.yml` passe sans erreur
- `ansible-lint playbook-deploy.yml` ne retourne pas d'erreurs bloquantes
- Appeler le playbook sans `-e env=` produit une erreur Ansible (variable mandatory)
- Le playbook appelle uniquement le rôle `app` (pas de tasks en ligne)
- `gather_facts: false` est présent (le rôle app n'a pas besoin des facts)

## Ressources

- [Ansible playbook documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_intro.html)
- [Ansible mandatory filter](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#mandatory)
