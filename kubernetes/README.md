# Kubernetes

## 1. Introduction

Kubernetes (K8s) est une plateforme d'orchestration de conteneurs : elle automatise le déploiement, la mise à l'échelle et la supervision d'applications conteneurisées sur un cluster de machines. Ce dossier suppose Docker acquis (voir [`../docker/`](../docker/)) — Kubernetes ne remplace pas Docker, il orchestre des conteneurs (construits, par exemple, avec Docker) à grande échelle.

**À quoi sert-il ?**
- Faire tourner une application sur plusieurs machines (nœuds) de façon résiliente : si un nœud tombe, Kubernetes replanifie automatiquement les conteneurs ailleurs.
- Mettre à l'échelle automatiquement le nombre d'instances d'une application selon la charge.
- Déployer de nouvelles versions sans interruption de service (rolling updates) et revenir en arrière automatiquement en cas de problème.

**Où se situe-t-il dans une architecture web ?** Une couche d'orchestration au-dessus de conteneurs (voir [`../docker/`](../docker/)) exécutés sur un ensemble de machines Linux (voir [`../linux/`](../linux/)) — pertinent à partir du moment où une seule machine ou un simple `docker compose` ne suffit plus (haute disponibilité, montée en charge, plusieurs équipes/services).

**Avantages** : auto-guérison (self-healing) des applications, scaling automatique, portabilité entre clouds (AWS, GCP, Azure, on-premise), écosystème d'outils mature (Helm, Prometheus, ArgoCD).
**Limites** : complexité opérationnelle élevée, courbe d'apprentissage raide (nombreux concepts : Pods, Services, Ingress, RBAC...), souvent disproportionné pour une petite application à faible trafic — un simple serveur ou `docker compose` suffit largement dans ce cas.

## 2. Prérequis

- Docker maîtrisé : images, conteneurs, réseau de base (voir [`../docker/`](../docker/)).
- Bases Linux et réseau (voir [`../linux/`](../linux/)).
- Un cluster local pour s'entraîner ([kind](https://kind.sigs.k8s.io/) ou [minikube](https://minikube.sigs.k8s.io/)) et `kubectl` installé.

## 3. Rappel des bases 🟢

### 01 - Pods

**Explication** — Le Pod est la plus petite unité déployable de Kubernetes : un ou plusieurs conteneurs partageant le même réseau et le même stockage. En pratique, un Pod contient le plus souvent un seul conteneur applicatif.

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mon-api
spec:
  containers:
    - name: api
      image: mon-api:1.0
      ports:
        - containerPort: 3000
```

```bash
kubectl apply -f pod.yaml
kubectl get pods
```

**Erreur fréquente** : créer des Pods directement en production — un Pod seul n'est ni auto-réparé ni répliqué. En pratique, on ne crée quasiment jamais de Pod directement, mais via un Deployment (voir 02).

### 02 - Deployments

**Explication** — Un Deployment gère un ensemble de Pods identiques (répliques), et prend en charge leur création, leur remplacement en cas de panne, et les mises à jour progressives (rolling updates).

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mon-api
spec:
  replicas: 3
  selector:
    matchLabels: { app: mon-api }
  template:
    metadata:
      labels: { app: mon-api }
    spec:
      containers:
        - name: api
          image: mon-api:1.0
          ports: [{ containerPort: 3000 }]
```

```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl scale deployment mon-api --replicas=5
```

**Bonne pratique** : toujours définir des `replicas` >= 2 en production — avec une seule réplique, une mise à jour ou une panne de nœud crée une interruption de service.

### 03 - Services

**Explication** — Les Pods sont éphémères (IP changeante à chaque recréation) : un **Service** fournit une adresse stable et répartit le trafic vers les Pods correspondant à son sélecteur de labels.

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mon-api-svc
spec:
  selector: { app: mon-api }
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
```

**Erreur fréquente** : oublier que le `selector` du Service doit correspondre exactement aux `labels` des Pods ciblés — un décalage silencieux (ex. une faute de frappe) fait que le Service ne route vers aucun Pod, sans erreur explicite.

### 04 - kubectl : commandes de base

**Explication** — `kubectl` est l'outil en ligne de commande principal pour interagir avec un cluster.

```bash
kubectl get pods / deployments / services      # lister des ressources
kubectl describe pod mon-api-xxxxx               # détails et événements d'un Pod
kubectl logs mon-api-xxxxx                        # logs d'un Pod
kubectl delete pod mon-api-xxxxx                   # supprimer (le Deployment le recrée)
```

### 05 - Manifestes YAML et apply

**Explication** — Toute ressource Kubernetes se décrit de façon déclarative en YAML, appliquée au cluster via `kubectl apply`, qui compare l'état désiré (le fichier) à l'état réel du cluster et n'applique que la différence.

**Bonne pratique** : toujours gérer les manifestes YAML comme du code source, versionnés dans Git — c'est la base d'une approche GitOps (voir [`../ci-cd/`](../ci-cd/)).

### 06 - Namespaces

**Explication** — Un namespace partitionne logiquement un cluster (ex. `dev`, `staging`, `prod`, ou un namespace par équipe), avec des ressources et des permissions isolées entre eux.

```bash
kubectl create namespace staging
kubectl apply -f deployment.yaml -n staging
kubectl get pods -n staging
```

**Bonne pratique** : ne jamais tout déployer dans le namespace `default` — nommer et isoler les namespaces dès le départ facilite grandement la gestion des permissions (RBAC) et le nettoyage.

## 4. Concepts intermédiaires 🟡

- **ConfigMaps et Secrets** : externaliser la configuration (ConfigMap) et les données sensibles (Secret) hors de l'image et du manifeste applicatif, injectées en variables d'environnement ou en fichiers montés.

```yaml
apiVersion: v1
kind: ConfigMap
metadata: { name: mon-api-config }
data:
  LOG_LEVEL: "info"
---
apiVersion: v1
kind: Secret
metadata: { name: mon-api-secret }
type: Opaque
stringData:
  DB_PASSWORD: "secret"
```

**Erreur fréquente** : traiter un Secret comme réellement chiffré par défaut — sans configuration additionnelle (chiffrement au repos, outil externe type Vault/Sealed Secrets), un Secret Kubernetes n'est encodé qu'en base64, pas chiffré.

- **Ingress** : point d'entrée HTTP(S) unique qui route le trafic externe vers différents Services selon le nom de domaine ou le chemin, avec gestion du TLS — nécessite un Ingress Controller (ex. nginx-ingress) installé dans le cluster.
- **Liveness et Readiness probes** : deux sondes de santé distinctes — *liveness* détermine si le conteneur doit être redémarré (bloqué/planté), *readiness* détermine s'il doit recevoir du trafic (encore en train de démarrer, ou temporairement surchargé).

```yaml
livenessProbe:
  httpGet: { path: /health, port: 3000 }
  periodSeconds: 10
readinessProbe:
  httpGet: { path: /ready, port: 3000 }
  periodSeconds: 5
```

- **Rolling updates et rollbacks** : par défaut, un Deployment remplace progressivement les anciens Pods par les nouveaux sans interruption ; `kubectl rollout undo deployment/mon-api` revient instantanément à la version précédente en cas de problème.
- **Horizontal Pod Autoscaler (HPA)** : ajuste automatiquement le nombre de répliques d'un Deployment selon une métrique (le plus souvent l'usage CPU moyen des Pods).
- **Volumes persistants** : `PersistentVolume`/`PersistentVolumeClaim` découplent le stockage physique de son utilisation par un Pod — nécessaires pour toute donnée devant survivre au remplacement d'un Pod (ex. une base de données).

## 5. Concepts avancés 🟠🔴

- **StatefulSets vs Deployments** : contrairement à un Deployment (Pods interchangeables et sans identité stable), un StatefulSet donne à chaque réplique un nom et un stockage stables et persistants — indispensable pour des applications avec état comme une base de données en cluster.
- **Helm** : gestionnaire de paquets pour Kubernetes — un "chart" Helm empaquette un ensemble de manifestes YAML templatés et paramétrables, simplifiant le déploiement d'applications complexes (ex. installer PostgreSQL en une commande plutôt que d'écrire tous les manifestes à la main).

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install ma-db bitnami/postgresql
```

- **RBAC (Role-Based Access Control)** : contrôle fin de qui (utilisateur, service account) peut faire quoi (verbes : get/list/create/delete) sur quelles ressources, dans quel namespace — via des objets `Role`/`ClusterRole` et `RoleBinding`/`ClusterRoleBinding`.
- **Autoscaling avancé** : au-delà du HPA basé CPU, le Vertical Pod Autoscaler (ajuste les requêtes de ressources d'un Pod) et le Cluster Autoscaler (ajoute/retire des nœuds physiques au cluster selon la demande).
- **Observabilité** : Prometheus pour la collecte de métriques, Grafana pour leur visualisation, agrégation centralisée des logs (car les logs d'un Pod disparaissent avec lui) — essentiel dès qu'un cluster dépasse quelques services.
- **GitOps** : synchroniser en continu l'état d'un cluster avec un dépôt Git faisant autorité (outils comme ArgoCD ou Flux), plutôt que d'appliquer des `kubectl apply` manuels — voir [`../ci-cd/`](../ci-cd/).
- **Sécurité approfondie** : Network Policies (restreindre quels Pods peuvent communiquer entre eux, refusé par défaut sinon tout le trafic est autorisé), Pod Security Standards (interdire les conteneurs privilégiés), gestion des secrets via un coffre-fort externe (Vault) plutôt que les Secrets natifs — voir [`../security/`](../security/).

## 6. Commandes / syntaxe à connaître

```bash
kubectl apply -f fichier.yaml           # créer/mettre à jour une ressource
kubectl get pods/deployments/services -n ns   # lister des ressources
kubectl describe pod nom                  # détails et événements
kubectl logs -f nom                        # logs en temps réel
kubectl exec -it nom -- sh                   # shell dans un Pod
kubectl scale deployment nom --replicas=N     # ajuster le nombre de répliques
kubectl rollout status/undo deployment/nom      # suivre / annuler un déploiement
kubectl delete -f fichier.yaml                    # supprimer les ressources d'un manifeste
```

## 7. Exercices

Trois niveaux progressifs, énoncés dans [`exercices/`](exercices/), corrections séparées dans [`solutions/`](solutions/) (à consulter seulement après avoir cherché) :

- [Niveau 1 — Bases](exercices/niveau-1.md)
- [Niveau 2 — Intermédiaire](exercices/niveau-2.md)
- [Niveau 3 — Avancé](exercices/niveau-3.md)

## 8. Mini-projet

**Déployer l'application conteneurisée (Docker) sur un cluster local**

En réutilisant l'image `mon-api` du mini-projet Docker (voir [`../docker/`](../docker/)), sur un cluster local (kind/minikube) :
- Un `Deployment` avec 3 répliques de `mon-api`, liveness et readiness probes configurées.
- Un `Service` de type `ClusterIP` exposant le Deployment.
- Un `Ingress` routant `mon-api.local` vers ce Service.
- Une `ConfigMap` pour la configuration non sensible et un `Secret` pour le mot de passe de base de données, injectés en variables d'environnement dans le Deployment.
- Un namespace dédié (`demo`) regroupant toutes ces ressources.

Bonus : ajouter un `HorizontalPodAutoscaler` basé sur le CPU, tester un rolling update (changer le tag de l'image) puis un `kubectl rollout undo`, et empaqueter le tout en chart Helm minimal.

## Checklist

- [ ] Comprendre les fondamentaux (Pod, Deployment, Service, manifestes YAML déclaratifs)
- [ ] Savoir déployer une application sur un cluster local (kind/minikube) via `kubectl apply`
- [ ] Maîtriser la syntaxe principale (`kubectl get/describe/logs/scale/rollout`)
- [ ] Comprendre les concepts importants (ConfigMap/Secret, Ingress, probes, HPA)
- [ ] Savoir debugger (`kubectl describe`, `kubectl logs`, événements du cluster)
- [ ] Connaître les bonnes pratiques (namespaces dédiés, replicas >= 2, GitOps)
- [ ] Réaliser les exercices (niveaux 1 à 3)
- [ ] Réaliser le mini-projet
- [ ] Comprendre les notions avancées (StatefulSets, Helm, RBAC, Network Policies)

## 10. Ressources

- [Documentation officielle Kubernetes](https://kubernetes.io/docs/home/) — référence complète.
- [Kubernetes by Example](https://kubernetesbyexample.com/) — approche pratique par exemples courts.
- [Helm — documentation officielle](https://helm.sh/docs/) — gestionnaire de paquets Kubernetes.
- [roadmap.sh — Kubernetes](https://roadmap.sh/kubernetes) — vue d'ensemble structurée des compétences à couvrir.
