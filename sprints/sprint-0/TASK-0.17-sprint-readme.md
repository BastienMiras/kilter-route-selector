# TASK-0.17 - Sprint 0 README + TASKS-INDEX

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Moyenne
**Dépendances** : TASK-0.1, TASK-0.2, TASK-0.3, TASK-0.4, TASK-0.5, TASK-0.6, TASK-0.7, TASK-0.8, TASK-0.9, TASK-0.10, TASK-0.11, TASK-0.12, TASK-0.13, TASK-0.14, TASK-0.15, TASK-0.16

## Objectif

Mettre à jour le README du Sprint 0 pour refléter les livraisons complètes et marquer les tâches Sprint 0 comme terminées dans `TASKS-INDEX.md`.

## Checklist

- [ ] Vérifier que `sprints/sprint-0/README.md` documente toutes les livraisons du sprint
- [ ] Marquer les 17 tâches Sprint 0 comme complètes dans `TASKS-INDEX.md` (ajouter ✅)
- [ ] Vérifier que le milestone Sprint 0 est bien décrit : `make up` fonctionnel, CI active, VPS provisionnable

## Structure attendue

```
sprints/sprint-0/
└── README.md  (mis à jour)
TASKS-INDEX.md  (modifié)
```

## Exemple de contenu

**Contenu cible de `sprints/sprint-0/README.md`**
```markdown
# Sprint 0 — Infrastructure as Code

**Objectif:** Mettre en place toute l'infrastructure (conteneurs, orchestration, déploiement) avant d'écrire le code applicatif.

**Livraisons:**
- Containerfile.backend + Containerfile.frontend (multi-stage)
- compose.dev.yml (local) + compose.vps-dev/staging/prod.yml (VPS)
- Roles Ansible : podman, firewall, app
- Playbooks Ansible : provision + deploy
- GitHub Actions : ci.yml, deploy-staging.yml, deploy-prod.yml
- Makefile (make up, make deploy-dev, make provision, make test)

**Résultat:** `make up` démarre un stack local fonctionnel. `make provision` configure un VPS de zéro. La CI bloque les PRs qui cassent le build.

**Durée estimée:** ~16h

**Spec:** `docs/plans/2026-03-11-iac-podman-design.md`
**Plan d'implémentation:** `docs/superpowers/plans/2026-03-11-sprint-0-iac.md`
```

**Mise à jour dans TASKS-INDEX.md** — ajouter ✅ sur chaque tâche Sprint 0 :
```markdown
1. `TASK-0.1` - Backend skeleton (main.py + requirements.txt) [30min] ✅ Créé
2. `TASK-0.2` - Containerfile.backend multi-stage [1h] ✅ Créé
...
```

## Critères de validation

- `sprints/sprint-0/README.md` liste toutes les livraisons du sprint
- `TASKS-INDEX.md` marque les 17 tâches Sprint 0 avec ✅
- Le milestone Sprint 0 est décrit avec les trois commandes clés : `make up`, `make provision`, `make deploy-dev`
- Les références aux fichiers de spec et de plan sont correctes

## Ressources

- `docs/plans/2026-03-11-iac-podman-design.md` — spec IaC
- `docs/superpowers/plans/2026-03-11-sprint-0-iac.md` — plan d'implémentation
