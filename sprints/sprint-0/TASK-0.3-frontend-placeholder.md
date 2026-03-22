# TASK-0.3 - Frontend placeholder + nginx.conf

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : Aucune

## Description

L'application finale aura une interface React complexe (Sprint 3), mais React n'existe pas encore.
Pour que l'infrastructure soit fonctionnelle dès le Sprint 0, il faut un frontend minimal : une simple
page HTML statique qui dit « en cours de développement ». Cette page sera remplacée par React plus tard.

**Nginx** est un serveur web très répandu et très performant. Dans notre architecture, il joue deux
rôles simultanément :
- **Servir les fichiers statiques** : il lit les fichiers HTML/CSS/JS depuis un dossier local et les
  envoie aux navigateurs clients.
- **Reverse proxy** : quand le navigateur appelle `/api/quelquechose`, Nginx intercepte la requête
  et la transmet au backend FastAPI (qui tourne sur le port 8000) en interne, puis renvoie la réponse
  au client. Le client ne communique jamais directement avec le backend — tout passe par Nginx.

Ce design est standard : Nginx sur le port 80/443 face à internet, et le backend sur un port interne
non exposé. Cela simplifie la sécurité et permet de faire du load balancing si besoin.

La directive `try_files $uri $uri/ /index.html` est cruciale pour les SPA (Single Page Applications
comme React) : si l'URL demandée (`/session/42` par exemple) ne correspond à aucun fichier sur le
disque, Nginx sert quand même `index.html` et laisse React gérer le routage côté client.

**Comment réaliser cette tâche :**

1. Crée `frontend/index.html` : une page HTML simple avec un titre et un message indiquant que
   l'application est en cours de développement. Un peu de CSS inline suffit pour la rendre présentable.
2. Crée `infra/nginx.conf` : configure un server block qui écoute sur le port 80, avec un bloc
   `location /api/` qui proxifie vers `http://backend:8000/` et un bloc `location /` qui sert les
   fichiers statiques avec `try_files`.
3. Le nom `backend` dans la configuration Nginx est le nom du service défini dans le compose (TASK-0.6).
   Dans un réseau Docker/Podman, les conteneurs se trouvent par leur nom de service.

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
