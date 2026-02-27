# Sprint 2 - Session Builder

**Durée estimée** : 1 semaine  
**Objectif** : Générer des sessions de qualité avec diversité

## 🎯 Milestone

API capable de générer des sessions personnalisées de 10+ voies selon critères utilisateur.

## 📋 Tâches

### 1. Scoring par style
- [ ] [TASK-1.1] Implémenter score_dynamic()
- [ ] [TASK-1.2] Implémenter score_technical()
- [ ] [TASK-1.3] Implémenter score_endurance()
- [ ] [TASK-1.4] Implémenter score_balanced()
- [ ] [TASK-1.5] Tests unitaires scoring

### 2. Session Builder
- [ ] [TASK-2.1] Algorithme de sélection de base
- [ ] [TASK-2.2] Diversité par zones du mur
- [ ] [TASK-2.3] Diversité par setters
- [ ] [TASK-2.4] Bonus popularité/qualité
- [ ] [TASK-2.5] Tests d'intégration

### 3. API Endpoints
- [ ] [TASK-3.1] POST `/api/sessions/generate`
- [ ] [TASK-3.2] Validation des paramètres (Pydantic)
- [ ] [TASK-3.3] Documentation Swagger
- [ ] [TASK-3.4] Gestion erreurs (pas assez de voies, etc.)

### 4. Optimisations
- [ ] [TASK-4.1] Cache des résultats fréquents
- [ ] [TASK-4.2] Index DB pour performance
- [ ] [TASK-4.3] Monitoring temps de réponse

## 📊 Critères d'acceptation

- ✅ Sessions générées en < 2s
- ✅ Diversité visible (pas 2 fois même zone)
- ✅ Score pertinent (sessions dynamic ont bien des grandes portées)
- ✅ Tests couvrent 80%+ du code
