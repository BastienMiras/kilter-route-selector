# TASK-2.1 - Téléchargement de la base via BoardLib

**Sprint** : 1
**Estimation** : 30min
**Priorité** : Haute
**Dépendances** : TASK-1.2

## Objectif

Télécharger la base SQLite Kilter Board via boardlib et valider son contenu.

BoardLib remplace entièrement le scraping API manuel : pas d'authentification,
pas de token JWT, pas d'itération sur les IDs. La base complète est disponible
via une seule commande.

## Checklist

- [ ] Confirmer que boardlib est installé (TASK-1.2)
- [ ] Télécharger/mettre à jour `data/kilter.db`
- [ ] Vérifier le nombre de voies téléchargées
- [ ] Explorer les colonnes disponibles dans `climbs`
- [ ] Vérifier la présence des données de layout (frames)

## Téléchargement

```bash
boardlib database kilter data/kilter.db
```

La commande synchronise la base locale avec le serveur officiel Kilter Board.
Si le fichier existe déjà, seules les nouvelles données sont ajoutées.

## Validation du contenu

```bash
# Nombre de voies publiques
sqlite3 data/kilter.db "SELECT COUNT(*) FROM climbs WHERE is_listed = 1;"

# Distribution par grade
sqlite3 data/kilter.db "
  SELECT difficulty, COUNT(*) as count
  FROM climbs
  WHERE is_listed = 1
  GROUP BY difficulty
  ORDER BY difficulty;
"

# Vérifier la présence du champ frames (layout des prises)
sqlite3 data/kilter.db "SELECT uuid, name, frames FROM climbs LIMIT 3;"
```

## Script Python de validation

**validate_db.py**
```python
import asyncio
import aiosqlite

async def validate():
    async with aiosqlite.connect("data/kilter.db") as db:
        db.row_factory = aiosqlite.Row

        # Nombre total
        async with db.execute("SELECT COUNT(*) as n FROM climbs WHERE is_listed = 1") as cur:
            row = await cur.fetchone()
            print(f"Voies publiques : {row['n']}")

        # Exemple de voie avec layout
        async with db.execute(
            "SELECT uuid, name, difficulty, frames FROM climbs WHERE frames IS NOT NULL LIMIT 1"
        ) as cur:
            row = await cur.fetchone()
            if row:
                print(f"Exemple : {row['name']} (diff={row['difficulty']})")
                print(f"  Frames : {row['frames'][:60]}...")
            else:
                print("Aucune voie avec frames trouvée")

asyncio.run(validate())
```

## Resultats attendus

- Plusieurs milliers de voies listées
- Champ `frames` contient le layout des prises (ex: `p1083r15p1117r15...`)
- Distribution de grades visible (V0 à V17)

## Mise a jour periodique

La base peut être resynchronisée régulièrement :
```bash
# Cron ou script de maintenance
boardlib database kilter data/kilter.db
```

Fréquence recommandée : 1 fois par jour ou par semaine selon le volume de nouvelles voies.

## Ressources

- [BoardLib GitHub](https://github.com/lemeryfertitta/BoardLib)
- [Climbdex - exemple d'usage](https://github.com/lemeryfertitta/Climbdex)
