# Solutions — Niveau 2 (Intermédiaire)

## Exercice 1

```js
// index.js
exports.handler = async (event) => {
  const name = event.name || "inconnu";
  return { statusCode: 200, body: `Bonjour, ${name} !` };
};
```

```bash
zip function.zip index.js

aws lambda create-function --function-name hello \
  --runtime nodejs20.x --handler index.handler \
  --role arn:aws:iam::123456789012:role/lambda-basic \
  --zip-file fileb://function.zip

echo '{"name":"Max"}' > input.json
aws lambda invoke --function-name hello --payload fileb://input.json output.json
cat output.json   # { "statusCode": 200, "body": "Bonjour, Max !" }
```

## Exercice 2

```bash
aws rds create-db-instance --db-instance-identifier mon-db \
  --db-instance-class db.t3.micro --engine postgres \
  --master-username admin --master-user-password "MotDePasseSolide123!" \
  --allocated-storage 20 --no-publicly-accessible \
  --vpc-security-group-ids sg-db-only

aws rds describe-db-instances --db-instance-identifier mon-db \
  --query "DBInstances[0].Endpoint"
```

L'instance est créée avec `--no-publicly-accessible` : elle n'a pas d'IP publique et n'est joignable que depuis des ressources situées dans le même VPC (typiquement l'application, via un security group qui autorise explicitement le port 5432 depuis le security group de l'application, jamais depuis Internet).

## Exercice 3

```bash
aws ecr create-repository --repository-name mon-api

aws ecr get-login-password | docker login --username AWS --password-stdin 123456789012.dkr.ecr.eu-west-3.amazonaws.com

docker tag mon-api:1.0 123456789012.dkr.ecr.eu-west-3.amazonaws.com/mon-api:1.0
docker push 123456789012.dkr.ecr.eu-west-3.amazonaws.com/mon-api:1.0

aws ecs create-cluster --cluster-name mon-cluster
# Puis créer une task definition référençant cette image, et un service Fargate qui l'exécute
aws ecs create-service --cluster mon-cluster --service-name mon-api-svc \
  --task-definition mon-api-task --desired-count 2 --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxxx],securityGroups=[sg-xxxx],assignPublicIp=ENABLED}"
```

## Exercice 4

```bash
aws cloudwatch put-metric-alarm --alarm-name cpu-eleve \
  --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average \
  --period 300 --threshold 80 --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 --dimensions Name=InstanceId,Value=i-0123456789abcdef0 \
  --alarm-actions arn:aws:sns:eu-west-3:123456789012:alertes
```

`--period 300` (5 minutes) combiné à `--evaluation-periods 2` déclenche l'alarme seulement après 10 minutes consécutives au-dessus du seuil, évitant les faux positifs sur un simple pic ponctuel.

## Exercice 5

```bash
aws route53 create-hosted-zone --name mon-domaine.com --caller-reference $(date +%s)

aws route53 change-resource-record-sets --hosted-zone-id Z1234567890 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "mon-domaine.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{ "Value": "203.0.113.10" }]
      }
    }]
  }'
```
