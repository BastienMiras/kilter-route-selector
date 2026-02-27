# TASK-2.1 - Tester authentification Kilter Board API

**Sprint** : 1  
**Estimation** : 1h  
**Priorité** : Haute  
**Dépendances** : Aucune

## 🎯 Objectif

Valider que l'authentification fonctionne avec l'API Kilter Board et récupérer un token valide.

## 📋 Checklist

- [ ] Créer compte de test Kilter Board
- [ ] Tester endpoint `/v1/logins` manuellement (curl/Postman)
- [ ] Implémenter fonction `login()` en Python
- [ ] Stocker token dans variable d'environnement
- [ ] Documenter format de réponse

## 💻 Test manuel avec curl

```bash
curl -X POST https://api.kilterboardapp.com/v1/logins \
  -H "Content-Type: application/json" \
  -d '{
    "username": "YOUR_USERNAME",
    "password": "YOUR_PASSWORD",
    "tou": "accepted",
    "pp": "accepted"
  }'
```

**Réponse attendue :**
```json
{
  "login": {
    "token": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user_id": 12345,
    "username": "YOUR_USERNAME"
  }
}
```

## 💻 Implémentation Python

**kilter_api.py**
```python
import httpx
from typing import Optional

class KilterAPI:
    BASE_URL = "https://api.kilterboardapp.com/v1"
    
    def __init__(self, username: str, password: str):
        self.username = username
        self.password = password
        self.token: Optional[str] = None
    
    async def login(self) -> str:
        """Authenticate and return Bearer token"""
        async with httpx.AsyncClient() as client:
            response = await client.post(
                f"{self.BASE_URL}/logins",
                json={
                    "username": self.username,
                    "password": self.password,
                    "tou": "accepted",
                    "pp": "accepted"
                }
            )
            response.raise_for_status()
            data = response.json()
            self.token = data["login"]["token"]
            return self.token
    
    def get_headers(self) -> dict:
        """Return headers with Bearer token"""
        if not self.token:
            raise ValueError("Not authenticated. Call login() first.")
        return {"Authorization": self.token}
```

## 💻 Test d'intégration

**test_auth.py**
```python
import asyncio
import os
from kilter_api import KilterAPI

async def test_auth():
    username = os.getenv("KILTER_USERNAME")
    password = os.getenv("KILTER_PASSWORD")
    
    if not username or not password:
        print("❌ Set KILTER_USERNAME and KILTER_PASSWORD env vars")
        return
    
    api = KilterAPI(username, password)
    
    try:
        token = await api.login()
        print(f"✅ Login successful!")
        print(f"Token (first 50 chars): {token[:50]}...")
    except Exception as e:
        print(f"❌ Login failed: {e}")

if __name__ == "__main__":
    asyncio.run(test_auth())
```

## ✅ Critères de validation

- Login réussit sans erreur
- Token JWT récupéré (commence par "Bearer eyJ...")
- Token peut être réutilisé dans headers
- Erreurs gérées (mauvais credentials, API down)

## 📝 Notes

- Ne jamais committer les credentials
- Utiliser `.env` pour stocker username/password
- Token a probablement une expiration → prévoir refresh

## 🔗 Ressources

- [httpx Documentation](https://www.python-httpx.org/)
- [JWT.io](https://jwt.io/) pour décoder le token
