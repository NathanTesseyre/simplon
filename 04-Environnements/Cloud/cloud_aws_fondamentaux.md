---
title: "Cloud — AWS Fondamentaux"
tags:
  - cloud
  - aws
  - infrastructure
  - devops
section: 04-Environnements
domaine: Cloud
statut: actif
liens_connexes:
  - [[cloud_virtualisation]]
  - [[fiche_architecture_infra]]
  - [[terraform_fondamentaux]]
---

# 📘 Fiche : Cloud — AWS Fondamentaux

---

## 1. Pourquoi le cloud ?

| Infrastructure on-premise | Cloud |
|---|---|
| Achat et maintenance du matériel | Loué à la demande, payé à l'usage |
| Capacité fixe (sur/sous-dimensionnement) | Scalable en quelques minutes |
| Déploiement lent (semaines) | Déploiement immédiat via API |
| Haute disponibilité complexe à mettre en place | HA et redondance intégrées |

**AWS** (Amazon Web Services) est le cloud provider le plus utilisé. Les concepts sont très proches sur **Azure** (Microsoft) et **GCP** (Google Cloud).

---

## 2. Concepts fondamentaux

### Région et Zone de disponibilité

```
Région (ex: eu-west-3 = Paris)
├── Zone A  (eu-west-3a) → datacenter A
├── Zone B  (eu-west-3b) → datacenter B
└── Zone C  (eu-west-3c) → datacenter C
```

- **Région** : zone géographique indépendante. Choisir selon la localisation des utilisateurs et les contraintes légales (RGPD → Europe).
- **Zone de disponibilité (AZ)** : datacenter physiquement séparé dans une région. Déployer sur plusieurs AZ = haute disponibilité.

### IAM (Identity and Access Management)
Gestion des accès à AWS.

| Concept | Rôle |
|---|---|
| **Utilisateur IAM** | Personne ou service avec identifiants |
| **Groupe** | Ensemble d'utilisateurs partageant les mêmes permissions |
| **Rôle IAM** | Permissions temporaires assumées par un service AWS (ex: EC2 qui lit S3) |
| **Politique (Policy)** | Document JSON définissant les permissions |

> 💡 Principe du moindre privilège : n'accorder que les permissions strictement nécessaires.

---

## 3. Services essentiels

### Calcul

| Service | Rôle |
|---|---|
| **EC2** (Elastic Compute Cloud) | VMs dans le cloud (instances) |
| **Lambda** | Fonctions serverless (exécution à la demande, sans gérer de serveur) |
| **ECS / EKS** | Orchestration de conteneurs (ECS = AWS natif, EKS = Kubernetes managé) |

### Stockage

| Service | Rôle |
|---|---|
| **S3** (Simple Storage Service) | Stockage d'objets (fichiers, backups, assets statiques) |
| **EBS** (Elastic Block Store) | Disque attaché à une instance EC2 |
| **EFS** (Elastic File System) | Système de fichiers partagé entre plusieurs instances |

### Réseau

| Service | Rôle |
|---|---|
| **VPC** (Virtual Private Cloud) | Réseau privé isolé dans AWS |
| **Security Group** | Firewall au niveau de l'instance (règles entrée/sortie) |
| **Route 53** | DNS managé |
| **ELB** (Elastic Load Balancer) | Load balancer managé |
| **CloudFront** | CDN mondial |

### Base de données

| Service | Rôle |
|---|---|
| **RDS** | Bases de données managées (PostgreSQL, MySQL, MariaDB...) |
| **DynamoDB** | Base NoSQL managée |
| **ElastiCache** | Cache managé (Redis, Memcached) |

---

## 4. AWS CLI

### Installation et configuration
```bash
# Installation (Linux)
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && sudo ./aws/install

# Configuration (crée ~/.aws/credentials et ~/.aws/config)
aws configure
# AWS Access Key ID: ...
# AWS Secret Access Key: ...
# Default region name: eu-west-3
# Default output format: json
```

### Commandes de base

```bash
# Identité courante
aws sts get-caller-identity

# EC2
aws ec2 describe-instances
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# S3
aws s3 ls                                      # lister les buckets
aws s3 ls s3://mon-bucket/                     # lister le contenu
aws s3 cp fichier.txt s3://mon-bucket/         # uploader
aws s3 sync ./dossier s3://mon-bucket/dossier/ # synchroniser

# IAM
aws iam list-users
aws iam get-user --user-name alice
```

---

## 5. VPC — Réseau isolé

Un **VPC** est un réseau privé virtuel dans AWS. Par défaut, AWS en fournit un par région.

```
VPC (10.0.0.0/16)
├── Subnet public  (10.0.1.0/24)  → accès Internet via Internet Gateway
│   └── Instance EC2 (IP publique)
└── Subnet privé   (10.0.2.0/24)  → pas d'accès Internet direct
    └── Base de données RDS
```

**Security Groups** : firewall attaché à chaque instance.
```
Règle entrante : TCP 443 depuis 0.0.0.0/0  → HTTPS autorisé depuis partout
Règle entrante : TCP 22 depuis 10.0.0.0/16 → SSH autorisé depuis le VPC uniquement
```

---

## 6. Bonnes pratiques

- **Ne jamais utiliser le compte root AWS** pour le quotidien — créer des utilisateurs IAM.
- **Ne jamais committer de credentials AWS** dans Git (Access Key ID + Secret).
- Utiliser des **rôles IAM** pour les services AWS entre eux (pas de credentials statiques).
- Déployer sur **au moins 2 zones de disponibilité** pour la haute disponibilité.
- Activer **CloudTrail** pour auditer toutes les actions sur le compte.
- Configurer des **alertes de facturation** pour éviter les mauvaises surprises.
- Préférer gérer l'infra AWS avec **Terraform** plutôt que via la console.

---

## 7. Équivalents Azure et GCP

| Concept | AWS | Azure | GCP |
|---|---|---|---|
| VM | EC2 | Virtual Machine | Compute Engine |
| Stockage objet | S3 | Blob Storage | Cloud Storage |
| BDD managée | RDS | Azure Database | Cloud SQL |
| Kubernetes managé | EKS | AKS | GKE |
| Serverless | Lambda | Azure Functions | Cloud Functions |
| DNS | Route 53 | Azure DNS | Cloud DNS |
| IAM | IAM | Azure AD / Entra | Cloud IAM |

---

## 8. ✅ À retenir

- AWS = cloud provider dominant ; les concepts sont transposables à Azure et GCP.
- **Région** = zone géographique ; **AZ** = datacenter dans la région — déployer sur plusieurs AZ pour la HA.
- Services clés : **EC2** (VMs), **S3** (stockage), **RDS** (BDD), **VPC** (réseau), **IAM** (accès).
- **IAM** : principe du moindre privilège, pas de credentials dans le code.
- **Terraform** = outil recommandé pour provisionner l'infra AWS.
