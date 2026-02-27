# Brainstorm - Kilter Route Selector

**Date** : 2026-02-27  
**Participants** : Lio, Mûre

---

## 🎯 Vision du projet

Créer un outil intelligent pour sélectionner automatiquement des voies sur Kilter Board en fonction de critères personnalisés (difficulté, type de mouvements, endurance, etc.).

## 🧗 Contexte : Qu'est-ce que la Kilter Board ?

- **Système d'entraînement moderne** avec LED sur les prises
- **App mobile** pour sélectionner des voies → allume les prises correspondantes
- **Layout standardisé** : mêmes positions de prises dans le monde entier
- **Codes couleurs** :
  - Prise de départ
  - Prise d'arrivée
  - Pied seul
  - Main et pied (standard)
  - Off (éteinte)

## 💡 Problème identifié

- **Trop de voies disponibles** → difficile de choisir
- **Pas de filtrage avancé** dans l'app officielle
- **Pas de génération automatique de sessions** d'entraînement
- **Pas d'analyse de styles** de mouvements

## ✨ Solution proposée

Un système qui :
1. **Agrège les données communautaires** (toutes les voies publiques)
2. **Calcule des métriques automatiques** (distances, types de mouvements, etc.)
3. **Génère des sessions personnalisées** avec diversité
4. **Recommande des voies** selon profil et progression

---

## 📊 Critères de sélection envisagés

### 1. Difficulté
- Grade V-scale/Font (V0 à V17)
- Fourchette min/max
- Progression graduelle

### 2. Type de mouvements
- **Statique vs dynamique** : analyse des distances entre prises
  - Grande distance = mouvement dynamique probable
  - Petite distance = technique, statique
- **Compression** : prises opposées
- **Coordination** : séquences complexes
- **Endurance** : nombre total de mouvements

### 3. Type de prises (analyse future)
- Crimps (petites)
- Slopers (arrondies)
- Pinces
- Jugs (bonnes prises)
- Répartition dans la voie

### 4. Zones du mur
- Division en grille 3×3 :
  - Vertical : top / middle / bottom
  - Horizontal : left / center / right
- **Objectif** : éviter de répéter les mêmes zones dans une session

### 5. Intensité physique
- **Core** : voies en dévers (selon angle du mur)
- **Force de doigts** : concentration de petites prises
- **Portées** : distance entre prises successives
- **Équilibre** : prises espacées latéralement

### 6. Méta-données communautaires
- **Popularité** : nombre d'ascensions
- **Notes/qualité** : moyenne communautaire (si disponible)
- **Setter** : ouvreur (réputation)
- **Fraîcheur** : date de création (nouvelles voies = découverte)

---

## 🏗️ Architecture technique

### Option retenue : **Agrégation communautaire**

**Avantages** :
- Beaucoup plus de voies disponibles
- Métadonnées enrichies (popularité, notes)
- Recommandations communautaires possibles
- Statistiques globales (trending, top setters)

**Schéma** :
```
API Kilter Board
    ↓ (scraping)
Backend central (PostgreSQL)
    ↓ (REST API)
Client (Flutter - Android/Desktop)
```

### Stack technique

**Backend** :
- Python + FastAPI
- PostgreSQL (voies, prises)
- Redis (cache)
- Worker async (scraping continu)

**Client** :
- Flutter (cross-platform : Android + Windows/Linux/macOS)
- Hive/sqflite (cache local)
- Dio (HTTP client)

---

## 🔍 Métriques calculées automatiquement

À partir du layout `p{position}r{radius}` :

### 1. Distance moyenne entre prises
Moyenne des distances euclidiennes entre prises successives.

**Interprétation** :
- Faible → technique, statique
- Élevée → dynamique, portées

### 2. Portée maximale
Plus grande distance entre deux prises consécutives.

**Interprétation** :
- Au-dessus d'un seuil → contient des dynos

### 3. Nombre de mouvements
`len(holds) - 1`

**Interprétation** :
- Élevé → endurance
- Faible → boulder court et intense

### 4. Distribution verticale
`max(y) - min(y)`

**Interprétation** :
- Grande plage → voie longue/haute
- Petite plage → traverse

### 5. Symétrie gauche/droite
Ratio des prises de chaque côté.

**Interprétation** :
- Score bas → équilibré
- Score élevé → concentré d'un côté (équilibre difficile)

### 6. Densité de prises
Nombre de prises par unité de surface.

**Interprétation** :
- Élevée → technique, choix tactique
- Faible → athlétique, peu de repos

---

## 🎯 Fonctionnalités envisagées

### Core Features (MVP)

1. **Filtrage multi-critères** :
   - Grade min/max
   - Style (dynamic/technical/endurance/balanced)
   - Nombre de mouvements min/max
   - Popularité min

2. **Session Builder** :
   - Génère 10+ voies selon critères
   - Diversité automatique (zones, setters)
   - Alternance intensité/repos

3. **Visualisation** :
   - Liste des voies avec métriques clés
   - Détail d'une voie (heatmap des prises)

### Features avancées (v2+)

4. **Recommandations communautaires** :
   - Trending routes (nouvelles + populaires)
   - Hidden gems (peu connues, bien notées)
   - Similar routes (clustering)

5. **Progression Tracking** :
   - Historique des sessions
   - "Trouve des V5 similaires à [réussie] mais plus dures"
   - Analyse des points faibles

6. **Setter Analytics** :
   - Top setters par style
   - Distribution des grades par setter

7. **Heat Maps** :
   - Zones du mur utilisées/sous-utilisées
   - Recommandations pour équilibrer

8. **Machine Learning** (plus tard) :
   - Clustering de styles de mouvements
   - Recommandations personnalisées par historique

---

## 🚀 Plan d'action par sprints

### Sprint 1 (1 semaine) - Backend fondation
**Objectif** : Avoir 1000+ voies en DB avec métriques calculées

- [ ] Setup serveur (FastAPI + PostgreSQL)
- [ ] Reverse engineering API Kilter Board
- [ ] Authentification et récupération de voies
- [ ] Parser layouts → table `holds` (x, y, order)
- [ ] Calculer métriques de base (distances, densité, etc.)
- [ ] Endpoint `/api/climbs` avec filtres (grade, style)
- [ ] Worker de scraping continu

**Livrables** :
- API fonctionnelle
- DB avec 1000+ voies
- Documentation des endpoints

### Sprint 2 (1 semaine) - Session Builder
**Objectif** : Générer des sessions de qualité

- [ ] Algorithmes de scoring par style (dynamic/technical/endurance)
- [ ] Session builder avec diversité (zones, setters)
- [ ] Endpoint `/api/sessions/generate`
- [ ] Tests avec différents profils utilisateur
- [ ] Bonus popularité/qualité

**Livrables** :
- Sessions variées et pertinentes
- Tests automatisés

### Sprint 3 (1 semaine) - Client MVP
**Objectif** : App fonctionnelle end-to-end

- [ ] Setup Flutter (Android + Desktop)
- [ ] Écran Home (filtres, bouton génération)
- [ ] Écran Session (liste des voies)
- [ ] Écran Detail (métriques, visualisation simple)
- [ ] Intégration API client (Dio)
- [ ] Cache local (Hive)

**Livrables** :
- App testable sur Android/PC
- UX fluide

### Sprint 4 (1 semaine) - Visualisation
**Objectif** : UX riche et informative

- [ ] Heatmap des prises (canvas custom)
- [ ] Graphiques de métriques (charts)
- [ ] Zone map du mur (grille 3×3)
- [ ] Animations et transitions

**Livrables** :
- Expérience visuelle engageante

### Sprint 5+ - Features avancées
- [ ] Comptes utilisateur & authentification
- [ ] Tracking de progression
- [ ] Recommandations ML
- [ ] Statistiques communautaires
- [ ] Export/import de sessions
- [ ] Mode hors-ligne complet
- [ ] Intégration avec Kilter Board app (si API officielle)

---

## 🔬 Reverse engineering API Kilter Board

### Endpoints identifiés

**Authentification** :
```http
POST https://api.kilterboardapp.com/v1/logins
Content-Type: application/json

{
  "username": "...",
  "password": "...",
  "tou": "accepted",
  "pp": "accepted"
}

Response:
{
  "login": {
    "token": "Bearer eyJhbGc..."
  }
}
```

**Récupération de voies** :
```http
GET https://api.kilterboardapp.com/v1/climbs/{id}
Authorization: Bearer {token}
```

### Stratégie de scraping

1. **Scraping initial** : Itérer sur IDs de voies (1 à N)
2. **Polling régulier** : Nouvelles voies toutes les 24h
3. **Rate limiting** : 1 req/sec max pour éviter ban
4. **Cache** : Stocker localement, ne pas re-scraper

### Format de layout

Exemple : `p1083r15p1117r15p1151r15...`

- `p{position}` : ID de la prise (mapping via table `leds`)
- `r{radius}` : Taille visuelle

**Conversion position → coordonnées** :
```python
x = position & 0xff          # 8 bits bas
y = (position & 0xff00) >> 8 # 8 bits haut
```

---

## 🎨 Wireframes UI

### Home Screen
```
┌─────────────────────────────┐
│  🧗 Kilter Route Selector   │
├─────────────────────────────┤
│                             │
│  Grade: [V4] ━━━━━ [V6]     │
│                             │
│  Style: ○ Dynamic           │
│         ● Technical         │
│         ○ Endurance         │
│         ○ Balanced          │
│                             │
│  Session size: [10] routes  │
│                             │
│  Advanced filters ▼         │
│                             │
│  [🎲 Generate Session]      │
│                             │
├─────────────────────────────┤
│  📊 Community Stats         │
│  • 15,432 total routes      │
│  • 847 new this week        │
│  • Top setter: VincePrince  │
│                             │
│  🔥 Trending Routes         │
│  1. Narasaki Bounce (V8)    │
│  2. Crimpy Bliss (V5)       │
│  3. Power Jug (V6)          │
└─────────────────────────────┘
```

### Session Screen
```
┌─────────────────────────────┐
│  ← 📋 Your Session (10)     │
├─────────────────────────────┤
│                             │
│  1. ⭐⭐⭐⭐ Narasaki Bounce │
│     V8 • 12 moves • Dynamic │
│     142 ascents • 3.8/5     │
│     [View Details →]        │
│                             │
│  2. ⭐⭐⭐ Crimpy Bliss      │
│     V5 • 8 moves • Technical│
│     89 ascents • 4.2/5      │
│     [View Details →]        │
│                             │
│  ... (8 more)               │
│                             │
├─────────────────────────────┤
│  [📤 Export] [🔄 Regenerate]│
│  [💾 Save Session]          │
└─────────────────────────────┘
```

### Detail Screen
```
┌─────────────────────────────┐
│  ← Narasaki Bounce          │
├─────────────────────────────┤
│                             │
│  ┌───────────────────────┐  │
│  │   [Heatmap canvas]    │  │
│  │   Visualisation des   │  │
│  │   prises sur le mur   │  │
│  └───────────────────────┘  │
│                             │
│  Grade: V8                  │
│  Setter: VinceThePrince     │
│  Ascents: 142 • Rating: 3.8 │
│                             │
│  📊 Metrics                 │
│  • 12 moves                 │
│  • Avg distance: 42.5 cm    │
│  • Max reach: 78.3 cm       │
│  • Style: Dynamic (8.5/10)  │
│                             │
│  [📤 Open in Kilter App]    │
│  [❤️  Save to favorites]    │
└─────────────────────────────┘
```

---

## 🤔 Questions ouvertes

### Techniques
1. **Rate limiting API** : Quelle limite avant ban ?
2. **Structure exacte des réponses API** : À confirmer par tests
3. **Mapping hold types** : Comment identifier crimps vs slopers ?
4. **Angle du mur** : Est-ce dans les métadonnées ou faut-il demander à l'utilisateur ?

### UX
1. **Export vers Kilter App** : Possible techniquement ?
2. **Authentification utilisateur** : Nécessaire dès le MVP ou plus tard ?
3. **Nom du projet** : KilterAI ? RouteBuilder ? Boulder Selector ? ClimbSmart ?

### Business/Legal
1. **Termes d'utilisation Kilter Board** : Scraping autorisé ?
2. **Monétisation** : Gratuit ? Freemium ? Donation ?
3. **Open source** : Oui/non ? Quelle licence ?

---

## 📝 Décisions prises

| Sujet | Décision | Raison |
|-------|----------|--------|
| **Source de données** | Agrégation communautaire (Option B) | Plus de voies, métadonnées riches |
| **Plateforme client** | Android + PC | Besoin identifié |
| **Approche scoring** | Règles simples d'abord, ML ensuite | Pragmatique, MVP rapide |
| **Stack backend** | Python + FastAPI | Familiarité, rapidité de dev |
| **Stack client** | Flutter | Cross-platform, un seul codebase |
| **Scraping** | Async worker continu | Données toujours à jour |

---

## 🎯 Prochaines étapes immédiates

1. **Tester l'API Kilter Board** :
   - Créer un compte de test
   - Authentification manuelle
   - Récupérer 10 voies pour analyser la structure

2. **Créer le repo Git** :
   - Structure de dossiers
   - README initial
   - Ce document de brainstorm

3. **Setup backend minimal** :
   - FastAPI boilerplate
   - PostgreSQL schema
   - Premier endpoint de test

4. **Parser un layout** :
   - Extraire les positions
   - Convertir en coordonnées
   - Calculer première métrique (distance moyenne)

---

## 📚 Ressources utiles

- [Kilter Board Official](https://settercloset.com/pages/kb-overview)
- [BoardLib (Python library)](https://github.com/lemeryfertitta/BoardLib)
- [Kilter.jl (Julia)](https://github.com/FrederikSchnack/Kilter.jl)
- [Blog reverse engineering](https://bazun.me/blog/kiterboard)

---

**Document vivant** - Mis à jour au fil du projet 🫐
