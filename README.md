# tech-stack-lab

Base de connaissances personnelle pour un développeur Full Stack (PHP / JavaScript) qui souhaite **reprendre les fondamentaux de façon structurée**, combler ses lacunes et progresser jusqu'aux notions avancées, techno par techno.

Ce README est le point d'entrée : il décrit l'architecture du repo, la roadmap des technologies à couvrir et l'ordre recommandé pour les revoir. Le contenu de chaque technologie est généré **progressivement**, une techno à la fois, sur demande.

---

## Légende de progression

Chaque chapitre d'un README de technologie est marqué par un niveau :

| Niveau | Signification |
|---|---|
| 🟢 Fondamentaux | Bases indispensables, à maîtriser à 100% |
| 🟡 Intermédiaire | Notions courantes en usage professionnel quotidien |
| 🟠 Avancé | Architecture, performance, patterns |
| 🔴 Expertise | Scalabilité, concurrence, optimisation fine, sujets de spécialiste |

---

## Architecture du repository

```text
tech-stack-lab/
├── README.md                      # ce fichier : architecture, roadmap, ordre
│
├── git/
├── html/
├── css/
├── javascript/
├── typescript/
├── php/                             # PHP natif (langage, sans framework)
│
├── design-patterns/                # SOLID, Clean Code, patterns transverses
│
├── nodejs/
├── symfony/
├── laravel/
├── frankenphp/                      # runtime PHP moderne
├── java/
├── springboot/
├── kotlin/                          # langage JVM alternatif, comparaison avec Java
├── python/
├── django/
│
├── vuejs/
├── nuxtjs/
├── react/
├── angular/
│
├── react-native/                    # mobile (JS)
├── flutter/                         # mobile (Dart)
│
├── wordpress/
├── drupal/
│
├── tailwindcss/
│
├── mysql/
├── postgresql/
├── redis/
│
├── linux/
├── docker/
├── kubernetes/
├── github/
├── ci-cd/
│
├── testing/
├── security/
├── api/
├── accessibility-performance/      # a11y + Core Web Vitals
│
├── algorithms-data-structures/
├── system-design/
│
└── ai/
```

Chaque dossier de technologie suit **le même gabarit** afin de garder une base de connaissances cohérente et rapide à consulter, des mois plus tard :

```text
<techno>/
├── README.md
│   1. Introduction
│   2. Prérequis
│   3. Rappel des bases        (🟢)
│   4. Concepts intermédiaires (🟡)
│   5. Concepts avancés        (🟠🔴)
│   6. Commandes / syntaxe à connaître
│   7. Exercices (niveau 1 / 2 / 3)
│   8. Mini-projet
│   9. Checklist
│   10. Ressources
├── exercices/
│   └── ...                    # énoncés
└── solutions/
    └── ...                    # corrections séparées des énoncés
```

---

## Roadmap & ordre de révision recommandé

L'ordre suit une logique de dépendances : les fondamentaux du web d'abord, puis les langages, les principes de conception (transverses à tout ce qui suit), les runtimes et frameworks, les CMS, les données, l'infra/outillage, la qualité/sécurité, puis les sujets théoriques avancés. L'IA est traitée en dernier car elle s'appuie sur des bases solides en développement plutôt que l'inverse.

| # | Dossier | Pourquoi à cette place |
|---|---|---|
| 1 | `git` | Outil de base pour versionner tout ce qui suit, y compris ce repo lui-même |
| 2 | `html` | Fondation de tout rendu web |
| 3 | `css` | Indissociable du HTML pour tout front |
| 4 | `javascript` | Langage central du front, prérequis à tout framework JS |
| 5 | `typescript` | Sur-couche de JS, à voir juste après pour renforcer les bases JS |
| 6 | `php` | Deuxième langage central, **PHP natif** (ton cœur de métier backend), avant tout framework |
| 7 | `design-patterns` | Principes transverses (SOLID, Clean Code) à connaître avant d'attaquer les frameworks, pour mieux comprendre leurs choix d'architecture |
| 8 | `nodejs` | Runtime JS backend, prérequis pour Vue/Nuxt/React tooling |
| 9 | `symfony` | Framework PHP structurant, référence entreprise |
| 10 | `laravel` | Framework PHP orienté productivité, écosystème très utilisé |
| 11 | `frankenphp` | Runtime PHP moderne (embarque PHP dans un binaire Go, alternative à Nginx+PHP-FPM), vu après Symfony/Laravel qu'il sert à exécuter/déployer |
| 12 | `java` | Langage backend alternatif typé statiquement, comparaison utile avec PHP avant d'attaquer Spring Boot |
| 13 | `springboot` | Framework Java de référence en entreprise, équivalent Symfony/Laravel côté JVM |
| 14 | `kotlin` | Langage JVM alternatif à Java, interopérable, vu juste après Java/Spring Boot pour comparer |
| 15 | `python` | Troisième langage backend alternatif, incontournable (scripting, data, IA), syntaxe très différente de PHP/Java à comparer |
| 16 | `django` | Framework Python le plus structurant, équivalent Symfony/Spring côté Python ("batteries included") |
| 17 | `vuejs` | Framework front progressif, bon pont après JS/TS |
| 18 | `nuxtjs` | Sur-couche full-stack de Vue (SSR, routing, etc.) |
| 19 | `react` | Autre grand framework front, comparaison utile avec Vue |
| 20 | `angular` | Troisième framework front, plus structurant/opinionated (proche de la logique Symfony/Spring côté front) |
| 21 | `react-native` | Développement mobile via React, réutilise directement les bases React |
| 22 | `flutter` | Développement mobile via Dart, paradigme différent (widgets, compilation native) à comparer avec React Native |
| 23 | `wordpress` | CMS PHP le plus répandu, logique différente d'un framework |
| 24 | `drupal` | CMS PHP plus structurant/entreprise |
| 25 | `tailwindcss` | Approche utility-first, à voir une fois le CSS et un framework front maîtrisés |
| 26 | `mysql` | SGBD relationnel de référence en PHP/web |
| 27 | `postgresql` | SGBD relationnel avancé, comparaison avec MySQL |
| 28 | `redis` | Cache / structures en mémoire, complément aux SGBD |
| 29 | `linux` | Environnement d'exécution serveur, prérequis Docker/CI |
| 30 | `docker` | Conteneurisation, standard pour dev et déploiement |
| 31 | `kubernetes` | Orchestration de conteneurs à l'échelle, suite logique de Docker |
| 32 | `github` | Collaboration, PR, Actions, gestion de projet |
| 33 | `ci-cd` | Automatisation build/test/déploiement, s'appuie sur Docker/Kubernetes/GitHub |
| 34 | `testing` | Qualité de code, à appliquer à tout ce qui a été vu |
| 35 | `security` | Sécurité applicative, transverse à toutes les technos précédentes |
| 36 | `api` | Conception d'API (REST/GraphQL), synthèse backend |
| 37 | `accessibility-performance` | a11y + Core Web Vitals, synthèse front |
| 38 | `algorithms-data-structures` | Bases théoriques, utiles pour la performance et les entretiens |
| 39 | `system-design` | Architecture logicielle et scalabilité, niveau avancé/expertise |
| 40 | `ai` | Intégration de l'IA dans des applications web, une fois les bases solides |

---

## Suivi global

- [x] git
- [x] html
- [x] css
- [x] javascript
- [x] typescript
- [x] php
- [x] design-patterns
- [x] nodejs
- [x] symfony
- [x] laravel
- [x] frankenphp
- [x] java
- [x] springboot
- [x] kotlin
- [x] python
- [x] django
- [x] vuejs
- [x] nuxtjs
- [x] react
- [x] angular
- [x] react-native
- [x] flutter
- [x] wordpress
- [x] drupal
- [x] tailwindcss
- [x] mysql
- [x] postgresql
- [x] redis
- [x] linux
- [x] docker
- [x] kubernetes
- [x] github
- [x] ci-cd
- [x] testing
- [x] security
- [x] api
- [x] accessibility-performance
- [x] algorithms-data-structures
- [x] system-design
- [x] ai

---

## Comment ça avance

Le repository n'est **pas généré d'un coup**. Pour chaque technologie de la roadmap, il suffit de la demander (ex. *"génère le dossier javascript"*) : le dossier est créé avec son `README.md` complet (introduction, rappel des bases, concepts intermédiaires/avancés, commandes, exercices, mini-projet, checklist, ressources) ainsi que les fichiers d'exercices et de solutions associés, dans le style et le niveau de détail décrits ci-dessus.

## Méthodologie & inspiration

Le découpage de la roadmap ci-dessus a été confronté aux roadmaps officielles de [roadmap.sh](https://roadmap.sh) (roadmaps *skill-based* : JavaScript, TypeScript, PHP, Node.js, React, Vue, Laravel, HTML, CSS, SQL, Redis, Docker, Linux, Git/GitHub, System Design, API Design...) : la couverture est cohérente avec ce standard de la communauté dev. Lors de la génération de chaque `README.md` de technologie, la roadmap.sh correspondante (quand elle existe) sera consultée pour vérifier qu'aucune notion importante n'est oubliée dans le rappel des bases et les concepts intermédiaires/avancés, et sera ajoutée dans sa section **Ressources**.

Les roadmaps **rôle-based** de roadmap.sh ([Frontend](https://roadmap.sh/frontend), [Backend](https://roadmap.sh/backend), [Full Stack](https://roadmap.sh/full-stack), [DevOps](https://roadmap.sh/devops)) confirment que la structure du repo (dossiers front + back + data + infra) couvre déjà l'ensemble de ces parcours : elles ne donnent donc pas lieu à un nouveau dossier, mais servent de vérification de complétude globale.

Certaines roadmaps plus ciblées viennent enrichir des dossiers déjà prévus plutôt que d'en créer de nouveaux :

| Source roadmap.sh | Dossier(s) enrichi(s) | Utilisation |
|---|---|---|
| [AI Engineer](https://roadmap.sh/ai-engineer) | `ai/` | Élargit le contenu au-delà de l'usage d'outils IA : prompt engineering, RAG, agents, intégration LLM dans une app |
| [Cyber Security](https://roadmap.sh/cyber-security) | `security/` | Ressource complémentaire pour les bases réseau/infra ; le dossier reste centré sécurité applicative (OWASP, auth, API) |
| [Frontend Performance](https://roadmap.sh/frontend-performance-best-practices) | `accessibility-performance/` | Alimente le chapitre performance (Core Web Vitals, chargement, rendu) |
| [Backend Performance](https://roadmap.sh/backend-performance-best-practices) | `php/`, `nodejs/`, `mysql/`, `api/`, `system-design/` | Bonnes pratiques de perf backend référencées dans les chapitres avancés concernés |
| [API Security](https://roadmap.sh/api-security-best-practices) | `api/`, `security/` | Alimente le chapitre sécurité des API (auth, rate limiting, validation) |
| [Code Review](https://roadmap.sh/code-review-best-practices) | `github/`, `testing/` | Alimente les bonnes pratiques de revue de code et de collaboration |

Ces liens seront repris dans la section **Ressources** des dossiers concernés au moment de leur génération.
