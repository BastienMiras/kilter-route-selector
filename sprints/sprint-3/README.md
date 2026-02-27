# Sprint 3 - Client MVP (Flutter)

**Durée estimée** : 1 semaine  
**Objectif** : App fonctionnelle end-to-end (Android + Desktop)

## 🎯 Milestone

Application Flutter permettant de générer et visualiser des sessions.

## 📋 Tâches

### 1. Setup Flutter
- [ ] [TASK-1.1] Initialiser projet Flutter
- [ ] [TASK-1.2] Configuration Android/Desktop
- [ ] [TASK-1.3] Structure de dossiers (clean architecture)
- [ ] [TASK-1.4] Dependencies (dio, riverpod, hive)

### 2. API Client
- [ ] [TASK-2.1] Service HTTP (Dio)
- [ ] [TASK-2.2] Modèles Dart (Climb, Metrics, etc.)
- [ ] [TASK-2.3] Repository pattern
- [ ] [TASK-2.4] Gestion erreurs réseau

### 3. UI Screens
- [ ] [TASK-3.1] Home Screen (filtres)
- [ ] [TASK-3.2] Session Screen (liste des voies)
- [ ] [TASK-3.3] Detail Screen (métriques)
- [ ] [TASK-3.4] Navigation (go_router)

### 4. State Management
- [ ] [TASK-4.1] Providers (Riverpod)
- [ ] [TASK-4.2] State pour filtres
- [ ] [TASK-4.3] State pour session
- [ ] [TASK-4.4] Loading/Error states

### 5. Cache Local
- [ ] [TASK-5.1] Hive setup
- [ ] [TASK-5.2] Cache des sessions
- [ ] [TASK-5.3] Sync stratégie

## 📊 Critères d'acceptation

- ✅ App compile Android + Desktop
- ✅ Génération de session fonctionnelle
- ✅ UI responsive et fluide
- ✅ Offline: dernière session visible
