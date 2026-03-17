# TASK-0.3 - Frontend placeholder + nginx.conf

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : Aucune

## Objectif

Créer une page HTML statique de placeholder et la configuration Nginx servant de proxy `/api/*` vers le backend, en attendant le développement React du Sprint 3.

## Checklist

- [ ] Créer le répertoire `frontend/`
- [ ] Créer `frontend/index.html` avec page de placeholder
- [ ] Créer `infra/nginx.conf` avec reverse proxy `/api/` et service statique
- [ ] Vérifier que le hostname `backend` dans nginx.conf correspond au service compose

## Structure attendue

```
frontend/
└── index.html

infra/
└── nginx.conf
```

## Exemple de contenu

**frontend/index.html**
```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Kilter Route Selector</title>
  <style>
    body { font-family: sans-serif; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; background: #1a1a2e; color: #eee; }
    h1 { font-size: 2rem; }
    p { color: #aaa; }
  </style>
</head>
<body>
  <div>
    <h1>Kilter Route Selector</h1>
    <p>Application en cours de développement — Sprint 1 en cours.</p>
  </div>
</body>
</html>
```

**infra/nginx.conf**
```nginx
server {
    listen 80;

    # Proxy API calls to the FastAPI backend
    location /api/ {
        proxy_pass         http://backend:8000/;
        proxy_http_version 1.1;
        proxy_set_header   Host $host;
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # Serve static frontend files
    location / {
        root  /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

Note : `nginx.conf` utilise `backend` comme hostname upstream — ce nom correspond au service compose défini dans TASK-0.6 (`compose.dev.yml`). Il doit rester cohérent entre les deux fichiers.

## Critères de validation

- `frontend/index.html` est valide et affiche le titre "Kilter Route Selector"
- `infra/nginx.conf` est un fichier Nginx syntaxiquement valide
- Le proxy `/api/` pointe vers `http://backend:8000/`
- `try_files` assure le routing SPA (`/index.html` en fallback)

## Ressources

- [Nginx reverse proxy documentation](https://nginx.org/en/docs/http/ngx_http_proxy_module.html)
- [Nginx try_files directive](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files)
