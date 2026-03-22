# TASK-0.6 - compose.dev.yml (local hot-reload)

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.1, TASK-0.2, TASK-0.4, TASK-0.5

## Description

**podman-compose** (ou docker-compose) est un outil qui permet de décrire et lancer plusieurs
conteneurs en même temps via un fichier YAML. Sans lui, il faudrait lancer chaque conteneur
manuellement avec une longue commande `podman run`. Le fichier compose définit les services, leurs
options, comment ils communiquent et quels ports ils exposent.

En développement local, on a besoin d'un comportement spécifique : **le hot-reload**. C'est la
capacité du serveur à détecter automatiquement les modifications du code source et à se recharger
sans qu'on ait besoin de stopper et relancer le conteneur. Uvicorn supporte cela avec l'option
`--reload`. Pour que ça fonctionne, le code source du dossier `backend/` doit être directement
accessible depuis l'intérieur du conteneur — c'est ce que fait le **volume mount** (`../backend:/app`) :
il monte le dossier local dans le conteneur. Chaque sauvegarde de fichier est immédiatement visible.

Le volume SQLite (`../data:/app/data`) fonctionne de la même façon : la base de données persiste sur
le disque local entre les arrêts et démarrages du conteneur.

En développement, les ports sont exposés directement : backend sur 8000 et frontend sur 5173. Le
réseau interne `kilter-dev-net` permet aux conteneurs de se parler par leur nom de service (`backend`,
`frontend`) sans passer par l'hôte.

**Comment réaliser cette tâche :**

1. Crée `infra/compose.dev.yml` avec les deux services `backend` et `frontend`.
2. Pour le service `backend` : utilise `build:` pour builder depuis le Containerfile, monte les
   volumes source et data, surcharge la commande avec `--reload`, expose le port 8000.
3. Pour le service `frontend` : utilise aussi `build:`, expose le port 5173 (mappé sur le 80 interne),
   déclare `depends_on: backend` pour démarrer dans le bon ordre.
4. Crée le dossier `data/` avec un `.gitkeep` pour que git versionne le dossier vide.
5. Lance avec `podman-compose -f infra/compose.dev.yml up -d` et vérifie les deux ports.

## Objectif

Créer le fichier compose pour le développement local avec montage de volumes source pour le hot-reload uvicorn, exposant le backend sur :8000 et le frontend sur :5173.

## Checklist

- [ ] Créer `infra/compose.dev.yml` avec les services backend et frontend
- [ ] Configurer le volume de montage source backend (`../backend:/app`)
- [ ] Configurer le volume de données SQLite (`../data:/app/data`)
- [ ] Configurer la commande uvicorn avec `--reload`
- [ ] Créer le répertoire `data/` avec `.gitkeep`
- [ ] Vérifier que le stack démarre et répond

## Structure attendue

```
infra/
└── compose.dev.yml
data/
└── .gitkeep
```

## Exemple de contenu

**infra/compose.dev.yml**
```yaml
# Local development stack
# Usage: podman-compose -f infra/compose.dev.yml up
# Backend source is mounted for hot-reload.

services:
  backend:
    build:
      context: ..
      dockerfile: infra/Containerfile.backend
    volumes:
      - ../backend:/app        # hot-reload: source mounted
      - ../data:/app/data      # SQLite volume
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=sqlite:///./data/kilter.db
      - ENVIRONMENT=dev
      - LOG_LEVEL=debug
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload
    restart: "no"

  frontend:
    build:
      context: ..
      dockerfile: infra/Containerfile.frontend
    ports:
      - "5173:80"
    depends_on:
      - backend
    restart: "no"

networks:
  default:
    name: kilter-dev-net
```

**Vérification**
```bash
podman-compose -f infra/compose.dev.yml up -d
sleep 3
curl http://localhost:8000/health
# Expected: {"status":"ok"}
curl -s http://localhost:5173 | grep "Kilter"
# Expected: line containing "Kilter"
podman-compose -f infra/compose.dev.yml down
```

## Critères de validation

- `curl http://localhost:8000/health` retourne `{"status":"ok"}` après `up -d`
- `curl -s http://localhost:5173` retourne la page placeholder
- Le hot-reload fonctionne : modifier `backend/main.py` rechargea le serveur sans restart
- `podman-compose -f infra/compose.dev.yml down` arrête le stack proprement

## Ressources

- [podman-compose documentation](https://github.com/containers/podman-compose)
- [Uvicorn hot-reload](https://www.uvicorn.org/settings/#development)
