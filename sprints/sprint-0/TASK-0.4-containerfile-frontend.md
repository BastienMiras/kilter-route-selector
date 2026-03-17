# TASK-0.4 - Containerfile.frontend

**Sprint** : 0
**Estimation** : 1h
**Priorité** : Haute
**Dépendances** : TASK-0.3

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
