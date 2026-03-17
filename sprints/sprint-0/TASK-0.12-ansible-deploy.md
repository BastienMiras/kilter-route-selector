# TASK-0.12 - playbook-deploy.yml

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.11

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
