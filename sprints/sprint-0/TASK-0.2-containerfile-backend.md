# TASK-0.2 - Containerfile.backend multi-stage

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.1

## Objectif

Créer un Containerfile multi-stage pour le backend Python/FastAPI, produisant une image slim prête pour la CI et le déploiement VPS.

## Checklist

- [ ] Créer le répertoire `infra/` si absent
- [ ] Créer `infra/Containerfile.backend` avec stage builder et stage runtime
- [ ] Vérifier le build avec podman
- [ ] Lancer un conteneur de test et vérifier `/health`

## Structure attendue

```
infra/
└── Containerfile.backend
```

## Exemple de contenu

**infra/Containerfile.backend**
```dockerfile
# --- Builder stage ---
FROM python:3.12 AS builder
WORKDIR /app
COPY backend/requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# --- Runtime stage ---
FROM python:3.12-slim AS runtime
WORKDIR /app
COPY --from=builder /install /usr/local
COPY backend/ .
ENV PYTHONUNBUFFERED=1
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Vérification build**
```bash
podman build -f infra/Containerfile.backend -t kilter-backend:test .
# Expected: Successfully tagged localhost/kilter-backend:test
```

**Vérification runtime**
```bash
podman run --rm -d --name kilter-backend-test -p 8000:8000 kilter-backend:test
sleep 2
curl http://localhost:8000/health
# Expected: {"status":"ok"}
podman stop kilter-backend-test
```

## Critères de validation

- `podman build -f infra/Containerfile.backend -t kilter-backend:test .` se termine sans erreur
- `curl http://localhost:8000/health` retourne `{"status":"ok"}` depuis le conteneur
- L'image utilise bien deux stages (builder + runtime slim)

## Ressources

- [Podman build documentation](https://docs.podman.io/en/latest/markdown/podman-build.1.html)
- [Python multi-stage builds best practices](https://docs.docker.com/develop/dev-best-practices/)
