# Solutions — Niveau 1 (Bases)

## Exercice 1

```bash
SET session:xyz "user42" EX 1800
TTL session:xyz
```

## Exercice 2

```bash
SET page:contact:views 0
INCR page:contact:views
INCRBY page:contact:views 10
```

## Exercice 3

```bash
HSET user:5 name "Diane" email "diane@test.com" role "editor"
HGET user:5 email
```

## Exercice 4

```bash
LPUSH queue:notifications "email-a" "email-b" "email-c"
RPOP queue:notifications   # retourne "email-a" (le plus ancien)
```

## Exercice 5

```bash
ZADD leaderboard:game1 300 "alice" 450 "bob" 200 "carol"
ZREVRANGE leaderboard:game1 0 -1 WITHSCORES
```
