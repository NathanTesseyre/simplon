---
title: "Kubernetes — Fondamentaux"
tags:
  - kubernetes
  - k8s
  - conteneurs
  - orchestration
section: 04-Environnements
domaine: Kubernetes
statut: actif
liens_connexes:
  - [[Docker]]
  - [[docker_compose]]
  - [[vm_vs_conteneurs]]
---

# 📘 Fiche : Kubernetes — Fondamentaux

---

## 1. Pourquoi Kubernetes ?

Docker et Docker Compose gèrent des conteneurs sur **une seule machine**. En production, ça ne suffit pas :

| Problème production | Solution Kubernetes |
|---|---|
| Si la machine tombe, l'app tombe | Redistribue les conteneurs sur d'autres machines |
| Montée en charge manuelle | Scale automatiquement selon la charge |
| Mise à jour = coupure de service | Déploiement progressif sans downtime |
| Surveillance et redémarrage manuel | Redémarre les conteneurs en échec automatiquement |

**Kubernetes** (abrégé **k8s**) est un orchestrateur de conteneurs : il gère le déploiement, la mise à l'échelle et la disponibilité des applications conteneurisées sur un **cluster** de machines.

---

## 2. Architecture d'un cluster

```
Cluster Kubernetes
├── Control Plane (cerveau du cluster)
│   ├── API Server       → point d'entrée de toutes les commandes
│   ├── Scheduler        → décide sur quel nœud placer les pods
│   ├── Controller       → surveille l'état réel vs l'état désiré
│   └── etcd             → base de données de l'état du cluster
│
└── Worker Nodes (machines qui font tourner les apps)
    ├── kubelet          → agent sur chaque nœud, exécute les pods
    ├── kube-proxy       → gère le réseau entre pods
    └── Container Runtime → Docker, containerd...
```

> 💡 En pratique (dev local) : un seul nœud fait office de control plane + worker. En production : plusieurs nœuds worker pour la haute disponibilité.

---

## 3. Concepts clés

### Pod
L'unité de base de Kubernetes. Un pod encapsule **un ou plusieurs conteneurs** qui partagent le même réseau et stockage.

> En général : 1 pod = 1 conteneur.

### Deployment
Décrit l'état désiré d'une application : quelle image, combien de réplicas.  
Kubernetes s'assure en permanence que cet état est respecté.

### Service
Expose un ensemble de pods sur le réseau. Fournit une IP stable et du load balancing entre les pods.

| Type de Service | Usage |
|---|---|
| `ClusterIP` | Accessible uniquement dans le cluster (défaut) |
| `NodePort` | Expose un port sur chaque nœud du cluster |
| `LoadBalancer` | Crée un load balancer externe (cloud) |

### Namespace
Espace logique pour isoler des ressources dans un même cluster (ex: `dev`, `staging`, `prod`).

### ConfigMap
Stocke des **configurations non sensibles** (variables d'environnement, fichiers de config).

### Secret
Stocke des **données sensibles** (mots de passe, tokens, certificats) encodées en base64.

---

## 4. Commandes `kubectl` essentielles

| Commande | Rôle |
|---|---|
| `kubectl get pods` | Liste les pods |
| `kubectl get pods -n namespace` | Liste les pods d'un namespace |
| `kubectl get all` | Liste toutes les ressources |
| `kubectl describe pod nom` | Détails d'un pod (événements, erreurs) |
| `kubectl logs nom-pod` | Logs d'un pod |
| `kubectl logs -f nom-pod` | Logs en temps réel |
| `kubectl exec -it nom-pod -- bash` | Terminal dans un pod |
| `kubectl apply -f fichier.yaml` | Crée ou met à jour des ressources |
| `kubectl delete -f fichier.yaml` | Supprime des ressources |
| `kubectl scale deployment nom --replicas=3` | Change le nombre de réplicas |
| `kubectl rollout status deployment/nom` | Statut d'un déploiement |
| `kubectl rollout undo deployment/nom` | Rollback au déploiement précédent |

---

## 5. Exemple : Deployment + Service

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mon-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mon-app
  template:
    metadata:
      labels:
        app: mon-app
    spec:
      containers:
        - name: mon-app
          image: nginx:alpine
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "250m"
              memory: "256Mi"
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mon-app-service
spec:
  selector:
    app: mon-app       # cible les pods avec ce label
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get pods        # vérifie que les 3 réplicas tournent
```

---

## 6. Kubernetes en local : outils

| Outil | Usage |
|---|---|
| **Minikube** | Cluster k8s local sur une VM ou Docker |
| **kind** (Kubernetes in Docker) | Cluster local dans des conteneurs Docker |
| **k3s** | Distribution légère, idéale sur VPS ou Raspberry Pi |
| **Docker Desktop** | Inclut un cluster k8s local (Windows/Mac) |

---

## 7. Bonnes pratiques

- Toujours définir des **`resources.requests` et `resources.limits`** pour éviter qu'un pod monopolise les ressources.
- Utiliser des **namespaces** pour séparer les environnements.
- Ne jamais stocker de secrets en clair dans les YAML → utiliser des `Secret` k8s ou un gestionnaire externe (Vault, Sealed Secrets).
- Versionner tous les fichiers YAML dans Git.
- Utiliser des **labels** cohérents (`app`, `env`, `version`) sur toutes les ressources.
- Préférer `kubectl apply` à `kubectl create` pour rester en mode déclaratif.

---

## 8. ✅ À retenir

- Kubernetes orchestre des conteneurs sur un **cluster** de machines.
- Unité de base : le **Pod** (1 conteneur en général).
- **Deployment** = état désiré ; Kubernetes maintient cet état en permanence.
- **Service** = point d'accès stable vers un groupe de pods.
- Workflow : écrire des YAML → `kubectl apply` → Kubernetes fait le reste.
- En local : Minikube ou kind pour s'entraîner.
