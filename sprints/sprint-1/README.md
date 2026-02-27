# Sprint 1 - Backend Fondation

**Durée estimée** : 1 semaine  
**Objectif** : Avoir 1000+ voies en DB avec métriques calculées

## 🎯 Milestone

API fonctionnelle qui scrape, parse et calcule les métriques pour les voies Kilter Board.

## 📋 Tâches

### 1. Setup Infrastructure
- [ ] [TASK-1.1] Setup serveur FastAPI
- [ ] [TASK-1.2] Configuration PostgreSQL
- [ ] [TASK-1.3] Schéma de base de données
- [ ] [TASK-1.4] Configuration environnement (Docker/venv)

### 2. Reverse Engineering API
- [ ] [TASK-2.1] Tester authentification Kilter Board API
- [ ] [TASK-2.2] Explorer endpoints disponibles
- [ ] [TASK-2.3] Documenter structure des réponses JSON
- [ ] [TASK-2.4] Implémenter client API Python

### 3. Scraping
- [ ] [TASK-3.1] Worker de scraping initial
- [ ] [TASK-3.2] Rate limiting (1 req/sec)
- [ ] [TASK-3.3] Gestion erreurs et retries
- [ ] [TASK-3.4] Scraper 1000+ voies

### 4. Parsing Layouts
- [ ] [TASK-4.1] Parser format `p{position}r{radius}`
- [ ] [TASK-4.2] Mapper positions → coordonnées (x, y)
- [ ] [TASK-4.3] Stocker dans table `holds`
- [ ] [TASK-4.4] Identifier start/finish/foot-only

### 5. Calcul Métriques
- [ ] [TASK-5.1] Distance moyenne entre prises
- [ ] [TASK-5.2] Portée maximale (max reach)
- [ ] [TASK-5.3] Nombre de mouvements
- [ ] [TASK-5.4] Distribution verticale/horizontale
- [ ] [TASK-5.5] Symétrie gauche/droite
- [ ] [TASK-5.6] Densité de prises
- [ ] [TASK-5.7] Stocker dans table `climb_metrics`

### 6. API Endpoints
- [ ] [TASK-6.1] Endpoint GET `/api/climbs` (filtrage basique)
- [ ] [TASK-6.2] Filtres : grade_min, grade_max, limit
- [ ] [TASK-6.3] Endpoint GET `/api/climbs/{id}` (détail)
- [ ] [TASK-6.4] Documentation OpenAPI/Swagger

### 7. Tests & Validation
- [ ] [TASK-7.1] Tests unitaires (parsing, métriques)
- [ ] [TASK-7.2] Tests d'intégration (API endpoints)
- [ ] [TASK-7.3] Validation cohérence des données
- [ ] [TASK-7.4] Logging et monitoring

## 📊 Critères d'acceptation

- ✅ Au moins 1000 voies dans la base de données
- ✅ Toutes les métriques calculées pour chaque voie
- ✅ API répond en < 500ms pour `/api/climbs`
- ✅ Documentation Swagger accessible
- ✅ Tests passent à 100%

## 🛠️ Stack & Outils

- **Backend** : Python 3.11+, FastAPI
- **DB** : PostgreSQL 15+
- **ORM** : SQLAlchemy ou asyncpg
- **Tests** : pytest, pytest-asyncio
- **Docker** : docker-compose pour dev
- **Logs** : structlog

## 📝 Notes

- Commencer par un petit dataset (100 voies) pour valider la pipeline complète
- Documenter les anomalies dans les données Kilter
- Prévoir un script de reset DB pour faciliter les itérations
