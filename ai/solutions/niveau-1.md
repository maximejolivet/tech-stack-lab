# Solutions — Niveau 1 (Bases)

## Exercice 1

```python
response = client.messages.create(
    model="claude-sonnet-5",
    max_tokens=10,
    system="Tu réponds uniquement par 'oui' ou 'non', sans aucun autre mot.",
    messages=[{"role": "user", "content": "Le ciel est-il vert ?"}],
)
print(response.content[0].text)  # "non"
```

## Exercice 2

Le code frontend s'exécute intégralement dans le navigateur de l'utilisateur, où n'importe qui peut inspecter le code source ou le trafic réseau et récupérer la clé — elle serait alors utilisable par n'importe qui, aux frais du propriétaire du compte. La clé doit vivre côté **backend** (variable d'environnement, jamais commitée), le frontend appelant un endpoint de son propre serveur qui, lui, détient la clé et appelle le fournisseur LLM.

## Exercice 3

```python
messages = [
    {"role": "system", "content": "Classe chaque email en 'spam' ou 'légitime'."},
    {"role": "user", "content": "Félicitations, vous avez gagné 1000000€ ! Cliquez ici immédiatement."},
    {"role": "assistant", "content": "spam"},
    {"role": "user", "content": "Bonjour, pouvez-vous me confirmer l'horaire de notre réunion de demain ?"},
    {"role": "assistant", "content": "légitime"},
    {"role": "user", "content": "Votre colis est bloqué en douane, payez 2€ de frais pour le débloquer : lien-suspect.com"},
]
```

## Exercice 4

```json
{
  "type": "object",
  "properties": {
    "titre": { "type": "string" },
    "date": { "type": "string" },
    "lieu": { "type": "string" }
  },
  "required": ["titre", "date", "lieu"]
}
```

## Exercice 5

Coût entrée : 2000 tokens × (0,15 $ / 1 000 000) = 0,0003 $
Coût sortie : 500 tokens × (0,60 $ / 1 000 000) = 0,0003 $
Coût total de l'appel : **0,0006 $** (soit 0,06 centime).
