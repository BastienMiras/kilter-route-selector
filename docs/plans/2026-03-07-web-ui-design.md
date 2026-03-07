# Web UI (React + Nginx + Podman) Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Remplacer Flutter par une interface web React+TypeScript+Vite servie par Nginx via Podman, compatible Android / iPhone / PC.

**Architecture:** Container Nginx sert les fichiers React compilés et proxy `/api/*` vers le container FastAPI. Deux containers orchestres par podman-compose. Pas de CORS, un seul point d'entrée.

**Tech Stack:** React 18, TypeScript, Vite, Nginx, Podman, podman-compose, TanStack Query, React Router v6, Recharts, Canvas API

---

### Task 1: Mise à jour BRAINSTORM.md

**Files:**
- Modify: `BRAINSTORM.md`

**Step 1: Remplacer la section "Stack technique" client**

Localiser le bloc (lignes ~103-114) :
```
**Client** :
- Flutter (cross-platform : Android + Windows/Linux/macOS)
- Hive/sqflite (cache local)
- Dio (HTTP client)
```

Remplacer par :
```
**Client** :
- React 18 + TypeScript + Vite (SPA cross-platform)
- Nginx (serving static + reverse proxy vers FastAPI)
- Podman + podman-compose (orchestration containers)
- TanStack Query (data fetching, cache, états loading/error)
- React Router v6 (navigation SPA)
- Canvas API / SVG (heatmap des prises)
- Recharts (graphiques de métriques)
```

**Step 2: Mettre à jour le schéma architecture**

Localiser le bloc schéma (lignes ~94-101) :
```
boardlib sync → kilter.db (SQLite)
                    ↓
             FastAPI (aiosqlite)
                    ↓
             Flutter client
```

Remplacer par :
```
boardlib sync → kilter.db (SQLite)
                    ↓
             FastAPI (aiosqlite)   ← container backend :8000
                    ↓
             Nginx (reverse proxy) ← container frontend :80
                    ↓
             React + TypeScript    ← SPA (Android / iPhone / PC)
```

**Step 3: Mettre à jour la décision dans le tableau**

Localiser la ligne Flutter dans le tableau des décisions (~ligne 444) :
```
| **Stack client** | Flutter | Cross-platform, un seul codebase |
```

Remplacer par :
```
| **Stack client** | React + TypeScript + Nginx (Podman) | Web universal (Android/iPhone/PC), pas d'installation native |
```

**Step 4: Mettre à jour le Sprint 3 dans le plan d'action**

Localiser la section Sprint 3 (~ligne 241) :
```
- [ ] Setup Flutter (Android + Desktop)
- [ ] Écran Home (filtres, bouton génération)
- [ ] Écran Session (liste des voies)
- [ ] Écran Detail (métriques, visualisation simple)
- [ ] Intégration API client (Dio)
- [ ] Cache local (Hive)
```

Remplacer par :
```
- [ ] Setup React + TypeScript + Vite
- [ ] Configuration Nginx (serve static + proxy /api/*)
- [ ] Dockerfiles + podman-compose.yml (2 containers)
- [ ] API client (TanStack Query → /api/climbs, /api/sessions)
- [ ] Pages : Home (filtres), Session (liste), Detail (métriques)
- [ ] Composants : FilterPanel, RouteCard, HeatmapCanvas
```

**Step 5: Commit**

```bash
git add BRAINSTORM.md
git commit -m "docs: replace Flutter with React+TypeScript+Nginx in brainstorm"
```

---

### Task 2: Mise à jour README.md

**Files:**
- Modify: `README.md`

**Step 1: Mettre à jour le schéma Architecture**

Localiser la section Architecture (~ligne 24) et remplacer :
```
boardlib sync
    ↓
kilter.db (SQLite) ← données officielles Kilter Board
    ↓
FastAPI + aiosqlite ← métriques calculées (climb_metrics)
    ↓
Client Flutter (Android / Desktop)
```

Par :
```
boardlib sync
    ↓
kilter.db (SQLite)          ← données officielles Kilter Board
    ↓
FastAPI + aiosqlite          ← métriques calculées (climb_metrics)
    ↓                            container backend (:8000, interne)
Nginx (reverse proxy)        ← container frontend (:80)
    ↓
React + TypeScript (Vite)   ← SPA responsive (Android / iPhone / PC)
```

**Step 2: Mettre à jour la section Stack technique**

Localiser la section "### Client" (~ligne 138) :
```
### Client
- **Framework** : Flutter 3.x
- **Platforms** : Android, Windows, Linux, macOS
- **State management** : Riverpod / Bloc
- **Local DB** : Hive / sqflite
- **HTTP** : Dio
```

Remplacer par :
```
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
```

**Step 3: Mettre à jour le Sprint 3 dans Roadmap**

Localiser "### Sprint 3" (~ligne 78) :
```
### Sprint 3 (1 semaine) - Client MVP
- [ ] Setup Flutter (Android + Desktop)
- [ ] Écrans : Home, Session, Detail
- [ ] Intégration API client
- [ ] Cache local (Hive/sqflite)
- **Milestone** : App fonctionnelle end-to-end
```

Remplacer par :
```
### Sprint 3 (1 semaine) - Client MVP (React + Nginx)
- [ ] Setup React + TypeScript + Vite + configuration Nginx
- [ ] Dockerfiles + podman-compose.yml (2 containers)
- [ ] API client TanStack Query (climbs, sessions)
- [ ] Pages : Home (filtres), Session (liste), Detail (métriques)
- [ ] Composants : FilterPanel, RouteCard, HeatmapCanvas
- **Milestone** : App accessible sur http://localhost, responsive mobile/desktop
```

**Step 4: Commit**

```bash
git add README.md
git commit -m "docs: replace Flutter with React+TypeScript+Nginx+Podman in README"
```

---

### Task 3: Réécriture sprints/sprint-3/README.md

**Files:**
- Modify: `sprints/sprint-3/README.md`

**Step 1: Réécrire entièrement le fichier**

Remplacer tout le contenu par :
```markdown
# Sprint 3 - Client MVP (React + Nginx)

**Duree estimee** : 1 semaine
**Objectif** : Interface web React fonctionnelle et accessible sur Android / iPhone / PC

## Milestone

Application web React accessible via navigateur, generant et affichant des sessions.
Deployee via 2 containers Podman (frontend Nginx + backend FastAPI).

## Taches

### 1. Setup React + TypeScript + Vite
- [ ] [TASK-1.1] Initialiser projet : `npm create vite@latest frontend -- --template react-ts`
- [ ] [TASK-1.2] Installer dependances : TanStack Query, React Router v6, Recharts
- [ ] [TASK-1.3] Structure de dossiers : pages/, components/, api/, types/
- [ ] [TASK-1.4] Configuration Vite (base path, proxy dev)

### 2. Containerisation Podman
- [ ] [TASK-2.1] `frontend/Dockerfile` (build Vite + Nginx)
- [ ] [TASK-2.2] `frontend/nginx.conf` (serve static + proxy /api/* -> backend:8000)
- [ ] [TASK-2.3] `backend/Dockerfile` (FastAPI + uvicorn)
- [ ] [TASK-2.4] `podman-compose.yml` (2 services : frontend, backend + volume kilter.db)

### 3. Types TypeScript
- [ ] [TASK-3.1] `src/types/climb.ts` (Climb, ClimbMetrics)
- [ ] [TASK-3.2] `src/types/session.ts` (Session, SessionParams)
- [ ] [TASK-3.3] `src/types/filters.ts` (FilterState)

### 4. API Client (TanStack Query)
- [ ] [TASK-4.1] `src/api/climbs.ts` (useClimbs, useClimbDetail)
- [ ] [TASK-4.2] `src/api/sessions.ts` (useGenerateSession)
- [ ] [TASK-4.3] Configuration QueryClient (staleTime, retry)
- [ ] [TASK-4.4] Gestion erreurs reseau (error boundaries)

### 5. Pages
- [ ] [TASK-5.1] `src/pages/Home.tsx` (FilterPanel + bouton Generate)
- [ ] [TASK-5.2] `src/pages/Session.tsx` (liste RouteCard + actions)
- [ ] [TASK-5.3] `src/pages/Detail.tsx` (metriques + HeatmapCanvas)
- [ ] [TASK-5.4] Navigation React Router v6 (/, /session, /route/:id)

### 6. Composants
- [ ] [TASK-6.1] `src/components/FilterPanel.tsx` (sliders grade, style, taille)
- [ ] [TASK-6.2] `src/components/RouteCard.tsx` (carte voie avec metriques)
- [ ] [TASK-6.3] `src/components/HeatmapCanvas.tsx` (Canvas API, positions prises)
- [ ] [TASK-6.4] `src/components/LoadingSkeleton.tsx`

## Criteres d'acceptation

- App accessible sur http://localhost (podman-compose up)
- Generation de session fonctionnelle via /api/sessions/generate
- UI responsive : mobile (375px) et desktop (1280px)
- HeatmapCanvas affiche les prises positionnees correctement
- Etats loading/error geres proprement
```

**Step 2: Commit**

```bash
git add sprints/sprint-3/README.md
git commit -m "docs: rewrite sprint-3 for React+Nginx+Podman (replace Flutter)"
```

---

### Task 4: Mise à jour sprints/sprint-4/README.md

**Files:**
- Modify: `sprints/sprint-4/README.md`

**Step 1: Remplacer les références Flutter par React**

Remplacer la section "Graphiques de métriques" :
```
- [ ] [TASK-3.1] Intégration fl_chart
- [ ] [TASK-3.2] Bar chart (move count, distances)
- [ ] [TASK-3.3] Radar chart (styles: dynamic/technical/endurance)
- [ ] [TASK-3.4] Distribution grades
```

Par :
```
- [ ] [TASK-3.1] Integration Recharts (bibliotheque React)
- [ ] [TASK-3.2] BarChart (move count, distances)
- [ ] [TASK-3.3] RadarChart (styles: dynamic/technical/endurance)
- [ ] [TASK-3.4] Distribution grades (BarChart)
```

**Step 2: Mettre à jour la section Heatmap**

Remplacer :
```
- [ ] [TASK-1.1] Canvas custom pour dessiner prises
```

Par :
```
- [ ] [TASK-1.1] Canvas API (amelioration HeatmapCanvas du Sprint 3)
```

**Step 3: Commit**

```bash
git add sprints/sprint-4/README.md
git commit -m "docs: update sprint-4 visualisation for React (Recharts, Canvas API)"
```

---

## Verification finale

```bash
# Verifier que toutes les references Flutter ont ete supprimees
grep -r "Flutter\|flutter\|Hive\|sqflite\|Riverpod\|fl_chart\|Dio" \
  BRAINSTORM.md README.md sprints/sprint-3/README.md sprints/sprint-4/README.md

# Resultat attendu : aucune ligne (0 occurrences)
```
