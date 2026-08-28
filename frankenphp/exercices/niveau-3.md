# Niveau 3 — Avancé

## Exercice 3.1 — Détection de fuite mémoire

Écris un worker qui accumule volontairement des données dans un tableau statique à chaque requête sans jamais le vider (fuite mémoire simulée). Mesure/observe l'augmentation de la mémoire du processus au fil des requêtes (via `memory_get_usage()` loggé à chaque itération). Ajoute ensuite `max_requests` dans la configuration et explique comment cette limite atténue le problème sans le corriger réellement.

## Exercice 3.2 — Migration classic → worker

Prends une petite application PHP native (routeur fait maison ou mini-projet d'un autre dossier) fonctionnant en mode classic sous FrankenPHP. Identifie tout l'état global qu'elle utilise (variables statiques, singletons, connexions DB), documente les changements nécessaires pour la rendre compatible avec le mode worker, puis applique-les.

## Exercice 3.3 — Décision d'architecture

Rédige une courte note de décision technique (format ADR) : pour un projet Symfony avec un existant important, dans quelles conditions passer en mode worker apporterait un bénéfice réel, et dans quelles conditions le risque (état partagé mal maîtrisé, code legacy) dépasse le bénéfice de performance. Conclus par une recommandation.
