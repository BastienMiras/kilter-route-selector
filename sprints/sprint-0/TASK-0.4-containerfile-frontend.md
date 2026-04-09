# TASK-0.4 - Containerfile.frontend

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.3

## Description

Comme pour le backend, le frontend doit être packagé dans une image conteneur pour pouvoir être
déployé de manière reproductible sur n'importe quel serveur. La différence avec le backend : ici,
pas de Python — l'image est basée sur Nginx, le serveur web qui fait déjà partie de notre stack.

**nginx:alpine** est l'image officielle Nginx basée sur Alpine Linux. Alpine est une distribution
Linux minimaliste (~5 Mo) conçue spécifiquement pour les conteneurs. L'image finale est très légère
(~25 Mo), ce qui la rend rapide à télécharger et à démarrer sur le VPS.

En Sprint 0, le Containerfile est intentionnellement simple : il copie simplement le fichier HTML et
la configuration Nginx dans l'image. En Sprint 3, quand React sera ajouté, un stage `node:20-alpine`
sera ajouté avant pour builder le bundle JavaScript avec Vite (`npm run build`), puis Nginx servira
les fichiers compilés. Ce design multi-stage évite d'installer Node.js dans l'image de production.

La configuration Nginx est copiée dans `/etc/nginx/conf.d/default.conf` — c'est l'emplacement
standard qu'Nginx lit au démarrage pour charger la configuration des serveurs virtuels.

**Comment réaliser cette tâche :**

1. Crée `infra/Containerfile.frontend` en partant de `FROM nginx:alpine AS runtime`.
2. Ajoute une ligne `COPY frontend/ /usr/share/nginx/html/` pour placer les fichiers HTML dans le
   dossier que Nginx sert par défaut.
3. Ajoute `COPY infra/nginx.conf /etc/nginx/conf.d/default.conf` pour écraser la config par défaut.
4. Expose le port 80 avec `EXPOSE 80` (documentation, pas obligatoire mais utile).
5. Build avec `podman build -f infra/Containerfile.frontend -t kilter-frontend:test .` et teste
   avec `podman run --rm -d -p 8080:80 kilter-frontend:test` puis `curl http://localhost:8080`.

## Objectif

Créer un Containerfile pour le frontend basé sur Nginx Alpine, servant le placeholder statique et la configuration nginx. Le Sprint 3 ajoutera le build Vite (stage node:20-alpine).

## Checklist

- [ ] Créer `infra/Containerfile.frontend` basé sur nginx:alpine
- [ ] Copier `frontend/` dans `/usr/share/nginx/html/`
- [ ] Copier `infra/nginx.conf` dans `/etc/nginx/conf.d/default.conf`
- [ ] Vérifier le build avec podman
- [ ] Lancer un conteneur de test et vérifier que la page est servie

## Structure attendue

```
infra/
└── Containerfile.frontend
```

## Exemple de contenu

**infra/Containerfile.frontend**
```dockerfile
# Sprint 0: static placeholder (no build step needed)
# Sprint 3 will add: FROM node:20-alpine AS builder + npm run build
FROM nginx:alpine AS runtime
COPY frontend/ /usr/share/nginx/html/
COPY infra/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

**Vérification build**
```bash
podman build -f infra/Containerfile.frontend -t kilter-frontend:test .
# Expected: Successfully tagged localhost/kilter-frontend:test
```

**Vérification runtime**
```bash
podman run --rm -d --name kilter-frontend-test -p 8080:80 kilter-frontend:test
sleep 1
curl -s http://localhost:8080 | grep "Kilter Route Selector"
# Expected: line containing "Kilter Route Selector"
podman stop kilter-frontend-test
```

## Critères de validation

- `podman build -f infra/Containerfile.frontend -t kilter-frontend:test .` se termine sans erreur
- `curl -s http://localhost:8080 | grep "Kilter Route Selector"` retourne une correspondance
- L'image est basée sur `nginx:alpine` (image légère)

## Ressources

- [Nginx Alpine image sur Docker Hub](https://hub.docker.com/_/nginx)
- [Podman build documentation](https://docs.podman.io/en/latest/markdown/podman-build.1.html)
