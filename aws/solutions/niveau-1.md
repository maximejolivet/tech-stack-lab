# Solutions — Niveau 1 (Bases)

## Exercice 1

```bash
aws iam create-user --user-name deploy-bot

cat > s3-restricted-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    { "Effect": "Allow", "Action": ["s3:GetObject", "s3:PutObject"], "Resource": "arn:aws:s3:::mon-bucket-unique-12345/*" }
  ]
}
EOF

aws iam put-user-policy --user-name deploy-bot --policy-name s3-restricted --policy-document file://s3-restricted-policy.json
```

## Exercice 2

```bash
aws s3 mb s3://mon-bucket-unique-12345
echo "contenu de test" > fichier.txt
aws s3 cp fichier.txt s3://mon-bucket-unique-12345/
aws s3 cp s3://mon-bucket-unique-12345/fichier.txt ./telecharge.txt
cat telecharge.txt   # affiche "contenu de test"

aws s3api get-public-access-block --bucket mon-bucket-unique-12345
# Un bucket nouvellement créé bloque l'accès public par défaut (BlockPublicAcls, etc. tous à true)
```

## Exercice 3

```bash
aws ec2 create-security-group --group-name web-sg --description "SG pour instance web"

aws ec2 authorize-security-group-ingress --group-id sg-xxxx \
  --protocol tcp --port 22 --cidr 203.0.113.0/24   # SSH restreint à une plage d'IP connue

aws ec2 authorize-security-group-ingress --group-id sg-xxxx \
  --protocol tcp --port 80 --cidr 0.0.0.0/0          # HTTP ouvert à tous, normal pour un site public

aws ec2 run-instances --image-id ami-0abcdef1234567890 --instance-type t3.micro \
  --key-name ma-cle --security-group-ids sg-xxxx
```

## Exercice 4

```bash
aws configure
```

Les identifiants saisis sont stockés en clair dans `~/.aws/credentials` (et la config associée dans `~/.aws/config`) sur la machine locale. Ce fichier ne doit jamais être committé dans Git : il contient des identifiants permanents qui, une fois exposés publiquement, sont immédiatement exploités par des bots automatisés scannant GitHub — le `.gitignore` doit exclure `.aws/` si ce dossier se trouve par erreur dans un projet versionné.

## Exercice 5

```bash
aws ec2 create-tags --resources i-0123456789abcdef0 \
  --tags Key=Environment,Value=staging Key=Project,Value=mon-api

aws ec2 describe-instances \
  --filters "Name=tag:Project,Values=mon-api" \
  --query "Reservations[].Instances[].InstanceId"
```
