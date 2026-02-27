# Index des Tâches - Kilter Route Selector

**Total: 103 tâches, ~240h estimées**

---

## Sprint 1 - Backend Fondation (28 tâches, ~48h)

### 📦 Setup Infrastructure (4 tâches, ~6h)
1. `TASK-1.1` - Setup FastAPI [2h] ✅ Créé
2. `TASK-1.2` - Configuration PostgreSQL [1h] ✅ Créé
3. `TASK-1.3` - Schéma de base de données [2h] ✅ Créé
4. `TASK-1.4` - Configuration Docker/venv [1h]

### 🔍 Reverse Engineering API (4 tâches, ~6h)
5. `TASK-2.1` - Test authentification [1h] ✅ Créé
6. `TASK-2.2` - Explorer endpoints [2h] ✅ Créé
7. `TASK-2.3` - Documenter structure JSON [1h]
8. `TASK-2.4` - Implémenter client API Python [2h]

### 🌐 Scraping (4 tâches, ~8h)
9. `TASK-3.1` - Worker de scraping [3h] ✅ Créé
10. `TASK-3.2` - Rate limiting [1h]
11. `TASK-3.3` - Error handling & retries [2h]
12. `TASK-3.4` - Scraper 1000+ voies [2h]

### 📝 Parsing Layouts (4 tâches, ~6h)
13. `TASK-4.1` - Parser format p{pos}r{rad} [2h] ✅ Créé
14. `TASK-4.2` - Mapper positions → coordonnées [1h]
15. `TASK-4.3` - Stocker dans table holds [2h]
16. `TASK-4.4` - Identifier start/finish/foot [1h]

### 📊 Calcul Métriques (4 tâches, ~8h)
17. `TASK-5.1` - Distances & portées [2h] ✅ Créé
18. `TASK-5.2` - Ranges vertical/horizontal [1h]
19. `TASK-5.3` - Symétrie & densité [2h]
20. `TASK-5.4` - Scores de style [3h]

### 🔌 API Endpoints (4 tâches, ~6h)
21. `TASK-6.1` - GET /api/climbs [2h]
22. `TASK-6.2` - Filtres & validation [1h]
23. `TASK-6.3` - GET /api/climbs/{id} [1h]
24. `TASK-6.4` - Documentation Swagger [2h]

### ✅ Tests & Validation (4 tâches, ~8h)
25. `TASK-7.1` - Tests unitaires [3h]
26. `TASK-7.2` - Tests d'intégration [3h]
27. `TASK-7.3` - Validation données [1h]
28. `TASK-7.4` - Logging & monitoring [1h]

---

## Sprint 2 - Session Builder (17 tâches, ~34h)

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
43. `TASK-4.1` - Cache résultats [2h]
44. `TASK-4.2` - Index DB performance [2h]
45. `TASK-4.3` - Monitoring temps réponse [2h]

---

## Sprint 3 - Client Flutter MVP (19 tâches, ~40h)

### 🚀 Setup Flutter (4 tâches, ~6h)
46. `TASK-1.1` - Init projet Flutter [1h]
47. `TASK-1.2` - Config Android/Desktop [2h]
48. `TASK-1.3` - Structure dossiers (clean arch) [1h]
49. `TASK-1.4` - Dependencies (dio, riverpod, hive) [2h]

### 🌐 API Client (4 tâches, ~8h)
50. `TASK-2.1` - Service HTTP (Dio) [2h]
51. `TASK-2.2` - Modèles Dart (Climb, Metrics) [2h]
52. `TASK-2.3` - Repository pattern [2h]
53. `TASK-2.4` - Gestion erreurs réseau [2h]

### 🎨 UI Screens (4 tâches, ~12h)
54. `TASK-3.1` - Home Screen (filtres) [3h]
55. `TASK-3.2` - Session Screen (liste voies) [3h]
56. `TASK-3.3` - Detail Screen (métriques) [3h]
57. `TASK-3.4` - Navigation (go_router) [3h]

### 🔄 State Management (4 tâches, ~8h)
58. `TASK-4.1` - Setup Riverpod providers [2h]
59. `TASK-4.2` - State pour filtres [2h]
60. `TASK-4.3` - State pour session [2h]
61. `TASK-4.4` - Loading/Error states [2h]

### 💾 Cache Local (3 tâches, ~6h)
62. `TASK-5.1` - Hive setup [2h]
63. `TASK-5.2` - Cache des sessions [2h]
64. `TASK-5.3` - Sync stratégie [2h]

---

## Sprint 4 - Visualisation (15 tâches, ~36h)

### 🗺️ Heatmap des prises (4 tâches, ~12h)
65. `TASK-1.1` - Canvas custom pour prises [4h]
66. `TASK-1.2` - Mapping positions → écran [2h]
67. `TASK-1.3` - Couleurs par type (start/finish) [3h]
68. `TASK-1.4` - Zoom & pan [3h]

### 📍 Zone Map (3 tâches, ~6h)
69. `TASK-2.1` - Grille 3×3 du mur [2h]
70. `TASK-2.2` - Highlighting zones utilisées [2h]
71. `TASK-2.3` - Stats par zone [2h]

### 📊 Graphiques (4 tâches, ~10h)
72. `TASK-3.1` - Setup fl_chart [2h]
73. `TASK-3.2` - Bar charts (distances, moves) [3h]
74. `TASK-3.3` - Radar chart (styles) [3h]
75. `TASK-3.4` - Distribution grades [2h]

### ✨ Animations (4 tâches, ~8h)
76. `TASK-4.1` - Transitions entre screens [2h]
77. `TASK-4.2` - Loading skeletons [2h]
78. `TASK-4.3` - Micro-interactions [2h]
79. `TASK-4.4` - Polish 60fps [2h]

---

## Sprint 5+ - Features Avancées (24 tâches, ~82h)

### 👤 Auth & Users (3 tâches, ~12h)
80. `TASK-1.1` - Backend JWT auth [4h]
81. `TASK-1.2` - Signup/Login UI [4h]
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
100. `TASK-7.1` - Optimisation DB (indexes) [3h]
101. `TASK-7.2` - Caching agressif [3h]
102. `TASK-7.3` - Compression data [3h]
103. `TASK-7.4` - Analytics (Sentry) [3h]

---

## 📊 Statistiques

| Catégorie | Sprint | Tâches | Heures |
|-----------|--------|--------|--------|
| Backend | 1 | 28 | 48h |
| Logic | 2 | 17 | 34h |
| Client | 3 | 19 | 40h |
| UI/UX | 4 | 15 | 36h |
| Advanced | 5+ | 24 | 82h |
| **Total** | **5** | **103** | **240h** |

## 🎯 Milestones

- **Semaine 1** : Backend fonctionnel avec 1000+ voies
- **Semaine 2** : Session builder intelligent
- **Semaine 3** : App mobile/desktop MVP
- **Semaine 4** : Visualisations graphiques
- **Semaines 5+** : Production-ready avec ML

## 📝 Légende

- ✅ = Fichier de tâche créé et détaillé
- 🔴 = Priorité haute
- 🟡 = Priorité moyenne
- 🟢 = Priorité basse

---

**Document généré le 2026-02-27** 🫐
