# TASK-0.2 - Containerfile.backend multi-stage

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.1

## Description

Un **conteneur** est un environnement d'exécution isolé et portable : il embarque l'application et
toutes ses dépendances (Python, bibliothèques, configuration) dans une image autonome. N'importe quel
serveur disposant de Podman ou Docker peut exécuter cette image et obtenir exactement le même
comportement, quelle que soit sa configuration. C'est le cœur du principe « build once, run anywhere ».

**Podman** est l'alternative sans daemon (sans processus root permanent) à Docker. Il fonctionne en
mode rootless — un avantage de sécurité important sur un VPS partagé. Les commandes sont identiques
à Docker : `podman build`, `podman run`, `podman push`.

Un **Containerfile multi-stage** divise la construction de l'image en plusieurs étapes :
- Le stage `builder` installe les dépendances Python dans un dossier dédié (`/install`). Ce stage
  utilise l'image complète `python:3.12` qui contient tous les outils de compilation nécessaires.
- Le stage `runtime` repart d'une image légère (`python:3.12-slim`) et copie uniquement les fichiers
  compilés depuis le stage précédent. Le résultat est une image finale beaucoup plus petite (~150 Mo
  au lieu de ~900 Mo), ce qui accélère les pull et réduit la surface d'attaque.

La variable `PYTHONUNBUFFERED=1` force Python à écrire ses logs en temps réel (sans buffer), ce qui
est essentiel pour voir les logs dans un conteneur.

**Comment réaliser cette tâche :**

1. Crée le dossier `infra/` s'il n'existe pas encore.
2. Écris `infra/Containerfile.backend` avec les deux stages décrits dans l'exemple.
3. Build l'image avec `podman build -f infra/Containerfile.backend -t kilter-backend:test .`
   (le `.` à la fin est le contexte de build — le répertoire depuis lequel les `COPY` sont résolus).
4. Lance un conteneur de test en arrière-plan avec `podman run --rm -d -p 8000:8000 kilter-backend:test`
   et vérifie que `/health` répond correctement. Arrête le conteneur ensuite.

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
