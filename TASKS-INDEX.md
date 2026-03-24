# Index des Tâches - Kilter Route Selector

**Total: 120 tâches, ~256h estimées**

> **Décisions architecturales majeures (voir `ARCHITECTURE-REVIEW.md` + `BRAINSTORM.md`) :**
> - PostgreSQL → **SQLite** (MVP, via BoardLib)
> - Scraper custom → **BoardLib** (200k routes en quelques minutes)
> - Flutter → **React + TypeScript + Vite** (PWA)
> - Docker/Kamal → **Podman + podman-compose + Ansible** (IaC)
> - Redis/Celery → **cron job** (simplification)

---

## Sprint 0 — Infrastructure as Code (17 tâches, ~16h)

> Spec : `docs/plans/2026-03-11-iac-podman-design.md`
> Plan : `docs/superpowers/plans/2026-03-11-sprint-0-iac.md`

### 🐳 Conteneurs (4 tâches, ~3h)
1. `TASK-0.1` - Backend skeleton (main.py + requirements.txt) [30min]
2. `TASK-0.2` - Containerfile.backend multi-stage [1h]
3. `TASK-0.3` - Frontend placeholder + nginx.conf [30min]
4. `TASK-0.4` - Containerfile.frontend [1h]

### 📦 Compose (3 tâches, ~2h)
5. `TASK-0.5` - .env.example + .gitignore [30min]
6. `TASK-0.6` - compose.dev.yml (local hot-reload) [1h]
7. `TASK-0.7` - compose.vps-dev/staging/prod.yml [30min]

### ⚙️ Ansible (5 tâches, ~6h)
8. `TASK-0.8` - Role: podman (install Podman + podman-compose) [1h]
9. `TASK-0.9` - Role: firewall (ufw ports 22/80/443/8081/8082) [1h]
10. `TASK-0.10` - Role: app (deploy + health check) [2h]
11. `TASK-0.11` - Inventory + group_vars + playbook-provision.yml [1h]
12. `TASK-0.12` - playbook-deploy.yml [1h]

### 🚀 CI/CD (4 tâches, ~4h30)
13. `TASK-0.13` - Makefile (up/down/build/test/provision/deploy-dev) [30min]
14. `TASK-0.14` - ci.yml (lint + tests + image build gate) [2h]
15. `TASK-0.15` - deploy-staging.yml (auto sur push develop) [1h]
16. `TASK-0.16` - deploy-prod.yml (approbation manuelle sur tag) [1h]

### 📝 Documentation (1 tâche, ~30min)
17. `TASK-0.17` - Sprint 0 README + TASKS-INDEX [30min]

---

## Sprint 1 — Backend Fondation (28 tâches, ~48h)

> Stack : Python 3.12, FastAPI, SQLite (BoardLib), aiosqlite, pytest

### 📦 Setup Backend (4 tâches, ~5h)
1. `TASK-1.1` - Setup FastAPI + structure projet [2h] ✅ Créé
2. `TASK-1.2` - Configuration SQLite + BoardLib [1h] ✅ Créé
3. `TASK-1.3` - Schéma climb_metrics (table à créer) [1h] ✅ Créé
4. `TASK-1.4` - TLS intra-conteneurs : vérifier que backend n'est pas exposé hors réseau Podman, documenter le flux Nginx → backend [1h]

### 🔍 Exploration BoardLib (2 tâches, ~3h)
4. `TASK-2.1` - Download DB boardlib + explorer tables climbs [1h] ✅ Créé
5. `TASK-2.2` - Documenter structure données Kilter (frames, roleCode) [2h] ✅ Créé

### 📝 Parsing Layouts (4 tâches, ~6h)
6. `TASK-4.1` - Parser format p{pos}r{roleCode} → x,y [2h] ✅ Créé
7. `TASK-4.2` - Mapper positions → coordonnées écran [1h]
8. `TASK-4.3` - Stocker parsed holds en mémoire/cache [2h]
9. `TASK-4.4` - Identifier start/finish/foot (roleCode mapping) [1h]

### 📊 Calcul Métriques (7 tâches, ~14h)
10. `TASK-5.1` - avg_distance & max_reach [2h] ✅ Créé
11. `TASK-5.2` - vertical_range & horizontal_range [1h]
12. `TASK-5.3` - symmetry_score & hold_density [2h]
13. `TASK-5.4` - score_dynamic / score_technical / score_endurance (normalisés 0-1) [3h]
14. `TASK-5.5` - Calcul batch pour toutes les voies [2h]
15. `TASK-5.6` - Stocker dans table climb_metrics [2h]
16. `TASK-5.7` - Prise en compte angle du mur dans scoring [2h]

### 🔌 API Endpoints (4 tâches, ~6h)
17. `TASK-6.1` - GET /api/climbs (liste + filtres) [2h]
18. `TASK-6.2` - Filtres & validation Pydantic (grade, style, angle) [1h]
19. `TASK-6.3` - GET /api/climbs/{id} (détail + métriques) [1h]
20. `TASK-6.4` - Documentation Swagger + /health [2h]

### ✅ Tests & Validation (8 tâches, ~15h)
21. `TASK-7.1` - Tests unitaires parsing [3h]
22. `TASK-7.2` - Tests unitaires métriques [3h]
23. `TASK-7.3` - Tests intégration API [3h]
24. `TASK-7.4` - Validation données (grade_numeric, roleCode) [1h]
25. `TASK-7.5` - Logging structuré [1h]
26. `TASK-7.6` - Cron job sync BoardLib (hebdomadaire) [2h]
27. `TASK-7.7` - Update Containerfile.backend (remplace skeleton Sprint 0) [1h]
28. `TASK-7.8` - Vérification end-to-end via `make up` [1h]

---

## Sprint 2 — Session Builder (17 tâches, ~34h)

### 🎯 Scoring par style (5 tâches, ~10h)
29. `TASK-1.1` - score_dynamic() [2h]
30. `TASK-1.2` - score_technical() [2h]
31. `TASK-1.3` - score_endurance() [2h]
32. `TASK-1.4` - score_balanced() [2h]
33. `TASK-1.5` - Tests unitaires scoring [2h]

### 🎲 Session Builder (5 tâches, ~12h)
34. `TASK-2.1` - Algorithme sélection base [3h]
35. `TASK-2.2` - Diversité zones du mur [2h]
36. `TASK-2.3` - Diversité setters [2h]
37. `TASK-2.4` - Bonus popularité/qualité [2h]
38. `TASK-2.5` - Tests intégration [3h]

### 🔌 API Endpoints (4 tâches, ~6h)
39. `TASK-3.1` - POST /api/sessions/generate [2h]
40. `TASK-3.2` - Validation Pydantic [1h]
41. `TASK-3.3` - Documentation Swagger [1h]
42. `TASK-3.4` - Gestion erreurs [2h]

### ⚡ Optimisations (3 tâches, ~6h)
43. `TASK-4.1` - Index SQLite performance [2h]
44. `TASK-4.2` - Cache résultats en mémoire [2h]
45. `TASK-4.3` - Monitoring temps réponse [2h]

---

## Sprint 3 — Client React MVP (19 tâches, ~40h)

> Stack : React 18, TypeScript, Vite, TanStack Query, React Router, Recharts, Nginx, Podman

### 🚀 Setup React (4 tâches, ~6h)
46. `TASK-1.1` - Init projet React + TypeScript + Vite [1h]
47. `TASK-1.2` - Config ESLint + Prettier + Vitest [2h]
48. `TASK-1.3` - Structure dossiers (features, components, hooks) [1h]
49. `TASK-1.4` - Dependencies (TanStack Query, Recharts, React Router) [2h]

### 🌐 API Client (4 tâches, ~8h)
50. `TASK-2.1` - Client HTTP (fetch/axios) [2h]
51. `TASK-2.2` - Modèles TypeScript (Climb, Metrics, Session) [2h]
52. `TASK-2.3` - TanStack Query hooks [2h]
53. `TASK-2.4` - Gestion erreurs réseau [2h]

### 🎨 UI Screens (4 tâches, ~12h)
54. `TASK-3.1` - Home Screen (filtres style/grade/angle) [3h]
55. `TASK-3.2` - Session Screen (liste voies) [3h]
56. `TASK-3.3` - Detail Screen (métriques + heatmap basique) [3h]
57. `TASK-3.4` - Navigation (React Router) [3h]

### 🐳 Conteneurs frontend (4 tâches, ~8h)
58. `TASK-4.1` - Mise à jour Containerfile.frontend (builder Vite + nginx) [2h]
59. `TASK-4.2` - Mise à jour compose.dev.yml (Vite dev server) [1h]
60. `TASK-4.3` - Config nginx.conf finalisée (SPA + proxy /api) [2h]
61. `TASK-4.4` - Tests end-to-end via `make up` [3h]

### 💾 État & Cache (3 tâches, ~6h)
62. `TASK-5.1` - State global (Context ou Zustand) [2h]
63. `TASK-5.2` - Cache local (TanStack Query staleTime) [2h]
64. `TASK-5.3` - Loading/Error states [2h]

---

## Sprint 4 — Visualisation (15 tâches, ~36h)

### 🗺️ Heatmap des prises (4 tâches, ~12h)
65. `TASK-1.1` - Canvas custom pour prises (positions x,y) [4h]
66. `TASK-1.2` - Mapping positions → écran [2h]
67. `TASK-1.3` - Couleurs par roleCode (start/finish/foot) [3h]
68. `TASK-1.4` - Zoom & pan [3h]

### 📍 Zone Map (3 tâches, ~6h)
69. `TASK-2.1` - Grille 3×3 du mur [2h]
70. `TASK-2.2` - Highlighting zones utilisées [2h]
71. `TASK-2.3` - Stats par zone [2h]

### 📊 Graphiques Recharts (4 tâches, ~10h)
72. `TASK-3.1` - Setup Recharts [2h]
73. `TASK-3.2` - Bar charts (distances, moves) [3h]
74. `TASK-3.3` - Radar chart (styles) [3h]
75. `TASK-3.4` - Distribution grades [2h]

### ✨ Animations (4 tâches, ~8h)
76. `TASK-4.1` - Transitions entre screens [2h]
77. `TASK-4.2` - Loading skeletons [2h]
78. `TASK-4.3` - Micro-interactions [2h]
79. `TASK-4.4` - Polish responsive (mobile/desktop) [2h]

---

## Sprint 5+ — Features Avancées (24 tâches, ~82h)

### 👤 Auth & Users (3 tâches, ~12h)
80. `TASK-1.1` - Backend JWT auth [4h]
81. `TASK-1.2` - Signup/Login UI React [4h]
82. `TASK-1.3` - Profile management [4h]

### 📈 Progression Tracking (4 tâches, ~12h)
83. `TASK-2.1` - Table user_progress [3h]
84. `TASK-2.2` - Mark completed UI [3h]
85. `TASK-2.3` - Stats personnelles [3h]
86. `TASK-2.4` - Graphiques progression [3h]

### 🤖 ML Recommendations (3 tâches, ~16h)
87. `TASK-3.1` - Clustering K-means [6h]
88. `TASK-3.2` - Collaborative filtering [6h]
89. `TASK-3.3` - UI "You might like" [4h]

### 🌍 Stats Communautaires (4 tâches, ~12h)
90. `TASK-4.1` - Trending routes [3h]
91. `TASK-4.2` - Top setters by style [3h]
92. `TASK-4.3` - Hidden gems [3h]
93. `TASK-4.4` - Global heatmap [3h]

### 📤 Export/Import (3 tâches, ~8h)
94. `TASK-5.1` - Export session JSON [2h]
95. `TASK-5.2` - Import session partagée [3h]
96. `TASK-5.3` - QR code partage [3h]

### 📴 Offline Mode (3 tâches, ~10h)
97. `TASK-6.1` - Sync stratégie complète [4h]
98. `TASK-6.2` - Offline indicator [2h]
99. `TASK-6.3` - Queue sync actions [4h]

### ✨ Polish (4 tâches, ~12h)
100. `TASK-7.1` - Optimisation SQLite (indexes avancés) [3h]
101. `TASK-7.2` - Caching agressif [3h]
102. `TASK-7.3` - Compression data [3h]
103. `TASK-7.4` - Analytics (Sentry) [3h]

### 📊 Monitoring & Observabilité (2 tâches, ~8h)
104. `TASK-8.1` - Monitoring centralisé : métriques VPS + alertes (Prometheus/Grafana ou UptimeRobot) [5h]
105. `TASK-8.2` - Centralisation des logs conteneurs (Loki ou journald → stdout structuré) [3h]

---

## 📊 Statistiques

| Catégorie | Sprint | Tâches | Heures |
|-----------|--------|--------|--------|
| Infra IaC | 0 | 17 | 16h |
| Backend | 1 | 29 | 49h |
| Logic | 2 | 17 | 34h |
| Client React | 3 | 19 | 40h |
| UI/UX | 4 | 15 | 36h |
| Advanced | 5+ | 26 | 90h |
| **Total** | **6** | **123** | **265h** |

## 🎯 Milestones

- **Sprint 0** : Infrastructure complète, `make up` fonctionnel, CI active
- **Sprint 1** : Backend avec 200k+ voies + métriques calculées
- **Sprint 2** : Session builder intelligent
- **Sprint 3** : App web React MVP déployée sur staging
- **Sprint 4** : Visualisations complètes
- **Sprint 5+** : Production-ready avec ML

## 📝 Légende

- ✅ = Fichier de tâche créé et détaillé
- 🔴 = Priorité haute
- 🟡 = Priorité moyenne
- 🟢 = Priorité basse

---

**Dernière mise à jour : 2026-03-11**
