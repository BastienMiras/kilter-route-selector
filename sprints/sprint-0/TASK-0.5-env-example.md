# TASK-0.5 - .env.example + .gitignore

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : Aucune

## Description

Toute application a besoin de configuration qui varie selon l'environnement : l'adresse de la base de
données n'est pas la même en local et en production, les clés d'API doivent rester secrètes, le niveau
de log est plus verbeux en développement. Les **variables d'environnement** sont le mécanisme standard
pour injecter cette configuration sans la coder en dur dans le source.

La convention est d'avoir deux fichiers :
- **`.env`** contient les vraies valeurs (secrets inclus). Il ne doit **jamais** être commité dans git
  — n'importe qui ayant accès au dépôt verrait les clés d'API et mots de passe. C'est pourquoi il
  est listé dans `.gitignore`.
- **`.env.example`** est une version publique avec des valeurs fictives ou vides qui documente quelles
  variables sont nécessaires. Quelqu'un qui clone le projet copie ce fichier, le renomme `.env` et
  remplit les valeurs. Ce fichier est commité dans git.

Cette convention vient du **12-factor app** (12factor.net), un ensemble de bonnes pratiques pour les
applications modernes. Elle garantit que les secrets ne fuient jamais dans l'historique git.

Le `.gitignore` protège aussi le dossier `data/` et les fichiers `.db` — la base de données SQLite
est générée localement et ne doit pas être versionnée (trop lourde, et chaque développeur a sa propre).

**Comment réaliser cette tâche :**

1. Crée `infra/.env.example` avec les variables documentées dans l'exemple : `DATABASE_URL`,
   `KILTER_API_TOKEN`, `ENVIRONMENT`, `LOG_LEVEL`. Ajoute des commentaires expliquant chaque variable.
2. Ouvre `.gitignore` à la racine du repo et ajoute une section pour l'infrastructure : ignore
   `infra/.env` et `infra/.env.*` mais pas `infra/.env.example` (via l'exception `!`). Ignore aussi
   `data/` et `*.db`.
3. Vérifie avec `git check-ignore infra/.env` que le fichier est bien ignoré, et que
   `git check-ignore infra/.env.example` ne retourne rien (fichier bien tracké).

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
