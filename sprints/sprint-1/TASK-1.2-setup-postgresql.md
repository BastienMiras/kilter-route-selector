# TASK-1.2 - Configuration PostgreSQL

**Sprint** : 1  
**Estimation** : 1h  
**Priorité** : Haute  
**Dépendances** : Aucune

## 🎯 Objectif

Installer et configurer PostgreSQL pour le projet.

## 📋 Checklist

- [ ] Installer PostgreSQL 15+ (ou via Docker)
- [ ] Créer base de données `kilter_routes`
- [ ] Créer utilisateur dédié
- [ ] Configuration connexion (asyncpg ou SQLAlchemy)
- [ ] Tester connexion depuis Python

## 💻 Setup Docker (recommandé)

**docker-compose.yml**
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: kilter-db
    environment:
      POSTGRES_DB: kilter_routes
      POSTGRES_USER: kilter_user
      POSTGRES_PASSWORD: kilter_pass
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

Lancer :
```bash
docker-compose up -d
```

## 💻 Configuration Python

**requirements.txt** (ajouter)
```txt
asyncpg==0.29.0
sqlalchemy[asyncio]==2.0.25
```

**config.py**
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    DATABASE_URL: str = "postgresql+asyncpg://kilter_user:kilter_pass@localhost:5432/kilter_routes"
    
    class Config:
        env_file = ".env"

settings = Settings()
```

**.env**
```env
DATABASE_URL=postgresql+asyncpg://kilter_user:kilter_pass@localhost:5432/kilter_routes
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

## ✅ Critères de validation

- PostgreSQL accessible sur `localhost:5432`
- Connexion réussie depuis Python
- Base de données `kilter_routes` créée
- Pas d'erreur dans les logs

## 🧪 Test de connexion

**test_connection.py**
```python
import asyncio
import asyncpg

async def test_connection():
    conn = await asyncpg.connect(
        host='localhost',
        port=5432,
        user='kilter_user',
        password='kilter_pass',
        database='kilter_routes'
    )
    
    version = await conn.fetchval('SELECT version()')
    print(f"Connected to: {version}")
    
    await conn.close()

asyncio.run(test_connection())
```

## 🔗 Ressources

- [PostgreSQL Docker Image](https://hub.docker.com/_/postgres)
- [asyncpg Documentation](https://magicstack.github.io/asyncpg/)
- [SQLAlchemy Async](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html)
