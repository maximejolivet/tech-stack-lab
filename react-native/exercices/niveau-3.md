# Exercices React Native — Niveau 3 (Avancé)

## Exercice 1 — FlatList optimisée

Prends une `FlatList` affichant 500 éléments avec un `renderItem` complexe (plusieurs `View`/`Text` imbriqués). Optimise-la avec `React.memo` sur le composant de ligne et `useCallback` sur `renderItem`. Explique en 3-4 lignes pourquoi ces deux optimisations sont nécessaires ensemble (l'une sans l'autre ne suffit pas toujours).

## Exercice 2 — Bridge JS/natif (conceptuel)

Sans écrire de code natif, explique en 5-6 lignes la différence entre l'ancien pont (bridge, sérialisation JSON asynchrone) et la nouvelle architecture (JSI, appels synchrones) — quel type d'interaction (ex: animation suivant le doigt en temps réel) bénéficie le plus de ce changement, et pourquoi.

## Exercice 3 — Reanimated vs Animated

Réimplémente l'animation de fade-in de l'exercice 2.5 avec `react-native-reanimated` (`useSharedValue`, `useAnimatedStyle`, `withTiming`). Explique en 2-3 lignes la différence de thread d'exécution avec l'API `Animated` de base, et l'impact pratique sur la fluidité si le thread JS est chargé.

## Exercice 4 — Build EAS (conceptuel)

Sans exécuter de build réel, décris en 4-5 lignes les étapes qu'`eas build --platform ios` effectue côté cloud, et pourquoi cette approche permet de builder une app iOS sans posséder de Mac localement.
