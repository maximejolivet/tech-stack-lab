# Solutions — Niveau 3 (Avancé)

## Exercice 1

```hcl
resource "aws_s3_bucket" "assets" {
  bucket = "mon-app-assets-prod"
}

resource "aws_iam_policy" "assets_rw" {
  name = "assets-read-write"
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect   = "Allow"
        Action   = ["s3:GetObject", "s3:PutObject"]
        Resource = "${aws_s3_bucket.assets.arn}/*"
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "attach" {
  role       = aws_iam_role.app_role.name
  policy_arn = aws_iam_policy.assets_rw.arn
}
```

Terraform décrit l'état désiré de l'infrastructure de façon déclarative et versionnable : `terraform plan` montre exactement ce qui va changer avant application, et `terraform apply` rend l'infrastructure conforme au fichier — reproductible et review-able comme du code, contrairement à des clics dans la console qui ne laissent aucune trace ni garantie de reproductibilité.

## Exercice 2

Un **Application Load Balancer** reçoit tout le trafic entrant et le répartit entre les instances saines d'un **Auto Scaling Group**, en écartant automatiquement toute instance qui échoue son health check. L'Auto Scaling Group surveille une métrique cible (le plus souvent le CPU moyen des instances, via CloudWatch) : au-dessus d'un seuil haut (ex. 70% pendant plusieurs minutes), il lance de nouvelles instances ; en dessous d'un seuil bas (ex. 20%), il en retire — bornées par un minimum et un maximum définis à l'avance pour éviter un scaling incontrôlé. Ce dispositif absorbe les pics de trafic (heures de pointe) sans payer en permanence pour la capacité maximale, et sans intervention manuelle.

## Exercice 3

**Service backend 24/7 à charge stable** → **Reserved Instance** : un engagement 1 ou 3 ans réduit significativement le coût par rapport à On-Demand, rentable précisément parce que la charge est prévisible et continue — aucun risque de payer un engagement pour une capacité finalement inutilisée.

**Job de traitement par lots nocturne, interruptible** → **Spot Instance** : le job tolère une interruption avec relance (pas de contrainte de continuité de service), ce qui permet de profiter de la capacité EC2 excédentaire à prix fortement réduit (souvent 70-90% moins cher qu'On-Demand) — le risque de récupération de l'instance par AWS avec un court préavis est acceptable ici, contrairement à un service utilisateur en production.

## Exercice 4

Le problème : `"Action": "*", "Resource": "*"` autorise **absolument toutes** les actions sur **toutes** les ressources du compte (créer/supprimer des instances, modifier IAM, effacer des buckets...) — bien au-delà du besoin réel de lire/écrire un seul bucket. Si les identifiants de cet utilisateur fuitent, l'attaquant a un contrôle total du compte.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:ListBucket"],
      "Resource": ["arn:aws:s3:::mon-bucket", "arn:aws:s3:::mon-bucket/*"]
    }
  ]
}
```

Cette politique restreinte n'autorise que la lecture/écriture d'objets et le listing du bucket `mon-bucket` précis — conforme au principe du moindre privilège : même compromise, elle ne donne accès qu'à un périmètre limité et connu.

## Exercice 5

Le **load balancer** est placé dans un sous-réseau **public** (il doit recevoir le trafic direct d'Internet). L'**API** peut être placée en sous-réseau public ou privé selon le besoin d'accès direct, mais idéalement en privé, atteinte uniquement via le load balancer. La **base de données** doit impérativement être en sous-réseau **privé**, sans route vers Internet : elle n'est jamais censée recevoir de trafic entrant direct depuis l'extérieur, seulement des connexions internes provenant de l'API. Placer une base de données en sous-réseau public l'expose directement à Internet — même protégée par un mot de passe, elle devient une cible de scans et de tentatives de connexion automatisées qu'un simple security group mal configuré peut laisser passer ; l'isolation réseau est une couche de défense supplémentaire indépendante des identifiants.
