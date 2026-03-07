# TASK-2.2 - Explorer le schéma SQLite boardlib

**Sprint** : 1
**Estimation** : 1h
**Priorité** : Moyenne
**Dépendances** : TASK-2.1

## Objectif

Explorer et documenter le schéma complet de la base SQLite Kilter Board
téléchargée via boardlib, afin de comprendre quelles données sont disponibles
et comment les utiliser dans nos requêtes.

## Checklist

- [ ] Lister toutes les tables disponibles
- [ ] Explorer le schema de la table `climbs`
- [ ] Identifier les colonnes utiles (grade, layout, popularité, etc.)
- [ ] Explorer les tables de référence (layouts, placements, etc.)
- [ ] Documenter les colonnes et leur signification
- [ ] Créer `docs/DB_SCHEMA.md` avec le résumé

## Exploration avec sqlite3

**1. Liste des tables**
```bash
sqlite3 data/kilter.db ".tables"
```

**2. Schema de chaque table**
```bash
sqlite3 data/kilter.db ".schema"
```

**3. Table climbs en détail**
```bash
sqlite3 data/kilter.db "PRAGMA table_info(climbs);"
```

**4. Données d'exemple**
```bash
# Une voie complète
sqlite3 data/kilter.db "SELECT * FROM climbs WHERE is_listed = 1 LIMIT 1;"

# Distribution des grades
sqlite3 data/kilter.db "
SELECT difficulty, COUNT(*) as n
FROM climbs WHERE is_listed = 1
GROUP BY difficulty ORDER BY difficulty;
"

# Top voies par popularité
sqlite3 data/kilter.db "
SELECT name, difficulty, ascensionist_count, quality_average
FROM climbs WHERE is_listed = 1
ORDER BY ascensionist_count DESC LIMIT 10;
"
```

## Script Python d'exploration

**explore_schema.py**
```python
import asyncio
import aiosqlite

DB_PATH = "data/kilter.db"

async def explore():
    async with aiosqlite.connect(DB_PATH) as db:
        db.row_factory = aiosqlite.Row

        # Tables disponibles
        async with db.execute("SELECT name FROM sqlite_master WHERE type='table'") as cur:
            tables = [row[0] async for row in cur]
        print("Tables:", tables)

        # Colonnes de climbs
        async with db.execute("PRAGMA table_info(climbs)") as cur:
            cols = await cur.fetchall()
        print("\nColonnes climbs:")
        for col in cols:
            print(f"  {col['name']} ({col['type']})")

        # Stats globales
        async with db.execute("SELECT COUNT(*) FROM climbs WHERE is_listed = 1") as cur:
            n = (await cur.fetchone())[0]
        print(f"\nVoies publiques : {n}")

        # Voie avec frames non vide
        async with db.execute(
            "SELECT uuid, name, difficulty, frames FROM climbs WHERE frames IS NOT NULL AND is_listed = 1 LIMIT 3"
        ) as cur:
            rows = await cur.fetchall()
        print("\nExemples de voies avec frames:")
        for row in rows:
            print(f"  {row['name']} diff={row['difficulty']} frames={row['frames'][:50]}...")

asyncio.run(explore())
```

## Documentation a produire

Créer `docs/DB_SCHEMA.md` avec les sections suivantes :

```markdown
# Schema SQLite Kilter Board (boardlib)

## Tables disponibles
...

## Table `climbs`
| Colonne | Type | Description |
|---------|------|-------------|
| uuid | TEXT | Identifiant unique |
| name | TEXT | Nom de la voie |
| setter_username | TEXT | Ouvreur |
| difficulty | INTEGER | Grade (valeur numérique) |
| frames | TEXT | Layout des prises |
| ascensionist_count | INTEGER | Nombre d'ascensions |
| quality_average | REAL | Note moyenne (/5) |
| is_listed | INTEGER | 1 = voie publique |
| created_at | TEXT | Date de création |
| ... | ... | ... |

## Mapping difficulty → grade V-scale
(à documenter selon les valeurs observées)

## Format frames
`p{position}r{radius}...`
- position → coordonnées via : x = position & 0xff, y = (position >> 8) & 0xff
```

## Criteres de validation

- Toutes les tables listées et documentées
- Colonnes de `climbs` identifiées avec leur signification
- Format du champ `frames` compris
- Fichier `docs/DB_SCHEMA.md` créé

## Ressources

- [BoardLib source](https://github.com/lemeryfertitta/BoardLib)
- [Climbdex models](https://github.com/lemeryfertitta/Climbdex)
