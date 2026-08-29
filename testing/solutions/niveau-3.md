# Solutions — Niveau 3 (Avancé)

## Exercice 1

Ce mutant survivant révèle qu'aucun test ne couvre le cas limite exact `age == 18` — la suite ne distingue pas un âge strictement supérieur à 18 d'un âge égal à 18, alors que le code source fait cette distinction. Il manque un test explicite du type `expect(isAdult(18)).toBe(true)` (et idéalement `expect(isAdult(17)).toBe(false)`) qui échouerait avec la mutation `>` mais pas avec le code original.

## Exercice 2

Un test d'intégration bout-en-bout nécessiterait de faire tourner les deux services réels à chaque run de CI, ce qui est lent, fragile (dépend de la disponibilité et de l'état de `users-api`), et couple artificiellement le cycle de CI des deux équipes. Un test de contrat élimine ce couplage : `orders-api` définit et vérifie qu'il consomme correctement un contrat (format de requête/réponse attendu) sans faire tourner `users-api`, pendant que `users-api` vérifie indépendamment, dans sa propre CI, qu'il continue de respecter ce même contrat. Chaque équipe peut ainsi faire évoluer son service en confiance, sans environnement partagé ni synchronisation de déploiement pour lancer les tests.

## Exercice 3

- **Involution** : `reverse(reverse(list))` doit toujours être égal à `list`, quelle que soit la liste générée.
- **Conservation de la longueur** : `reverse(list).length` doit toujours être égal à `list.length`.

(Autres propriétés valables : conservation des éléments, ex. même multiset d'éléments avant/après.)

## Exercice 4

Un test fonctionnel vérifie la **correction** d'une seule réponse dans des conditions normales (le bon code HTTP, le bon corps de réponse) — son critère de succès est binaire (correct/incorrect). Un test de charge vérifie le **comportement du système sous contrainte** (latence, taux d'erreur, utilisation ressources) avec de nombreuses requêtes simultanées — son critère de succès est un seuil statistique (ex. "95e percentile de latence sous 200ms", "taux d'erreur sous 0,1%"), et il peut révéler des problèmes (fuite de connexions, contention de ressources) totalement invisibles dans un test fonctionnel exécuté seul.

## Exercice 5

Un test flaky ignoré érode la confiance de l'équipe non seulement dans ce test précis, mais dans le signal donné par un échec de pipeline en général : les développeurs prennent l'habitude de relancer systématiquement un run rouge en espérant qu'il passe, au lieu d'investiguer chaque échec. À terme, un vrai échec (une régression réelle) risque d'être traité de la même façon — relancé et ignoré — ce qui annule l'intérêt même de la CI comme garde-fou, et le flaky test non corrigé finit par masquer de vrais bugs plutôt que de simplement gêner.
