# TASK-1.3 - Schéma de base de données

**Sprint** : 1  
**Estimation** : 2h  
**Priorité** : Haute  
**Dépendances** : TASK-1.2

## 🎯 Objectif

Créer le schéma de base de données avec toutes les tables nécessaires.

## 📋 Checklist

- [ ] Créer modèles SQLAlchemy
- [ ] Table `climbs` (voies)
- [ ] Table `holds` (prises)
- [ ] Table `climb_metrics` (métriques calculées)
- [ ] Table `user_sessions` (historique - optionnel sprint 1)
- [ ] Migrations avec Alembic
- [ ] Script d'initialisation

## 💻 Modèles SQLAlchemy

**models.py**
```python
from sqlalchemy import Column, Integer, String, Boolean, DECIMAL, Text, TIMESTAMP, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.sql import func

Base = declarative_base()

class Climb(Base):
    __tablename__ = "climbs"
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    kilter_id = Column(Integer, unique=True, nullable=False, index=True)
    name = Column(String(255))
    setter = Column(String(100), index=True)
    setter_id = Column(Integer)
    grade = Column(String(10), index=True)
    angle = Column(Integer)
    layout = Column(Text, nullable=False)
    ascents = Column(Integer, default=0)
    quality_avg = Column(DECIMAL(2, 1))
    is_public = Column(Boolean, default=True)
    created_at = Column(TIMESTAMP, server_default=func.now())
    updated_at = Column(TIMESTAMP, server_default=func.now(), onupdate=func.now())
    last_scraped_at = Column(TIMESTAMP)

class Hold(Base):
    __tablename__ = "holds"
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    climb_id = Column(Integer, ForeignKey("climbs.id", ondelete="CASCADE"), nullable=False, index=True)
    position = Column(Integer, nullable=False)
    x = Column(Integer)
    y = Column(Integer)
    radius = Column(Integer)
    hold_order = Column(Integer)  # Ordre dans la voie (1 = start, n = finish)
    is_start = Column(Boolean, default=False)
    is_finish = Column(Boolean, default=False)
    is_foot_only = Column(Boolean, default=False)

class ClimbMetric(Base):
    __tablename__ = "climb_metrics"
    
    climb_id = Column(Integer, ForeignKey("climbs.id", ondelete="CASCADE"), primary_key=True)
    move_count = Column(Integer)
    avg_distance = Column(DECIMAL(5, 2))
    max_reach = Column(DECIMAL(5, 2))
    vertical_range = Column(Integer)
    horizontal_range = Column(Integer)
    symmetry_score = Column(DECIMAL(3, 2))
    hold_density = Column(DECIMAL(5, 4))
    style_dynamic_score = Column(DECIMAL(5, 2))
    style_technical_score = Column(DECIMAL(5, 2))
    style_endurance_score = Column(DECIMAL(5, 2))
    computed_at = Column(TIMESTAMP, server_default=func.now())
```

## 💻 Script SQL brut (alternative)

**schema.sql**
```sql
-- Table des voies
CREATE TABLE climbs (
    id SERIAL PRIMARY KEY,
    kilter_id INT UNIQUE NOT NULL,
    name VARCHAR(255),
    setter VARCHAR(100),
    setter_id INT,
    grade VARCHAR(10),
    angle INT,
    layout TEXT NOT NULL,
    ascents INT DEFAULT 0,
    quality_avg DECIMAL(2,1),
    is_public BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    last_scraped_at TIMESTAMP
);

CREATE INDEX idx_climbs_grade ON climbs(grade);
CREATE INDEX idx_climbs_ascents ON climbs(ascents);
CREATE INDEX idx_climbs_setter ON climbs(setter);
CREATE INDEX idx_climbs_kilter_id ON climbs(kilter_id);

-- Table des prises
CREATE TABLE holds (
    id SERIAL PRIMARY KEY,
    climb_id INT REFERENCES climbs(id) ON DELETE CASCADE,
    position INT NOT NULL,
    x INT,
    y INT,
    radius INT,
    hold_order INT,
    is_start BOOLEAN DEFAULT false,
    is_finish BOOLEAN DEFAULT false,
    is_foot_only BOOLEAN DEFAULT false
);

CREATE INDEX idx_holds_climb_id ON holds(climb_id);

-- Table des métriques
CREATE TABLE climb_metrics (
    climb_id INT PRIMARY KEY REFERENCES climbs(id) ON DELETE CASCADE,
    move_count INT,
    avg_distance DECIMAL(5,2),
    max_reach DECIMAL(5,2),
    vertical_range INT,
    horizontal_range INT,
    symmetry_score DECIMAL(3,2),
    hold_density DECIMAL(5,4),
    style_dynamic_score DECIMAL(5,2),
    style_technical_score DECIMAL(5,2),
    style_endurance_score DECIMAL(5,2),
    computed_at TIMESTAMP DEFAULT NOW()
);
```

## 💻 Migrations avec Alembic

**Installation**
```bash
pip install alembic
alembic init migrations
```

**migrations/env.py** (configurer)
```python
from models import Base
target_metadata = Base.metadata
```

**Créer migration**
```bash
alembic revision --autogenerate -m "Initial schema"
alembic upgrade head
```

## 💻 Script d'initialisation

**init_db.py**
```python
import asyncio
from sqlalchemy.ext.asyncio import create_async_engine
from models import Base
from config import settings

async def init_db():
    engine = create_async_engine(settings.DATABASE_URL, echo=True)
    
    async with engine.begin() as conn:
        # Drop all tables (dev only!)
        await conn.run_sync(Base.metadata.drop_all)
        # Create all tables
        await conn.run_sync(Base.metadata.create_all)
    
    await engine.dispose()
    print("✅ Database initialized successfully!")

if __name__ == "__main__":
    asyncio.run(init_db())
```

## ✅ Critères de validation

- Toutes les tables créées sans erreur
- Indexes créés correctement
- Foreign keys fonctionnent (cascade delete)
- `\dt` dans psql liste les 3 tables
- Script d'init peut être relancé sans erreur

## 🧪 Test du schéma

```python
import asyncio
from database import async_session
from models import Climb

async def test_insert():
    async with async_session() as session:
        climb = Climb(
            kilter_id=1,
            name="Test Route",
            grade="V5",
            layout="p100r15p200r15"
        )
        session.add(climb)
        await session.commit()
        print(f"✅ Inserted climb ID: {climb.id}")

asyncio.run(test_insert())
```

## 🔗 Ressources

- [SQLAlchemy Models](https://docs.sqlalchemy.org/en/20/orm/mapping_styles.html)
- [Alembic Tutorial](https://alembic.sqlalchemy.org/en/latest/tutorial.html)
