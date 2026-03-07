# Kilter Route Selector

**Sélection automatique de voies d'escalade pour Kilter Board basée sur des critères personnalisés**

## 🎯 Concept

Un système qui agrège les voies de la communauté Kilter Board et recommande/génère des sessions d'entraînement en fonction de critères comme la difficulté, le type de mouvements, l'endurance, etc.

## Source de données

Les voies proviennent de la base officielle Kilter Board via [BoardLib](https://github.com/lemeryfertitta/BoardLib) :

```bash
pip install boardlib
boardlib database kilter data/kilter.db
```

BoardLib télécharge la base SQLite complète (toutes les voies publiques avec métadonnées)
sans nécessiter de compte ni de token. La commande peut être relancée pour synchroniser
les nouvelles voies.

## Architecture

```
boardlib sync
    ↓
kilter.db (SQLite)          ← données officielles Kilter Board
    ↓
FastAPI + aiosqlite          ← métriques calculées (climb_metrics)
                                 container backend (:8000, interne)
    ↓
Nginx (reverse proxy)        ← container frontend (:80)
    ↓
React + TypeScript (Vite)   ← SPA responsive (Android / iPhone / PC)
```

## 📋 Fonctionnalités

### Core Features (MVP)

- **Filtrage par difficulté** : Sélection par grade (V0-V17)
- **Filtrage par style** :
  - Dynamic (grands mouvements, dynos)
  - Technical (prises fines, séquences complexes)
  - Endurance (longues voies)
  - Balanced (mixte)
- **Session Builder** : Génération automatique de sessions de 10+ voies avec diversité
- **Métriques calculées** :
  - Distance moyenne entre prises
  - Portée maximale (détection dynos)
  - Nombre de mouvements
  - Symétrie gauche/droite
  - Densité de prises

### Features avancées (v2+)

- **Recommandations communautaires** : Voies populaires, trending, hidden gems
- **Visualisation** : Heatmap des prises, zone map sur le mur
- **Tracking de progression** : Historique des sessions
- **Setter analytics** : Top ouvreurs par style
- **Recommandations personnalisées** : ML-based similarity

## 🚀 Roadmap

### Sprint 1 (1 semaine) - Backend fondation
- [ ] Setup serveur (FastAPI + aiosqlite)
- [ ] Téléchargement base Kilter via boardlib
- [ ] Explorer schema SQLite boardlib
- [ ] Parser layouts → coordonnées (champ `frames`)
- [ ] Calcul métriques de base
- [ ] Endpoint `/api/climbs` avec filtres
- **Milestone** : API fonctionnelle sur la base boardlib avec métriques

### Sprint 2 (1 semaine) - Session Builder
- [ ] Algorithme de scoring par style
- [ ] Session builder avec diversité (zones, setters)
- [ ] Endpoint `/api/sessions/generate`
- [ ] Tests avec différents profils utilisateur
- **Milestone** : Sessions de qualité générées

### Sprint 3 (1 semaine) - Client MVP (React + Nginx)
- [ ] Setup React + TypeScript + Vite + configuration Nginx
- [ ] Dockerfiles + podman-compose.yml (2 containers)
- [ ] API client TanStack Query (climbs, sessions)
- [ ] Pages : Home (filtres), Session (liste), Detail (métriques)
- [ ] Composants : FilterPanel, RouteCard, HeatmapCanvas
- **Milestone** : App accessible sur http://localhost, responsive mobile/desktop

### Sprint 4 (1 semaine) - Visualisation
- [ ] Heatmap des prises sur canvas
- [ ] Graphiques de métriques
- [ ] Zone map du mur
- **Milestone** : UX riche et informative

### Sprint 5+ - Advanced Features
- [ ] User accounts & progression tracking
- [ ] ML-based recommendations
- [ ] Community stats (top setters, trends)
- [ ] Session export/import
- [ ] Offline mode complet
- [ ] Intégration directe Kilter Board app (si API officielle)

## 📊 Critères de sélection

### 1. Difficulté
- Grade V-scale/Font
- Filtrage par fourchette (ex: V4-V6)

### 2. Type de mouvements
- **Statique vs dynamique** : Analyse espacement entre prises
- **Compression** : Prises opposées
- **Coordination** : Séquences complexes
- **Endurance** : Nombre de mouvements

### 3. Zones du mur
- Division en grille 3x3 (top/middle/bottom × left/center/right)
- Évite répétition des mêmes zones dans une session

### 4. Intensité physique
- **Core** : Voies en dévers
- **Force de doigts** : Petites prises
- **Portées** : Distance entre prises
- **Équilibre** : Prises espacées latéralement

### 5. Méta-données communautaires
- Popularité (nombre d'ascensions)
- Notes/qualité moyenne
- Setter (ouvreur)
- Fraîcheur (nouvelles voies)

## 🛠️ Stack technique

### Backend
- **Langage** : Python 3.11+
- **Framework** : FastAPI
- **Database** : SQLite via boardlib (`kilter.db`)
- **Driver async** : aiosqlite
- **Hosting** : Railway / Fly.io (MVP) → VPS (production)

### Client
- **Framework** : React 18 + TypeScript + Vite
- **Platforms** : Tous navigateurs (Android, iPhone, PC)
- **Serving** : Nginx (static files + reverse proxy /api/*)
- **Data fetching** : TanStack Query
- **Navigation** : React Router v6
- **Visualisation** : Canvas API (heatmap), Recharts (graphiques)

### Orchestration
- **Containers** : Podman + podman-compose
- **Services** : frontend (Nginx), backend (FastAPI/Uvicorn)

### DevOps
- **CI/CD** : GitHub Actions
- **Monitoring** : Sentry (errors), Prometheus (metrics)
- **Logs** : Loki / CloudWatch

## Schéma de données

La table `climbs` est fournie par boardlib (schema officiel Kilter Board).
Colonnes clés : `uuid`, `name`, `setter_username`, `difficulty`, `frames`,
`ascensionist_count`, `quality_average`, `is_listed`.

### Table `climb_metrics` (ajoutée par le projet)
```sql
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
);
```

## Données Kilter Board

Les données proviennent de boardlib qui synchronise la base officielle Kilter Board.
Aucune authentification manuelle ni reverse engineering requis.

```bash
# Téléchargement initial
boardlib database kilter data/kilter.db

# Mise à jour (sync incrémental)
boardlib database kilter data/kilter.db
```

## 📖 Algorithmes clés

### Calcul de distance entre prises
```python
def avg_distance(holds):
    distances = []
    for i in range(len(holds) - 1):
        x1, y1 = holds[i]['coords']
        x2, y2 = holds[i+1]['coords']
        dist = sqrt((x2-x1)**2 + (y2-y1)**2)
        distances.append(dist)
    return sum(distances) / len(distances)
```

### Scoring par style
```python
# Style Dynamic
score = (
    metrics['avg_distance'] * 2 +
    metrics['max_reach'] * 3 +
    -metrics['density'] * 1
)

# Style Technical
score = (
    metrics['density'] * 3 +
    -metrics['avg_distance'] * 1 +
    metrics['move_count'] * 2
)

# Style Endurance
score = (
    metrics['move_count'] * 3 +
    metrics['vertical_range'] * 2
)
```

### Session Builder avec diversité
```python
async def build_session(count=10, grade_min="V4", grade_max="V6", style="balanced"):
    candidates = await fetch_filtered_climbs(grade_min, grade_max)
    
    # Score selon style
    for climb in candidates:
        climb['score'] = calculate_style_score(climb, style)
        climb['score'] += min(climb['ascents'] / 100, 2)  # Bonus popularité
    
    candidates.sort(key=lambda c: c['score'], reverse=True)
    
    # Sélection avec diversité
    session = []
    used_zones = set()
    used_setters = set()
    
    for climb in candidates:
        zone = get_zone(climb)
        setter = climb['setter']
        
        # Éviter répétitions
        if zone not in used_zones or len(session) >= count * 0.7:
            session.append(climb)
            used_zones.add(zone)
            used_setters.add(setter)
        
        if len(session) >= count:
            break
    
    return session
```

## 🎨 Wireframes

### Écran Home
```
┌─────────────────────────────┐
│  🧗 Kilter Route Selector   │
├─────────────────────────────┤
│                             │
│  Grade: [V4] ━━━━ [V6]      │
│                             │
│  Style: ○ Dynamic           │
│         ● Technical         │
│         ○ Endurance         │
│         ○ Balanced          │
│                             │
│  Session size: [10] routes  │
│                             │
│  [🎲 Generate Session]      │
│                             │
├─────────────────────────────┤
│  📊 Community Stats         │
│  • 15,432 total routes      │
│  • 847 new this week        │
│  • Top setter: VinceTheP... │
└─────────────────────────────┘
```

### Écran Session
```
┌─────────────────────────────┐
│  📋 Your Session (10)       │
├─────────────────────────────┤
│  1. ⭐⭐⭐⭐ Narasaki Bounce │
│     V8 • 12 moves • Dynamic │
│     142 ascents • 3.8/5     │
│                             │
│  2. ⭐⭐⭐ Crimpy Bliss      │
│     V5 • 8 moves • Technical│
│     89 ascents • 4.2/5      │
│                             │
│  [📤 Export] [🔄 Regenerate]│
└─────────────────────────────┘
```

## 🤝 Contribution

Ce projet est en phase de brainstorming/MVP. Contributions bienvenues !

## 📝 License

À définir

## 🔗 Liens utiles

- [Kilter Board Official](https://settercloset.com/pages/kb-overview)
- [BoardLib (Python library)](https://github.com/lemeryfertitta/BoardLib)
- [Kilter.jl (Julia)](https://github.com/FrederikSchnack/Kilter.jl)

---

**Créé le 2026-02-27** 🫐
