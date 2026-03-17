# TASK-0.6 - compose.dev.yml (local hot-reload)

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.1, TASK-0.2, TASK-0.4, TASK-0.5

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
