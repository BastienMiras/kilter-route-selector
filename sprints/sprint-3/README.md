# Sprint 3 - Client MVP (React + Nginx)

**Durée estimée** : 1 semaine
**Objectif** : Interface web React fonctionnelle et accessible sur Android / iPhone / PC

## Milestone

Application web React accessible via navigateur, générant et affichant des sessions.
Déployée via 2 containers Podman (frontend Nginx + backend FastAPI).

## Tâches

### 1. Setup React + TypeScript + Vite
- [ ] [TASK-1.1] Initialiser projet : `npm create vite@latest frontend -- --template react-ts`
- [ ] [TASK-1.2] Installer dépendances : TanStack Query, React Router v6, Recharts
- [ ] [TASK-1.3] Structure de dossiers : pages/, components/, api/, types/
- [ ] [TASK-1.4] Configuration Vite (base path, proxy dev vers backend)

### 2. Containerisation Podman
- [ ] [TASK-2.1] `frontend/Dockerfile` (build Vite multi-stage + Nginx)
- [ ] [TASK-2.2] `frontend/nginx.conf` (serve static + proxy /api/* → backend:8000)
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
- [ ] [TASK-4.4] Gestion erreurs réseau

### 5. Pages
- [ ] [TASK-5.1] `src/pages/Home.tsx` (FilterPanel + bouton Generate)
- [ ] [TASK-5.2] `src/pages/Session.tsx` (liste RouteCard + actions)
- [ ] [TASK-5.3] `src/pages/Detail.tsx` (métriques + HeatmapCanvas)
- [ ] [TASK-5.4] Navigation React Router v6 (/, /session, /route/:id)

### 6. Composants
- [ ] [TASK-6.1] `src/components/FilterPanel.tsx` (sliders grade, style, taille session)
- [ ] [TASK-6.2] `src/components/RouteCard.tsx` (carte voie avec métriques clés)
- [ ] [TASK-6.3] `src/components/HeatmapCanvas.tsx` (Canvas API, positions des prises)
- [ ] [TASK-6.4] `src/components/LoadingSkeleton.tsx`

## Critères d'acceptation

- App accessible sur http://localhost (`podman-compose up`)
- Génération de session fonctionnelle via /api/sessions/generate
- UI responsive : mobile (375px) et desktop (1280px)
- HeatmapCanvas affiche les prises positionnées correctement
- États loading/error gérés proprement
