# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```json
{
  "name": "rechercher_produit",
  "description": "Recherche des produits dont le nom correspond et dont le prix ne dépasse pas prix_max.",
  "parameters": {
    "type": "object",
    "properties": {
      "nom": { "type": "string", "description": "Nom ou mot-clé du produit recherché" },
      "prix_max": { "type": "number", "description": "Prix maximum en euros" }
    },
    "required": ["nom", "prix_max"]
  }
}
```

## Exercice 2

1. **Indexation** : découper chaque document source en passages (chunks) de taille raisonnable.
2. Calculer l'embedding de chaque passage et le stocker dans une base vectorielle avec une référence au document d'origine.
3. **Requête** : au moment d'une question utilisateur, calculer l'embedding de cette question.
4. Rechercher dans la base vectorielle les N passages dont l'embedding est le plus proche (similarité cosinus) de celui de la question.
5. Construire un prompt combinant des instructions système, les passages retrouvés comme contexte, et la question originale.
6. Envoyer ce prompt au LLM, qui génère une réponse ancrée dans les passages fournis, retournée à l'utilisateur (idéalement avec les sources citées).

## Exercice 3

```python
import math

def similarite_cosinus(a, b):
    produit_scalaire = sum(x * y for x, y in zip(a, b))
    norme_a = math.sqrt(sum(x ** 2 for x in a))
    norme_b = math.sqrt(sum(y ** 2 for y in b))
    return produit_scalaire / (norme_a * norme_b)

def document_le_plus_proche(embedding_question, documents_embeddings):
    return max(documents_embeddings, key=lambda doc: similarite_cosinus(embedding_question, doc["embedding"]))
```

## Exercice 4

1. **RAG** — la documentation change chaque semaine ; le RAG permet de toujours interroger la version à jour sans réentraîner quoi que ce soit, contrairement au fine-tuning qui figerait une connaissance datée.
2. **Fine-tuning** — un format très spécifique et inhabituel, exigé de façon parfaitement fiable à très grande échelle, dépasse ce que le prompt engineering peut garantir de façon constante ; le fine-tuning ancre ce comportement directement dans le modèle.
3. **Prompt engineering** — pour un premier prototype simple (résumé en 3 phrases), un bon prompt système suffit largement ; introduire RAG ou fine-tuning serait disproportionné à ce stade.

## Exercice 5

1. **Résumer l'historique ancien** : au-delà d'un certain nombre d'échanges, remplacer les messages les plus anciens par un résumé généré (par un appel LLM dédié) qui condense leur contenu essentiel — réduit fortement la taille tout en conservant le contexte utile, au prix d'une perte de détail et d'un appel LLM supplémentaire pour produire le résumé.
2. **Ne garder qu'une fenêtre glissante des N derniers messages** : tronquer simplement l'historique aux échanges les plus récents — simple et rapide à implémenter, mais perd complètement le contexte des échanges plus anciens, ce qui peut nuire à la cohérence si l'utilisateur y fait référence plus tard dans la conversation.
