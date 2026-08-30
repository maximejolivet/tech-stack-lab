# AWS

## 1. Introduction

AWS (Amazon Web Services) est le fournisseur de cloud public dominant : il loue à la demande du calcul, du stockage et du réseau facturés à l'usage, plutôt que d'acheter et maintenir des serveurs physiques. Ce dossier suppose Docker et Kubernetes acquis (voir [`../docker/`](../docker/), [`../kubernetes/`](../kubernetes/)) — une partie du contenu couvre justement où et comment ces conteneurs tournent réellement en production, en plus des briques cloud (stockage, bases de données, fonctions) qui existent indépendamment de la conteneurisation.

**À quoi sert-il ?**
- Héberger une application (VM, conteneurs, fonctions serverless) sans posséder ni gérer le matériel physique.
- Stocker des fichiers, sauvegardes et assets statiques de façon durable et hautement disponible.
- Faire tourner des bases de données managées, sans administrer soi-même le SGBD sous-jacent.
- Déployer et faire évoluer les images Docker et clusters Kubernetes déjà couverts, à l'échelle d'une infrastructure mondiale.

**Où se situe-t-il dans une architecture web ?** La couche la plus basse de toutes : l'application tourne **sur** l'infrastructure AWS plutôt que sur une machine possédée ou louée en bare-metal. Docker (voir [`../docker/`](../docker/)) empaquette l'application, Kubernetes (voir [`../kubernetes/`](../kubernetes/)) l'orchestre, et AWS fournit les machines, le réseau et les services managés sur lesquels tout cela s'exécute réellement.

**Avantages** : élasticité (scaler à la hausse ou à la baisse en quelques minutes, facturé à l'usage réel), catalogue de services managés immense qui évite de réinventer l'infrastructure (bases de données, files d'attente, CDN...), présence mondiale (regions/AZ) pour rapprocher l'application de ses utilisateurs.
**Limites** : complexité de la facturation (un service mal configuré peut générer une facture disproportionnée sans avertissement), risque de dépendance forte à des services propriétaires (vendor lock-in) qui compliquent une migration future, courbe d'apprentissage raide tant le nombre de services est élevé (plusieurs centaines).

## 2. Prérequis

- Bases Linux (voir [`../linux/`](../linux/)) — la plupart des machines et conteneurs AWS tournent sous Linux.
- Docker et Kubernetes déjà couverts (voir [`../docker/`](../docker/), [`../kubernetes/`](../kubernetes/)) — plusieurs services AWS de ce dossier (ECS, EKS) s'appuient directement dessus.
- Un compte AWS (le [Free Tier](https://aws.amazon.com/free/) couvre une utilisation légère gratuitement pendant 12 mois) et l'AWS CLI installée (`aws --version`).

## 3. Rappel des bases 🟢

### 01 - Régions et zones de disponibilité

**Explication** — Une **région** est une zone géographique autonome (ex. `eu-west-3` = Paris), contenant plusieurs **zones de disponibilité** (Availability Zones, AZ) — des datacenters physiquement séparés au sein de la même région, isolés les uns des autres (alimentation, réseau) pour la tolérance aux pannes.

```bash
aws ec2 describe-regions --output table
aws configure set region eu-west-3
```

**Bonne pratique** : répartir les ressources critiques sur plusieurs AZ d'une même région (jamais une seule) — la panne d'un datacenter ne doit jamais interrompre le service.

### 02 - IAM : utilisateurs, rôles et politiques

**Explication** — IAM (Identity and Access Management) contrôle **qui** peut faire **quoi** sur quelles ressources. Un utilisateur ou un rôle se voit attacher des **politiques** (documents JSON) qui autorisent ou refusent des actions précises.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["s3:GetObject", "s3:PutObject"], "Resource": "arn:aws:s3:::mon-bucket/*" }
  ]
}
```

```bash
aws iam create-user --user-name deploy-bot
aws iam attach-user-policy --user-name deploy-bot --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

**Erreur fréquente** : utiliser le compte **root** (celui créé à l'inscription) ou une politique `"Action": "*", "Resource": "*"` pour le quotidien — le root ne doit servir qu'à la configuration initiale (activer la MFA, créer le premier utilisateur IAM), et toute politique doit suivre le principe du moindre privilège (voir [`../security/`](../security/)).

### 03 - S3 : stockage d'objets

**Explication** — S3 (Simple Storage Service) stocke des fichiers (objets) dans des **buckets**, sans hiérarchie de dossiers réelle (les "/" dans une clé ne sont qu'une convention visuelle) — conçu pour une durabilité et une disponibilité très élevées.

```bash
aws s3 mb s3://mon-bucket-unique-12345
aws s3 cp fichier.txt s3://mon-bucket-unique-12345/
aws s3 ls s3://mon-bucket-unique-12345/
aws s3 cp s3://mon-bucket-unique-12345/fichier.txt ./telecharge.txt
```

**Erreur fréquente** : rendre un bucket public par erreur (accès en lecture/écriture ouvert à tout Internet) — cause régulière de fuites de données médiatisées. Un bucket est **privé par défaut** ; ne l'ouvrir publiquement que pour un cas précis et documenté (ex. assets statiques d'un site), jamais par défaut.

### 04 - EC2 : machines virtuelles

**Explication** — EC2 (Elastic Compute Cloud) loue des machines virtuelles (**instances**), disponibles dans différents **types** (combinaisons CPU/RAM/réseau, ex. `t3.micro` pour un usage léger, `c5.xlarge` pour du calcul intensif).

```bash
aws ec2 run-instances --image-id ami-0abcdef1234567890 --instance-type t3.micro \
  --key-name ma-cle --security-group-ids sg-0123456789abcdef0

aws ec2 describe-instances --query "Reservations[].Instances[].State.Name"
aws ec2 stop-instances --instance-ids i-0123456789abcdef0
```

**Cas d'usage** : une instance EC2 convient à une application qui doit tourner en continu et nécessite un contrôle total sur l'OS — pour un besoin plus ponctuel ou événementiel, voir Lambda (section 4).

### 05 - Security groups : le pare-feu d'une instance

**Explication** — Un security group agit comme un pare-feu virtuel attaché à une instance EC2 (ou un autre service) : il n'autorise **que** les règles entrantes/sortantes explicitement déclarées, tout le reste est bloqué par défaut.

```bash
aws ec2 authorize-security-group-ingress --group-id sg-0123456789abcdef0 \
  --protocol tcp --port 22 --cidr 203.0.113.0/24   # SSH restreint à un bloc d'IP précis
```

**Erreur fréquente** : ouvrir le port 22 (SSH) à `0.0.0.0/0` (tout Internet) — cible immédiate des scans automatisés. Restreindre systématiquement l'accès administratif à une plage d'IP connue (bureau, VPN).

### 06 - VPC : le réseau privé virtuel

**Explication** — Un VPC (Virtual Private Cloud) est le réseau isolé dans lequel vivent les ressources d'un compte, découpé en **sous-réseaux publics** (accessibles depuis Internet, typiquement pour un load balancer) et **sous-réseaux privés** (aucune route directe vers Internet, pour une base de données ou un service interne).

```bash
aws ec2 describe-vpcs --query "Vpcs[].{Id:VpcId,CIDR:CidrBlock}"
```

**Bonne pratique** : ne jamais placer une base de données dans un sous-réseau public — elle ne doit être joignable que depuis l'application, via le réseau privé du VPC.

### 07 - Configuration et identifiants de l'AWS CLI

**Explication** — `aws configure` enregistre localement une clé d'accès, une clé secrète et une région par défaut, utilisées par la CLI et les SDK pour s'authentifier.

```bash
aws configure
# AWS Access Key ID: ...
# AWS Secret Access Key: ...
# Default region name: eu-west-3
```

**Erreur fréquente** : committer des identifiants AWS (`AWS_ACCESS_KEY_ID`/`AWS_SECRET_ACCESS_KEY`) dans un fichier versionné dans Git — des bots scannent en continu GitHub à la recherche de ces clés pour les exploiter en quelques minutes. Les fournir via variables d'environnement, un gestionnaire de secrets, ou un rôle IAM attaché à la ressource plutôt qu'en dur (voir [`../security/`](../security/)).

### 08 - Modèle de facturation et Free Tier

**Explication** — Presque tout service AWS est facturé à l'usage réel (temps d'exécution, volume stocké, requêtes) plutôt qu'un forfait fixe. Le Free Tier offre un quota gratuit sur de nombreux services pendant 12 mois (ou en permanence pour certains, comme Lambda).

```bash
aws ce get-cost-and-usage --time-period Start=2026-08-01,End=2026-08-31 \
  --granularity MONTHLY --metrics "BlendedCost"
```

**Bonne pratique** : configurer une **Billing Alarm** dès la création du compte (via CloudWatch, voir section 4) qui notifie dès qu'un seuil de dépense est dépassé — une ressource oubliée en fonctionnement (ex. une instance EC2 non arrêtée) peut générer une facture surprise en quelques jours.

### 09 - Tags : organiser les ressources

**Explication** — Un tag (paire clé/valeur) attaché à une ressource permet de la retrouver, la filtrer, et ventiler les coûts par projet/environnement/équipe — indispensable dès que le compte héberge plus de quelques ressources.

```bash
aws ec2 create-tags --resources i-0123456789abcdef0 --tags Key=Environment,Value=staging Key=Project,Value=mon-api
```

**Bonne pratique** : définir une convention de tags dès le départ (`Environment`, `Project`, `Owner`) et l'appliquer systématiquement — un compte non tagué devient vite impossible à auditer ou à facturer précisément par projet.

## 4. Concepts intermédiaires 🟡

- **Lambda (serverless)** : exécute une fonction à la demande, déclenchée par un événement (requête HTTP, fichier déposé dans S3, message dans une file), facturée à la milliseconde d'exécution réelle — contrairement à une instance EC2 qui coûte tant qu'elle tourne, même inactive.

```bash
aws lambda create-function --function-name hello \
  --runtime nodejs20.x --handler index.handler \
  --role arn:aws:iam::123456789012:role/lambda-basic \
  --zip-file fileb://function.zip

aws lambda invoke --function-name hello output.json
```

- **RDS (bases de données managées)** : fait tourner MySQL/PostgreSQL (voir [`../mysql/`](../mysql/), [`../postgresql/`](../postgresql/)) sans administrer soi-même le serveur — sauvegardes automatiques, mises à jour de sécurité, et réplicas gérés par AWS plutôt qu'à la main.

```bash
aws rds create-db-instance --db-instance-identifier mon-db \
  --db-instance-class db.t3.micro --engine postgres \
  --master-username admin --master-user-password "MotDePasseSolide123!" \
  --allocated-storage 20
```

- **ECS et Fargate** : ECS (Elastic Container Service) fait tourner des conteneurs Docker sans installer/gérer Kubernetes ; Fargate va plus loin en supprimant même la gestion des instances EC2 sous-jacentes — on ne déclare que le conteneur et ses ressources, AWS s'occupe du serveur.
- **EKS (Kubernetes managé)** : la version managée d'AWS pour faire tourner un cluster Kubernetes ([`../kubernetes/`](../kubernetes/)) — AWS opère le plan de contrôle (control plane), l'équipe garde la main sur les workloads exactement comme sur un cluster auto-géré.
- **CloudWatch (observabilité de base)** : collecte logs, métriques et alarmes de la quasi-totalité des services AWS.

```bash
aws cloudwatch put-metric-alarm --alarm-name cpu-eleve \
  --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average \
  --period 300 --threshold 80 --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 --dimensions Name=InstanceId,Value=i-0123456789abcdef0
```

- **Route 53 (DNS)** : service DNS managé — héberge une zone pour un nom de domaine et route le trafic vers les ressources AWS (ou externes) correspondantes.

## 5. Concepts avancés 🟠🔴

- **Infrastructure as Code (Terraform/CloudFormation)** : décrire l'infrastructure de façon déclarative dans des fichiers versionnés plutôt que de cliquer dans la console — reproductible, review-able comme du code, et seule approche sérieuse au-delà d'un compte de test. Ne jamais construire une infrastructure de production "à la main" dans la console AWS.

```hcl
resource "aws_s3_bucket" "assets" {
  bucket = "mon-app-assets-prod"
}
```

- **Auto-scaling et répartition de charge** : un Auto Scaling Group ajuste automatiquement le nombre d'instances EC2 selon la charge, derrière un Application Load Balancer (ALB, HTTP/HTTPS) ou Network Load Balancer (NLB, TCP bas niveau) qui répartit le trafic entrant — le pendant AWS du Horizontal Pod Autoscaler déjà vu côté Kubernetes ([`../kubernetes/`](../kubernetes/)).
- **Haute disponibilité multi-région** : au-delà de la répartition multi-AZ (section 01), une architecture critique peut répliquer ses données et ses services sur plusieurs régions pour survivre à la panne complète d'une région entière — complexité et coût significatifs, à réserver aux besoins de disponibilité les plus stricts.
- **Optimisation des coûts** : instances **Reserved** (engagement 1-3 ans contre une forte réduction pour une charge stable et prévisible) ou **Spot** (capacité inutilisée d'AWS à prix cassé, mais récupérable à tout moment avec un court préavis — adapté à des charges tolérantes à l'interruption comme du traitement par lots), et le *right-sizing* (ajuster le type d'instance à l'usage réel plutôt que surdimensionner "par précaution").
- **Sécurité à l'échelle** : politiques IAM à privilège minimal systématiques, chiffrement au repos (S3, RDS, EBS) et en transit (TLS partout), et le [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) comme référentiel de bonnes pratiques transverses (sécurité, fiabilité, performance, coût) — voir [`../security/`](../security/).

## 6. Commandes / syntaxe à connaître

```bash
aws configure                              # configurer identifiants et région par défaut
aws s3 mb s3://nom / aws s3 cp / aws s3 ls   # créer un bucket, copier, lister
aws ec2 describe-instances                    # lister les instances EC2
aws ec2 run-instances / stop-instances           # démarrer / arrêter une instance
aws iam list-users / create-user / attach-user-policy   # gestion IAM
aws lambda invoke --function-name nom out.json      # invoquer une fonction Lambda
aws rds describe-db-instances                          # lister les bases RDS
aws cloudwatch describe-alarms                            # lister les alarmes
aws sts get-caller-identity                                  # vérifier l'identité actuellement authentifiée
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Déployer sur AWS l'application conteneurisée du mini-projet Docker**

En repartant de l'API + base de données conteneurisées décrites dans le mini-projet de [`../docker/README.md`](../docker/README.md) :
- Pousser l'image Docker de l'API vers **ECR** (Elastic Container Registry, le registre d'images d'AWS).
- Faire tourner cette image sur **ECS/Fargate** plutôt que sur un conteneur local — sans gérer d'instance EC2 dédiée.
- Remplacer le conteneur Postgres auto-hébergé par une instance **RDS PostgreSQL**, dans un sous-réseau privé du VPC.
- Stocker un asset statique (ex. une image uploadée) dans un bucket **S3** privé.
- Mettre en place une **alarme CloudWatch** sur le CPU du service ECS.

Bonus : provisionner l'ensemble (bucket S3, instance RDS, service ECS) avec un minimum de Terraform plutôt qu'à la main dans la console, et ajouter un Application Load Balancer devant le service ECS.

## Checklist

- [ ] Comprendre les fondamentaux (régions/AZ, IAM, S3, EC2, security groups)
- [ ] Savoir créer un utilisateur IAM avec une politique restreinte et configurer l'AWS CLI
- [ ] Maîtriser la syntaxe principale (`aws s3`, `aws ec2`, `aws iam`, `aws lambda`)
- [ ] Comprendre les concepts importants (Lambda, RDS, ECS/Fargate, EKS, CloudWatch)
- [ ] Savoir surveiller les coûts (Free Tier, Billing Alarm, tags)
- [ ] Connaître les bonnes pratiques (jamais de root/clés en dur, moindre privilège, bases en sous-réseau privé)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (Infrastructure as Code, auto-scaling/load balancing, Well-Architected Framework)

## 10. Ressources

- [Documentation officielle AWS](https://docs.aws.amazon.com/) — référence complète, service par service.
- [AWS Free Tier](https://aws.amazon.com/free/) — détail des quotas gratuits pour s'entraîner sans frais.
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/) — référentiel officiel de bonnes pratiques d'architecture cloud.
- [roadmap.sh — AWS](https://roadmap.sh/aws) — vue d'ensemble structurée des compétences à couvrir.
