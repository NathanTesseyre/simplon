---
title: "Helm — Gestionnaire de paquets Kubernetes"
tags:
  - helm
  - kubernetes
  - déploiement
  - devops
section: 04-Environnements
domaine: Kubernetes
statut: actif
liens_connexes:
  - [[kubernetes_fondamentaux]]
  - [[docker_compose]]
  - [[terraform_fondamentaux]]
---

# 📘 Fiche : Helm — Gestionnaire de paquets Kubernetes

---

## 1. Pourquoi Helm ?

Déployer une application sur Kubernetes implique souvent 5 à 10 fichiers YAML (Deployment, Service, Ingress, ConfigMap, Secret...). Gérer ces fichiers à la main pose des problèmes :

| Problème | Solution Helm |
|---|---|
| Dupliquer les YAML pour chaque environnement (dev/staging/prod) | Un seul chart, des **values** différentes par env |
| Pas de versioning des déploiements | Helm garde un **historique** des releases |
| Rollback manuel | `helm rollback` en une commande |
| Réinstaller des stacks connues (Nginx, Prometheus...) | Des milliers de charts prêts sur **Artifact Hub** |

> 💡 Helm est à Kubernetes ce que `apt` est à Debian — un gestionnaire de paquets.

---

## 2. Concepts clés

| Concept | Définition |
|---|---|
| **Chart** | Package Helm : ensemble de templates YAML + valeurs par défaut |
| **Release** | Instance déployée d'un chart dans un cluster |
| **Repository** | Serveur hébergeant des charts (comme Docker Hub pour les images) |
| **Values** | Paramètres qui personnalisent un chart (`values.yaml`) |
| **Template** | Fichier YAML avec des variables `{{ .Values.xxx }}` |

---

## 3. Installation

```bash
# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# macOS
brew install helm

# Vérification
helm version
```

---

## 4. Commandes essentielles

### Repositories

```bash
# Ajouter un repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable

# Mettre à jour les repos
helm repo update

# Lister les repos configurés
helm repo list

# Chercher un chart
helm search repo nginx
helm search hub wordpress          # cherche sur Artifact Hub
```

### Installer et gérer des releases

```bash
# Installer un chart
helm install ma-release bitnami/nginx

# Installer avec des values personnalisées
helm install ma-release bitnami/nginx -f values.yaml
helm install ma-release bitnami/nginx --set service.type=NodePort

# Installer dans un namespace
helm install ma-release bitnami/nginx -n mon-namespace --create-namespace

# Lister les releases
helm list
helm list -A          # tous les namespaces

# Mettre à jour une release
helm upgrade ma-release bitnami/nginx -f values.yaml

# Installer ou mettre à jour en une commande
helm upgrade --install ma-release bitnami/nginx -f values.yaml

# Voir le statut d'une release
helm status ma-release

# Historique des déploiements
helm history ma-release

# Rollback à la version précédente
helm rollback ma-release
helm rollback ma-release 2     # rollback vers la révision 2

# Désinstaller
helm uninstall ma-release
```

### Inspecter un chart

```bash
# Voir les values par défaut
helm show values bitnami/nginx

# Voir les templates générés (sans déployer)
helm template ma-release bitnami/nginx -f values.yaml

# Valider sans déployer
helm install ma-release bitnami/nginx --dry-run
```

---

## 5. Créer son propre chart

```bash
helm create mon-app
```

Structure générée :
```
mon-app/
├── Chart.yaml          # métadonnées du chart (nom, version, description)
├── values.yaml         # valeurs par défaut
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl    # fonctions réutilisables
│   └── NOTES.txt       # message affiché après install
└── charts/             # dépendances (sous-charts)
```

### `Chart.yaml`
```yaml
apiVersion: v2
name: mon-app
description: Mon application
version: 0.1.0          # version du chart
appVersion: "1.0.0"     # version de l'application
```

### `values.yaml`
```yaml
replicaCount: 2

image:
  repository: mon-image
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
```

### Template avec variables (`templates/deployment.yaml`)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-app
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: {{ .Values.service.port }}
```

### Values par environnement
```bash
# values-prod.yaml
replicaCount: 5
image:
  tag: "2.1.0"
```

```bash
helm upgrade --install mon-app ./mon-app -f values-prod.yaml
```

---

## 6. Bonnes pratiques

- Toujours utiliser `--dry-run` avant un `upgrade` en production.
- Versionner les fichiers `values-*.yaml` dans Git — pas les secrets.
- Utiliser `helm diff` (plugin) pour visualiser les changements avant un upgrade.
- Préférer `upgrade --install` à `install` seul dans les pipelines CI/CD.
- Vérifier les **notes** après installation (`helm status release`) — elles contiennent souvent les étapes post-install.

---

## 7. ✅ À retenir

- Helm = gestionnaire de paquets Kubernetes. Un chart = un package déployable.
- `helm upgrade --install` = commande universelle (installe ou met à jour).
- Les **values** permettent de personnaliser un chart sans modifier ses templates.
- `helm rollback` = retour arrière immédiat sur une release.
- **Artifact Hub** (artifacthub.io) = catalogue de charts communautaires.
