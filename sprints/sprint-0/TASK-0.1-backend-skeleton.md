# TASK-0.1 - Backend skeleton (main.py + requirements.txt)

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : Aucune

## Description

Ce projet est une application web qui recommande des voies d'escalade sur Kilter Board. Avant de
construire quoi que ce soit d'autre — conteneurs, déploiement, interface graphique — l'application
backend doit exister et fonctionner, même sous sa forme la plus basique.

**FastAPI** est un framework Python moderne pour créer des API web. Il est rapide à écrire et génère
automatiquement une documentation interactive accessible dans un navigateur (à `/docs`). **Uvicorn**
est le serveur qui exécute cette application Python et gère les requêtes HTTP entrantes.

Un endpoint `/health` est une convention universelle dans les applications web professionnelles. C'est
une route très simple qui répond `{"status": "ok"}`. Elle sert à vérifier que le serveur tourne
correctement — les outils de déploiement, les conteneurs et les pipelines CI l'appellent tous
automatiquement pour savoir si l'application est vivante. Sans ce point d'entrée, il serait impossible
de tester automatiquement que le service démarre bien après un déploiement.

Le fichier `requirements.txt` est le moyen standard de déclarer les dépendances Python d'un projet.
Il permet à n'importe qui — ou à n'importe quel serveur — de recréer exactement le même environnement
avec une seule commande : `pip install -r requirements.txt`. Les versions sont fixées (`==`) pour
garantir que le code se comportera de la même façon partout et dans le temps.

**Comment réaliser cette tâche :**

1. Crée le répertoire `backend/` et le sous-répertoire `backend/tests/`. Un fichier `.gitkeep` vide
   permet à git de versionner ce dossier même s'il est vide — git ne suit pas les dossiers seuls.
2. Écris `backend/requirements.txt` avec deux lignes : `fastapi==0.115.0` et `uvicorn[standard]==0.32.0`.
3. Écris `backend/main.py` : importe `FastAPI`, crée une instance `app = FastAPI(...)`, puis décore
   une fonction avec `@app.get("/health")` qui retourne le dictionnaire `{"status": "ok"}`.
4. Teste localement en créant un virtualenv temporaire, en installant les dépendances et en lançant
   le serveur avec `uvicorn backend.main:app --port 8000`, puis vérifie avec `curl`.

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
