# TASK-2.2 - Explorer endpoints disponibles

**Sprint** : 1  
**Estimation** : 2h  
**Priorité** : Moyenne  
**Dépendances** : TASK-2.1

## 🎯 Objectif

Identifier tous les endpoints disponibles de l'API Kilter Board et documenter leur usage.

## 📋 Checklist

- [ ] Tester GET `/v1/climbs/{id}`
- [ ] Tester GET `/v1/circuits/{id}`
- [ ] Chercher endpoint de listing (paginated)
- [ ] Tester filtres disponibles (grade, setter, etc.)
- [ ] Documenter rate limits observés
- [ ] Identifier endpoints pour setters, users, etc.

## 💻 Tests à effectuer

**1. Récupération d'une voie**
```bash
curl -X GET https://api.kilterboardapp.com/v1/climbs/12345 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**2. Recherche de circuits**
```bash
curl -X GET https://api.kilterboardapp.com/v1/circuits/789 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

**3. Tentative de listing**
```bash
# Tester différentes variations
curl https://api.kilterboardapp.com/v1/climbs?limit=10
curl https://api.kilterboardapp.com/v1/climbs?page=1
curl https://api.kilterboardapp.com/v1/climbs?grade=V5
```

## 💻 Script d'exploration

**explore_api.py**
```python
import asyncio
import httpx
from kilter_api import KilterAPI

async def explore_endpoints(api: KilterAPI):
    await api.login()
    headers = api.get_headers()
    
    endpoints_to_test = [
        "/climbs/1",
        "/climbs/100",
        "/climbs/1000",
        "/circuits/1",
        "/users/me",
        "/setters",
        "/grades",
    ]
    
    async with httpx.AsyncClient() as client:
        for endpoint in endpoints_to_test:
            url = f"{api.BASE_URL}{endpoint}"
            try:
                response = await client.get(url, headers=headers)
                print(f"✅ {endpoint}: {response.status_code}")
                if response.status_code == 200:
                    print(f"   Sample: {str(response.json())[:200]}...")
            except Exception as e:
                print(f"❌ {endpoint}: {e}")
            
            await asyncio.sleep(1)  # Rate limiting
```

## 📝 Documentation des découvertes

Créer un fichier `API_ENDPOINTS.md` avec :

```markdown
# Kilter Board API Endpoints

## Base URL
https://api.kilterboardapp.com/v1

## Authentification
POST /logins
- Retourne Bearer token

## Voies
GET /climbs/{id}
- Récupère une voie par ID
- Paramètres: aucun
- Retourne: objet climb complet

## Circuits
GET /circuits/{id}
- Récupère un circuit/session

## Rate Limits
- Observé: X req/sec
- Headers: X-RateLimit-Remaining, X-RateLimit-Reset

## Notes
- Pas de pagination apparente sur /climbs
- Filtres non documentés
```

## ✅ Critères de validation

- Au moins 5 endpoints testés
- Structure JSON documentée pour chaque endpoint réussi
- Rate limits identifiés
- Fichier `API_ENDPOINTS.md` créé

## 📝 Notes

- Utiliser Burp Suite ou mitmproxy pour intercepter l'app mobile si besoin
- Noter les headers de réponse (rate-limit, cache, etc.)
- Identifier les patterns d'IDs (séquentiels ? UUIDs ?)

## 🔗 Ressources

- [httpx AsyncClient](https://www.python-httpx.org/async/)
- [Postman API Testing](https://www.postman.com/)
