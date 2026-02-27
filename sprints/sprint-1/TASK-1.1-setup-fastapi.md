# TASK-1.1 - Setup serveur FastAPI

**Sprint** : 1  
**Estimation** : 2h  
**Priorité** : Haute  
**Dépendances** : Aucune

## 🎯 Objectif

Créer la structure de base du serveur FastAPI avec configuration initiale.

## 📋 Checklist

- [ ] Créer virtualenv Python 3.11+
- [ ] Installer dépendances : `fastapi`, `uvicorn`, `pydantic`
- [ ] Structure de dossiers backend
- [ ] Point d'entrée `main.py`
- [ ] Configuration CORS
- [ ] Health check endpoint (`/health`)
- [ ] Lancer serveur en dev mode

## 📁 Structure attendue

```
backend/
├── main.py
├── requirements.txt
├── config.py
├── api/
│   ├── __init__.py
│   └── routes/
│       ├── __init__.py
│       └── health.py
└── tests/
    └── test_health.py
```

## 💻 Exemple de code

**requirements.txt**
```txt
fastapi==0.109.0
uvicorn[standard]==0.27.0
pydantic==2.5.0
pydantic-settings==2.1.0
```

**main.py**
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from api.routes import health

app = FastAPI(
    title="Kilter Route Selector API",
    description="API pour sélection automatique de voies Kilter Board",
    version="0.1.0"
)

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # À restreindre en prod
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Routes
app.include_router(health.router)

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000, reload=True)
```

**api/routes/health.py**
```python
from fastapi import APIRouter

router = APIRouter(tags=["health"])

@router.get("/health")
async def health_check():
    return {"status": "ok", "service": "kilter-route-selector"}
```

## ✅ Critères de validation

- Le serveur démarre sans erreur
- `http://localhost:8000/health` retourne `{"status": "ok"}`
- Documentation Swagger accessible sur `http://localhost:8000/docs`
- Rechargement auto fonctionne (hot reload)

## 🔗 Ressources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Uvicorn Configuration](https://www.uvicorn.org/settings/)
