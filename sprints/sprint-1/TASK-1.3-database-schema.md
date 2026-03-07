# TASK-1.3 - Schéma de base de données

**Sprint** : 1
**Estimation** : 2h
**Priorité** : Haute
**Dépendances** : TASK-1.2

## Objectif

Explorer le schéma boardlib existant et ajouter la table `climb_metrics` pour
les métriques calculées.

La base `kilter.db` fournie par boardlib contient déjà une table `climbs` avec
toutes les voies. L'objectif n'est pas de recréer ce schéma, mais de l'explorer
puis d'y ajouter uniquement la table de métriques.

## Checklist

- [ ] Explorer le schéma boardlib (tables, colonnes)
- [ ] Identifier les colonnes utiles dans `climbs`
- [ ] Créer la table `climb_metrics` dans le SQLite existant
- [ ] Créer les modèles SQLAlchemy correspondants
- [ ] Script d'initialisation des métriques

## Etape 1 : Explorer le schema boardlib

```bash
# Lister toutes les tables
sqlite3 data/kilter.db ".tables"

# Schema complet de la table climbs
sqlite3 data/kilter.db ".schema climbs"

# Colonnes disponibles
sqlite3 data/kilter.db "PRAGMA table_info(climbs);"

# Exemple de voie complète
sqlite3 data/kilter.db "SELECT * FROM climbs LIMIT 1;"
```

**Colonnes clés attendues dans `climbs`** (à confirmer par exploration) :
- `uuid` : identifiant unique de la voie
- `name` : nom de la voie
- `setter_username` : ouvreur
- `difficulty` : grade (valeur numérique)
- `frames` : layout des prises (`p1083r15p1117r15...`)
- `ascensionist_count` : nombre d'ascensions
- `quality_average` : note moyenne communautaire
- `is_listed` : voie publique
- `created_at` : date de création

## Etape 2 : Ajouter la table climb_metrics

La table `climb_metrics` est ajoutée au SQLite existant par un script Python
utilisant la stdlib (sqlite3). Pas de migration Alembic nécessaire pour le MVP.

**init_metrics_table.py**
```python
import sqlite3

def init_metrics_table(db_path: str = "data/kilter.db"):
    conn = sqlite3.connect(db_path)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS climb_metrics (
            climb_uuid TEXT PRIMARY KEY REFERENCES climbs(uuid) ON DELETE CASCADE,
            move_count INTEGER,
            avg_distance REAL,
            max_reach REAL,
            vertical_range INTEGER,
            horizontal_range INTEGER,
            symmetry_score REAL,
            hold_density REAL,
            style_dynamic_score REAL,
            style_technical_score REAL,
            style_endurance_score REAL,
            computed_at TEXT DEFAULT (datetime('now'))
        )
    """)
    conn.commit()
    conn.close()
    print("Table climb_metrics créée (ou déjà existante).")

if __name__ == "__main__":
    init_metrics_table()
```

## Etape 3 : Modèles SQLAlchemy

**models.py**
```python
from sqlalchemy import Column, Integer, String, Boolean, Float, Text, ForeignKey
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Climb(Base):
    """Voie boardlib — table existante, lecture seule."""
    __tablename__ = "climbs"

    uuid = Column(String, primary_key=True)
    name = Column(String)
    setter_username = Column(String, index=True)
    difficulty = Column(Integer, index=True)
    frames = Column(Text)           # Layout des prises
    ascensionist_count = Column(Integer, default=0)
    quality_average = Column(Float)
    is_listed = Column(Boolean, default=True)
    created_at = Column(String)

class ClimbMetric(Base):
    """Métriques calculées — table ajoutée par nous."""
    __tablename__ = "climb_metrics"

    climb_uuid = Column(String, ForeignKey("climbs.uuid", ondelete="CASCADE"), primary_key=True)
    move_count = Column(Integer)
    avg_distance = Column(Float)
    max_reach = Column(Float)
    vertical_range = Column(Integer)
    horizontal_range = Column(Integer)
    symmetry_score = Column(Float)
    hold_density = Column(Float)
    style_dynamic_score = Column(Float)
    style_technical_score = Column(Float)
    style_endurance_score = Column(Float)
    computed_at = Column(String)
```

## Etape 4 : Script d'initialisation complet

**init_db.py**
```python
import sqlite3
import asyncio
import aiosqlite

DB_PATH = "data/kilter.db"

def init_metrics_table():
    """Ajoute la table climb_metrics si elle n'existe pas."""
    with sqlite3.connect(DB_PATH) as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS climb_metrics (
                climb_uuid TEXT PRIMARY KEY REFERENCES climbs(uuid) ON DELETE CASCADE,
                move_count INTEGER,
                avg_distance REAL,
                max_reach REAL,
                vertical_range INTEGER,
                horizontal_range INTEGER,
                symmetry_score REAL,
                hold_density REAL,
                style_dynamic_score REAL,
                style_technical_score REAL,
                style_endurance_score REAL,
                computed_at TEXT DEFAULT (datetime('now'))
            )
        """)
        conn.commit()

async def verify_schema():
    """Vérifie que le schéma est correct."""
    async with aiosqlite.connect(DB_PATH) as db:
        async with db.execute("SELECT COUNT(*) FROM climbs WHERE is_listed = 1") as cur:
            count = (await cur.fetchone())[0]
        async with db.execute("SELECT COUNT(*) FROM climb_metrics") as cur:
            metrics_count = (await cur.fetchone())[0]

    print(f"Voies publiques : {count}")
    print(f"Métriques calculées : {metrics_count}")

if __name__ == "__main__":
    init_metrics_table()
    print("Table climb_metrics initialisée.")
    asyncio.run(verify_schema())
```

## Criteres de validation

- `.schema climb_metrics` affiche la table créée
- Les colonnes correspondent aux métriques prévues
- `SELECT COUNT(*) FROM climbs` retourne plusieurs milliers de voies
- Script d'init peut être relancé sans erreur (`CREATE TABLE IF NOT EXISTS`)
- Modèles SQLAlchemy importables sans erreur

## Ressources

- [aiosqlite Documentation](https://aiosqlite.omnilib.dev/)
- [SQLAlchemy Core](https://docs.sqlalchemy.org/en/20/core/)
- [BoardLib schema (Climbdex)](https://github.com/lemeryfertitta/Climbdex)
