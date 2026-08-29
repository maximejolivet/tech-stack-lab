# Solutions — Niveau 3 (Avancé)

## Exercice 1

C'est une injection de prompt : le texte censé être une **donnée** à résumer (l'avis client, non fiable car récupéré depuis une source externe) contient en réalité des **instructions** qui tentent de détourner le comportement du modèle, en exploitant le fait que le modèle ne distingue pas nativement "instruction légitime du système" et "texte à traiter" une fois tout concaténé dans le même prompt.

Mesures : (1) séparer clairement, dans la construction du prompt, les instructions système (fixes, contrôlées par le développeur) du contenu externe non fiable (à traiter uniquement comme donnée, jamais comme instruction), et rappeler explicitement dans le prompt système que le contenu des avis ne doit jamais être interprété comme une instruction ; (2) ne jamais donner au modèle, dans ce contexte, un accès réel à des informations sensibles (clé API, données internes) qu'une injection pourrait tenter d'extraire — principe du moindre privilège appliqué au contexte fourni au modèle.

## Exercice 2

```text
iteration = 0
max_iterations = 5
historique = [message_utilisateur]

tant que iteration < max_iterations:
    reponse = appeler_llm(historique, outils_disponibles)
    si reponse.contient_appel_outil:
        resultat = executer_outil(reponse.appel_outil)
        historique.ajouter(reponse)
        historique.ajouter(resultat_comme_message)
        iteration += 1
    sinon:
        retourner reponse.texte  # réponse finale directe
        arreter

si iteration == max_iterations:
    retourner "Je n'ai pas pu terminer cette tâche dans le temps imparti."
```

La limite est nécessaire pour éviter qu'un comportement inattendu du modèle (ex. il rappelle le même outil en boucle sans jamais converger vers une réponse finale) ne consomme des ressources indéfiniment (coût, latence, voire blocage du système appelant) — c'est un garde-fou de robustesse, pas seulement de coût.

## Exercice 3

Router selon la complexité estimée de la requête : par exemple, classifier d'abord la requête (via une règle simple — longueur attendue de la réponse, présence de mots-clés type "rapport", "rédige", "analyse en détail" — ou via un appel préalable très léger à un modèle rapide chargé uniquement de cette classification), puis envoyer les questions factuelles courtes vers le modèle léger/rapide, et les demandes de rédaction complexe vers le modèle plus capable. Le critère de décision : la longueur/complexité de sortie attendue et l'exigence de qualité de raisonnement, pas la longueur de la question elle-même (une question courte peut appeler une réponse complexe).

## Exercice 4

Après génération, exécuter une vérification automatique qui compare la réponse aux documents source fournis en contexte : par exemple, un second appel LLM dédié ("LLM-as-judge") à qui l'on soumet la réponse générée et les documents sources, avec pour instruction explicite de signaler toute affirmation de la réponse qui n'est pas directement soutenue par les documents fournis. Si des affirmations non soutenues sont détectées, soit la réponse est retournée avec un avertissement explicite, soit elle est rejetée et remplacée par une réponse par défaut ("je ne trouve pas cette information dans les documents disponibles") plutôt que d'être affichée telle quelle à l'utilisateur.

## Exercice 5

Constituer un jeu de test de questions représentatives avec leurs réponses attendues (ou au minimum les passages sources qui devraient être retrouvés), puis mesurer séparément : (1) la **qualité de la récupération (retrieval)** — le pipeline retrouve-t-il effectivement les bons passages pour chaque question, indépendamment de la génération ; (2) la **qualité de la génération** — étant donné les bons passages, la réponse générée est-elle correcte et bien ancrée dans ces passages (via une évaluation humaine sur échantillon ou un LLM-as-judge). Distinguer ces deux dimensions permet de savoir si un problème de qualité vient de la recherche de documents ou de la génération de la réponse, plutôt que de juger uniquement la réponse finale de façon globale.
