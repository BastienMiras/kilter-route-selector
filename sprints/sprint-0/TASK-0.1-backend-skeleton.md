# TASK-0.1 - Backend skeleton (main.py + requirements.txt)

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : Aucune

## Objectif

Créer le squelette minimal du backend FastAPI avec un endpoint `/health` fonctionnel, servant de base pour la containerisation et la CI.

## Checklist

- [ ] Créer le répertoire `backend/tests/`
- [ ] Créer `backend/tests/.gitkeep`
- [ ] Créer `backend/requirements.txt` avec fastapi et uvicorn
- [ ] Créer `backend/main.py` avec l'app FastAPI et l'endpoint `/health`
- [ ] Vérifier que le serveur démarre localement

## Structure attendue

```
backend/
├── main.py
├── requirements.txt
└── tests/
    └── .gitkeep
```

## Exemple de contenu

**backend/requirements.txt**
```text
fastapi==0.115.0
uvicorn[standard]==0.32.0
```

**backend/main.py**
```python
from fastapi import FastAPI

app = FastAPI(title="Kilter Route Selector")


@app.get("/health")
def health():
    return {"status": "ok"}
```

**Vérification locale**
```bash
python -m venv /tmp/kilter-venv
source /tmp/kilter-venv/bin/activate
pip install -r backend/requirements.txt
uvicorn backend.main:app --port 8000 &
curl http://localhost:8000/health
# Expected: {"status":"ok"}
kill %1
deactivate
```

## Critères de validation

- `curl http://localhost:8000/health` retourne `{"status":"ok"}`
- Le serveur démarre sans erreur avec `uvicorn backend.main:app --port 8000`
- La structure `backend/tests/` existe avec `.gitkeep`

## Ressources

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Uvicorn Configuration](https://www.uvicorn.org/settings/)
