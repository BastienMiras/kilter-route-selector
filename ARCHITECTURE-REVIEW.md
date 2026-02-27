# Review Architecture – Kilter Route Selector

> **Pour qui** : Product Owner junior ou développeur débutant sur ce projet
> **Pourquoi ce document** : Avant d'écrire la première ligne de code, plusieurs hypothèses de départ méritent d'être challengées. Ce document explique les problèmes identifiés, **pourquoi** ils posent problème, et ce qu'on propose à la place.

---

## 🔑 Le principe de base : ne pas réinventer la roue

Quand on commence un projet, il est tentant de tout construire from scratch. Mais souvent, des outils ou des données existent déjà. Dans notre cas, **tout le travail de "connexion à l'API Kilter" est déjà fait** par une librairie Python open-source appelée **BoardLib**.

### Ce que BoardLib fait déjà
- Se connecter à l'API Kilter
- Télécharger **toute** la base de données des voies publiques en une seule commande
- Mettre à jour (sync) avec les nouvelles voies

```bash
# Une seule commande = ~200,000 voies téléchargées
boardlib database kilter kilter.db
```

**Notre plan actuel** prévoit d'écrire un "scraper" maison qui télécharge une voie à la fois, à raison d'une requête par seconde.

> **Analogie** : C'est comme vouloir recopier un livre à la main, alors qu'on peut l'acheter en librairie. Résultat : la version maison prendrait **55 heures** là où BoardLib prend quelques minutes.

---

## 📊 Problème 1 : On a sous-estimé la taille du problème (×200)

**Ce qu'on a planifié** : 1,000 voies en base de données comme première milestone.
**La réalité** : Il y a **200,000+ voies** publiques sur Kilter Board.

| Paramètre | Ce qu'on planifiait | La réalité |
|-----------|---------------------|------------|
| Voies totales | 1,000 | ~200,000 |
| Temps scraping | ~17 minutes | ~55 heures |
| Taille base de données | Très petite | Significative |

**Impact** : Les estimations de performance ("réponse API < 500ms"), les choix d'index de base de données, et les ressources serveur sont tous calibrés sur un volume ×200 trop faible.

> **Analogie** : Comme dimensionner un parking pour 50 voitures alors que le bâtiment accueillera 10,000 personnes.

---

## 🔴 Problème 2 : Un bug fondamental dans la lecture des données

Le format de layout d'une voie Kilter ressemble à ça : `p1083r15p1117r12p1164r13`

**Ce qu'on a compris** : `p{position}r{radius}` → `r15` = rayon de la prise (sa taille)
**La réalité** : `r` = **role code** (type et couleur de la prise)

| Code | Couleur LED | Signification |
|------|-------------|---------------|
| `r12` | Bleu | Départ (start) |
| `r13` | Vert | Pied seul (foot only) |
| `r14` | Violet | Main + pied |
| `r15` | Jaune | Arrivée (finish) |

Ce n'est pas un rayon, c'est le **type de prise**. Or, notre schéma de base de données a une colonne `radius INT` qui stockera des données sans sens.

**Impact** :
- La colonne `radius` dans la base de données doit s'appeler `role_code`
- Les algorithmes qui utilisent "le rayon" produiront des résultats absurdes
- `is_start` et `is_finish` qu'on allait calculer séparément sont directement dérivables du role code — pas besoin de les parser en double

> **Analogie** : Comme confondre le numéro de siège dans un avion avec sa position (fenêtre, couloir, milieu) — ce sont deux informations différentes codées au même endroit.

---

## 🔴 Problème 3 : Les grades de difficulté ne peuvent pas être comparés

**Notre plan** : Stocker le grade comme texte (`grade VARCHAR`) ex: `"V5"`, `"V10"`
**Le problème** : En informatique, les textes se comparent lettre par lettre (ordre alphabétique).

```
"V10" < "V2"   →  VRAI  (en comparaison de texte : "1" vient avant "2")
"V10" < "V2"   →  FAUX  (en réalité : V10 est bien plus dur que V2)
```

Un filtre "voies entre V4 et V6" retournerait des résultats incorrects — V10 et V11 passeraient le filtre.

**Solution** : Ajouter une colonne `grade_numeric INT` (V0=0, V1=1, ..., V17=17) pour les comparaisons. Garder `grade TEXT` uniquement pour l'affichage.

---

## 🟡 Problème 4 : Un bug silencieux dans le Session Builder

Le code prévu du session builder contient une erreur discrète. La logique voulue est : *éviter de mettre deux voies du même setter dans une session*. Mais le code écrit ne le fait pas :

```python
used_setters = set()

for climb in candidates:
    setter = climb['setter']

    used_setters.add(setter)     # On note le setter...
    session.append(climb)        # ...mais on ajoute la voie QUOI QU'IL ARRIVE
```

Il manque la condition de rejet :
```python
    if setter not in used_setters:  # ← cette ligne est absente
        session.append(climb)
```

La diversité de setters est *enregistrée* mais jamais *appliquée*. La feature est décrite dans la documentation mais le code ne la réalise pas — c'est un bug silencieux qui ne plantera pas mais livrera un résultat incorrect.

---

## 🟡 Problème 5 : Les scores de style sont mathématiquement incohérents

Notre algorithme de scoring :
```python
score_dynamic = avg_distance * 2 + max_reach * 3 - density * 1
```

**Le problème** : Ces trois valeurs ne sont pas dans la même unité ni sur la même échelle.

- `avg_distance` ≈ 50 pixels → contribution : `50 × 2 = 100`
- `density` ≈ 0.001 prises/px² → contribution : `0.001 × 1 = 0.001`

La densité contribue donc **100,000× moins** que la distance, peu importe son coefficient. On croit pondérer l'algorithme, mais en pratique `density` n'a aucun impact.

> **Analogie** : Comparer des euros et des centimes sans conversion. `100€ × 2 = 200` vs `50 centimes × 3 = 1.50` — le deuxième terme est négligeable même avec un coefficient plus grand.

**Solution** : Normaliser chaque métrique sur une échelle commune (0 à 1) avant de les combiner. Ainsi les coefficients auront le sens voulu.

---

## 🟡 Problème 6 : L'angle du mur est stocké mais jamais utilisé

Une voie cotée V5 à **40° d'inclinaison** n'est pas du tout la même chose qu'une V5 à **50°**. Plus le mur est déversant, plus c'est difficile. Le style (dynamique, technique) varie aussi avec l'angle.

Notre schéma stocke correctement `angle INT` dans la table des voies, mais **aucune métrique de style n'en tient compte**. Les scores seront donc identiques pour deux voies au même grade mais à des angles très différents.

---

## 🔵 Question d'architecture : PostgreSQL est-il nécessaire ?

**Notre plan** : PostgreSQL — une base de données "enterprise" robuste
**La réalité du besoin** : Les données Kilter sont déjà au format SQLite (c'est ce que télécharge BoardLib).

| Critère | SQLite | PostgreSQL |
|---------|--------|-----------|
| Installation | Zéro — un simple fichier | Docker + serveur séparé à gérer |
| Performances (200k voies) | Très bonnes en lecture | Légèrement meilleures |
| Opérations | Aucune maintenance | Backups, tunning, monitoring |
| Compatibilité BoardLib | Natif | Conversion nécessaire |
| Justifié quand ? | MVP, usage personnel | Beaucoup d'utilisateurs simultanés |

PostgreSQL a été mentionné pour "PostGIS (requêtes spatiales)" — mais PostGIS n'apparaît dans aucun des 5 sprints planifiés. La raison d'être de PostgreSQL n'est donc pas concrétisée dans le plan.

**Recommandation** : Démarrer avec SQLite. Migrer vers PostgreSQL si et seulement si la concurrence d'accès le justifie.

---

## 🔵 Question d'architecture : Flutter est-il nécessaire ?

**Notre plan** : Application native Flutter (Android + Desktop — Dart, compilation native)
**La confirmation** : Le projet n'a pas besoin de contrôler les LEDs Bluetooth — un **deep-link** vers l'app officielle suffit.

Sans contrôle Bluetooth natif, Flutter n'apporte pas de valeur unique ici. Une **Progressive Web App (PWA)** — un site web qu'on peut "installer" comme une app — fonctionne sur tous les appareils et est 3× plus rapide à développer.

De plus, un concurrent direct existe déjà : **Climbdex** ([climbdex.com](https://climbdex.com)), une PWA open-source maintenue par le créateur de BoardLib qui fait déjà du filtrage de voies Kilter.

Le vrai différenciateur de notre projet n'est pas la plateforme : c'est le **session builder intelligent** et les **métriques de style calculées automatiquement**. Ces fonctionnalités fonctionnent tout aussi bien dans une PWA.

---

## 🔵 Question d'architecture : Celery + Redis pour un simple cron job ?

**Notre plan** : Celery (gestionnaire de tâches asynchrones) + Redis (file d'attente de messages) pour le scraping
**Le vrai besoin** : Déclencher une synchro une fois par jour

Ce besoin se résout avec une seule ligne de configuration système (cron) :
```bash
0 3 * * *  boardlib database kilter /data/kilter.db
# = "tous les jours à 3h du matin, synchro Kilter"
```

Celery + Redis = 2 services supplémentaires à installer, configurer, monitorer et maintenir — pour un besoin qui est au fond *"lancer un script une fois par jour"*.

---

## 🗺️ Architecture recommandée vs architecture planifiée

### Ce qui était planifié
```
API Kilter (par ID, 1 req/sec)
    → Scraper maison
    → Redis (file de tâches)
    → Celery (worker)
    → PostgreSQL
    → FastAPI
    → Flutter (Android + Desktop)
```
**Nombre de services à gérer** : 6+

### Ce qui est recommandé
```
boardlib database kilter kilter.db  (cron quotidien, 1 commande)
    → SQLite (un fichier)
    → FastAPI
    → PWA web
```
**Nombre de services à gérer** : 2

**Résultat pour l'utilisateur final** : identique. Complexité opérationnelle : 3× inférieure.

---

## ✅ Résumé des actions prioritaires

| Priorité | Action | Pourquoi |
|----------|--------|----------|
| 🔴 Critique | Remplacer le scraper par BoardLib | 55h → quelques minutes, même résultat |
| 🔴 Critique | `r{code}` = role type, pas rayon | Bug de compréhension fondamental du format |
| 🔴 Critique | Ajouter `grade_numeric INT` en base | Filtrage par grade cassé avec VARCHAR |
| 🔴 Critique | Corriger setter diversity (condition manquante) | Feature silencieusement non fonctionnelle |
| 🟡 Important | Normaliser les métriques avant scoring | Poids sans signification sinon |
| 🟡 Important | Trier les prises avant calcul métriques | Ordre arbitraire dans le layout → métriques fausses |
| 🟡 Important | Gérer l'expiration du token JWT | Scraper planterait silencieusement en prod |
| 🟡 Important | Intégrer l'angle dans les scores de style | Un V5 à 40° ≠ V5 à 50° |
| 🔵 Architecture | SQLite pour le MVP | Même résultat, 0 service supplémentaire |
| 🔵 Architecture | PWA plutôt que Flutter | BLE non requis, 3× moins de code |
| 🔵 Architecture | Cron job plutôt que Celery/Redis | 1 ligne de config vs 2 services |

---

## 📚 Ressources identifiées pendant l'analyse

- [BoardLib](https://github.com/lemeryfertitta/BoardLib) — librairie Python officieuse qui wrappe l'API Kilter, maintenue activement
- [Climbdex](https://github.com/lemeryfertitta/Climbdex) — PWA de filtrage de voies par le même auteur
- [Dataset Hugging Face KilterBoard](https://huggingface.co/datasets/Vilin97/KilterBoard) — 100k-1M voies déjà exportées et disponibles
- [Blog reverse-engineering Kilter](https://bazun.me/blog/kiterboard) — documentation du format BLE et du format layout

---

*Document rédigé le 2026-02-27 — basé sur une analyse des specs et recherches sur l'écosystème Kilter existant.*
