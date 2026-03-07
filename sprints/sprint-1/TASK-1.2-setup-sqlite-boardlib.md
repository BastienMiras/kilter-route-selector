# TASK-1.2 - Configuration SQLite via BoardLib

**Sprint** : 1
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : Aucune

## Objectif

Installer boardlib et télécharger la base SQLite complète de Kilter Board.

BoardLib est une librairie Python qui télécharge directement la base de données officielle
Kilter Board (SQLite) sans nécessiter de scraping manuel ni d'authentification JWT.

## Checklist

- [ ] Installer boardlib
- [ ] Télécharger la base Kilter (kilter.db)
- [ ] Vérifier le contenu téléchargé
- [ ] Configurer la connexion aiosqlite dans le projet

## Installation

```bash
pip install boardlib
```

**requirements.txt** (ajouter)
```txt
boardlib
aiosqlite==0.19.0
sqlalchemy[asyncio]==2.0.25
```

## Téléchargement de la base

```bash
# Créer le dossier data si besoin
mkdir -p data

# Télécharger la base Kilter Board complète
boardlib database kilter data/kilter.db
```

La commande télécharge la base SQLite officielle et la place dans `data/kilter.db`.
Aucun compte ni token requis — les données publiques sont accessibles directement.

## Vérification du téléchargement

```bash
# Lister les tables
sqlite3 data/kilter.db ".tables"

# Compter les voies
sqlite3 data/kilter.db "SELECT COUNT(*) FROM climbs;"

# Voir quelques voies
sqlite3 data/kilter.db "SELECT uuid, name, frames FROM climbs LIMIT 5;"
```

## Configuration Python

**config.py**
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str = "sqlite+aiosqlite:///./data/kilter.db"

    class Config:
        env_file = ".env"

settings = Settings()
```

**.env**
```env
DATABASE_URL=sqlite+aiosqlite:///./data/kilter.db
```

**database.py**
```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker
from config import settings

engine = create_async_engine(settings.DATABASE_URL, echo=True)

async_session = sessionmaker(
    engine, class_=AsyncSession, expire_on_commit=False
)

async def get_db():
    async with async_session() as session:
        yield session
```

## Test de connexion

**test_connection.py**
```python
import asyncio
import aiosqlite

async def test_connection():
    async with aiosqlite.connect("data/kilter.db") as db:
        async with db.execute("SELECT COUNT(*) FROM climbs") as cursor:
            count = await cursor.fetchone()
            print(f"Connected. Total climbs: {count[0]}")

asyncio.run(test_connection())
```

## Criteres de validation

- `data/kilter.db` existe et pèse plusieurs Mo
- `sqlite3 data/kilter.db ".tables"` liste des tables (climbs, etc.)
- `SELECT COUNT(*) FROM climbs` retourne plusieurs milliers de voies
- Connexion Python réussie via aiosqlite

## Mise a jour de la base

Pour resynchroniser avec la dernière version officielle :
```bash
boardlib database kilter data/kilter.db
```

boardlib gère la mise à jour incrémentale automatiquement.

## Ressources

- [BoardLib GitHub](https://github.com/lemeryfertitta/BoardLib)
- [boardlib PyPI](https://pypi.org/project/boardlib/)
- [aiosqlite Documentation](https://aiosqlite.omnilib.dev/)
- [SQLAlchemy Async](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)
