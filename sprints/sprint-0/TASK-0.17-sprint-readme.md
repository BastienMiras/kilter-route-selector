# TASK-0.17 - Sprint 0 README + TASKS-INDEX

**Sprint** : 0
**Estimation** : 30min
**Priorité** : Moyenne
**Dépendances** : TASK-0.1, TASK-0.2, TASK-0.3, TASK-0.4, TASK-0.5, TASK-0.6, TASK-0.7, TASK-0.8, TASK-0.9, TASK-0.10, TASK-0.11, TASK-0.12, TASK-0.13, TASK-0.14, TASK-0.15, TASK-0.16

## Description

La dernière tâche d'un sprint est toujours de documenter ce qui a été accompli et de mettre à jour
le suivi de projet. C'est une étape souvent négligée par les débutants, mais elle est essentielle
pour plusieurs raisons.

Le **README du sprint** (`sprints/sprint-0/README.md`) sert de point d'entrée pour quiconque veut
comprendre ce que contient ce sprint : qu'est-ce qui a été livré, pourquoi, et comment vérifier que
ça fonctionne. Il aide aussi l'équipe à se rappeler du contexte lors de corrections futures. Sans
ce document, quelqu'un qui arrive sur le projet en Sprint 4 devra lire 17 fichiers de tâches pour
comprendre ce que fait l'infrastructure.

Le **TASKS-INDEX.md** est le suivi central de toutes les tâches du projet (120 tâches réparties sur
plusieurs sprints). Marquer les tâches Sprint 0 avec ✅ permet de voir en un coup d'œil l'avancement
global du projet. C'est l'équivalent d'un tableau Kanban en mode markdown.

Cette tâche est aussi l'occasion de vérifier que tout fonctionne de bout en bout avant de passer au
Sprint 1 : `make up` doit démarrer un stack local fonctionnel, le CI doit être actif, et le VPS doit
être provisionnable avec `make provision`. Si l'une de ces vérifications échoue, c'est maintenant
qu'on le corrige — pas au milieu du Sprint 1.

**Comment réaliser cette tâche :**

1. Crée (ou mets à jour) `sprints/sprint-0/README.md` avec un tableau listant les 17 livrables,
   l'architecture résultante, les commandes clés (`make up`, `make provision`, `make deploy-dev`),
   la durée estimée et les références aux fichiers de spec.
2. Ouvre `TASKS-INDEX.md` et ajoute le symbole ✅ devant chacune des 17 tâches du Sprint 0 pour
   indiquer qu'elles sont terminées.
3. Vérifie que les trois jalons du sprint sont bien fonctionnels avant de clore le sprint.

## Objectif

Mettre à jour le README du Sprint 0 pour refléter les livraisons complètes et marquer les tâches Sprint 0 comme terminées dans `TASKS-INDEX.md`.

## Checklist

- [ ] Créer `sprints/sprint-0/README.md` avec le tableau des livraisons et l'architecture déployée
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
