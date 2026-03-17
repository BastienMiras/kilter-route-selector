# TASK-0.5 - .env.example + .gitignore

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : Aucune

## Objectif

Créer le fichier `.env.example` documentant toutes les variables d'environnement requises et mettre à jour `.gitignore` pour exclure les secrets et les données.

## Checklist

- [ ] Créer `infra/.env.example` avec toutes les variables documentées
- [ ] Ajouter une section infrastructure dans `.gitignore`
- [ ] Vérifier que `infra/.env` n'est pas tracké par git
- [ ] Vérifier que `infra/.env.example` est bien tracké (exception gitignore)

## Structure attendue

```
infra/
└── .env.example
.gitignore  (modifié)
```

## Exemple de contenu

**infra/.env.example**
```bash
# Copy this file to infra/.env and fill in the values
# NEVER commit infra/.env

# --- Backend ---
DATABASE_URL=sqlite:///./data/kilter.db

# --- Kilter Board API (optional for Sprint 1) ---
KILTER_API_TOKEN=

# --- App ---
ENVIRONMENT=dev   # dev | staging | prod
LOG_LEVEL=info
```

**Ajout dans .gitignore** (section à ajouter en bas du fichier)
```gitignore
# Infrastructure — never commit real secrets or data
infra/.env
infra/.env.*
!infra/.env.example
data/
*.db
```

## Critères de validation

- `infra/.env.example` est présent et contient toutes les variables nécessaires
- `git check-ignore infra/.env` confirme que le fichier `.env` est ignoré
- `git check-ignore infra/.env.example` ne retourne rien (fichier tracké)
- `git check-ignore data/` confirme que le dossier data est ignoré
- `git check-ignore kilter.db` confirme que les fichiers `.db` sont ignorés

## Ressources

- [gitignore documentation](https://git-scm.com/docs/gitignore)
- [dotenv format convention](https://12factor.net/config)
