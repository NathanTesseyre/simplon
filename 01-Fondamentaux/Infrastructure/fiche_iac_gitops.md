---
title: "IaC et GitOps — concepts fondamentaux"
tags:
  - fondamentaux
  - infrastructure
  - IaC
  - GitOps
  - DevOps
section: 01-Fondamentaux
domaine: Infrastructure
statut: actif
liens_connexes:
  - [[terraform_fondamentaux]]
  - [[ansible_fondamentaux]]
  - [[fiche_git]]
  - [[kubernetes_fondamentaux]]
---

# 🏗️ Fiche : IaC et GitOps — concepts fondamentaux
*(à destination d'administrateurs système en formation)*

---

## 1. Le problème que l'IaC résout

### L'infrastructure traditionnelle ("ClickOps")
```
Admin → Console web → Clics manuels → Serveur créé
```
- **Non reproductible** : impossible de recréer exactement le même environnement.
- **Non versionné** : pas d'historique des changements.
- **Risque d'erreur humaine** : oubli d'une option, faute de frappe.
- **Dérive de configuration** : les serveurs divergent au fil des opérations manuelles.

### Avec l'IaC
```
Dev → Fichier de code → Git → Pipeline CI/CD → Infrastructure
```
- **Reproductible** : recréer prod en 10 minutes.
- **Versionné** : `git log` montre qui a changé quoi et pourquoi.
- **Révisable** : code review avant d'appliquer.
- **Auditabe** : conformité, sécurité, traçabilité.

---

## 2. Infrastructure as Code (IaC)

**L'IaC** consiste à décrire l'infrastructure dans des fichiers de code, gérés comme du code source.

### Deux approches

| Approche | Description | Exemple |
|---|---|---|
| **Déclarative** | On décrit **l'état final souhaité**, l'outil trouve comment y arriver | Terraform, Kubernetes YAML |
| **Impérative** | On décrit **les étapes à suivre** pour atteindre l'état | Scripts Bash, Ansible (en partie) |

> La **déclarative** est préférable pour l'IaC : l'outil gère la convergence et est idempotent.

### Idempotence
Un outil IaC est **idempotent** si appliquer la même configuration plusieurs fois donne toujours le même résultat, sans effets de bord.

```bash
# Bash — NON idempotent
echo "127.0.0.1 monserveur" >> /etc/hosts   # ajoute à chaque exécution !

# Ansible — idempotent
- name: Ajouter entrée hosts
  lineinfile:
    path: /etc/hosts
    line: "127.0.0.1 monserveur"            # vérifie si déjà présent avant d'agir
```

### Catégories d'outils IaC

| Catégorie | Rôle | Outils |
|---|---|---|
| **Provisioning** | Créer/détruire l'infrastructure (VM, réseaux, BDD…) | Terraform, Pulumi, CloudFormation |
| **Configuration** | Configurer et maintenir les OS et logiciels | Ansible, Chef, Puppet, Salt |
| **Conteneurs** | Définir les images et les déploiements | Dockerfile, Kubernetes YAML, Helm |
| **CI/CD** | Automatiser les pipelines de déploiement | GitHub Actions, GitLab CI, Jenkins |

---

## 3. Terraform — provisioning déclaratif

Terraform décrit l'infrastructure en **HCL (HashiCorp Configuration Language)** et gère son cycle de vie.

### Workflow
```
terraform init    → télécharge les providers
terraform plan    → montre ce qui va changer (dry-run)
terraform apply   → applique les changements
terraform destroy → détruit l'infrastructure
```

### Exemple minimal (créer une VM sur AWS)
```hcl
# main.tf
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" {
  region = "eu-west-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  tags = { Name = "web-server" }
}
```

### State — concept clé
Terraform maintient un **fichier d'état** (`terraform.tfstate`) qui représente l'infrastructure actuelle.

```
Code HCL (souhaité) ──→ Terraform ──→ API Cloud
                              │
                        terraform.tfstate
                        (état réel connu)
```

> En équipe, le state doit être **partagé et verrouillé** (S3 + DynamoDB sur AWS, GitLab-managed state...).

---

## 4. Ansible — configuration déclarative

Ansible configure les serveurs **sans agent** (juste SSH) via des **playbooks** YAML.

### Concepts

| Terme | Définition |
|---|---|
| **Inventory** | Liste des serveurs à gérer |
| **Playbook** | Fichier YAML décrivant les tâches à exécuter |
| **Task** | Une action unitaire (installer un paquet, copier un fichier…) |
| **Module** | Brique prête à l'emploi (apt, copy, service, template…) |
| **Role** | Regroupement de tasks réutilisables |

### Exemple minimal
```yaml
# playbook.yml
- hosts: webservers
  become: true          # sudo
  tasks:
    - name: Installer Nginx
      apt:
        name: nginx
        state: present

    - name: Démarrer et activer Nginx
      service:
        name: nginx
        state: started
        enabled: true
```

### Différence Terraform vs Ansible

| | Terraform | Ansible |
|---|---|---|
| **Quand** | Créer l'infrastructure | Configurer ce qui tourne |
| **Quoi** | VMs, réseaux, DNS, BDD cloud | OS, paquets, fichiers de config, services |
| **State** | Oui (fichier tfstate) | Non (stateless par défaut) |
| **Idempotence** | Native | Par module (à vérifier) |

> En pratique : **Terraform crée la VM, Ansible la configure**.

---

## 5. GitOps

Le **GitOps** étend l'IaC en faisant de **Git la source de vérité unique** pour l'infrastructure ET les applications.

### Principe fondateur
```
Git repository
   └── état souhaité (code, config, manifests Kubernetes…)
         │
         ▼ (automatiquement)
Infrastructure / Cluster
   └── état réel (doit correspondre au Git)
```

Tout changement passe par **Git** : `git push` → pipeline → déploiement. Plus de `kubectl apply` ou `terraform apply` manuels.

### Les 4 principes GitOps (OpenGitOps)

1. **Déclaratif** — l'état souhaité est décrit, pas les étapes.
2. **Versionné et immuable** — Git est la source de vérité, avec historique complet.
3. **Tiré automatiquement** — un agent surveille Git et applique les changements.
4. **Réconciliation continue** — l'état réel converge toujours vers l'état souhaité.

### Push vs Pull

| Modèle | Description | Exemple |
|---|---|---|
| **Push** | Le pipeline CI/CD pousse les changements vers l'infra | GitHub Actions → `kubectl apply` |
| **Pull** | Un agent dans le cluster surveille Git et tire les changements | ArgoCD, Flux |

```
Push model :
[Git] → [CI/CD pipeline] → kubectl apply → [Cluster]

Pull model (GitOps pur) :
[Git] ← surveille ← [ArgoCD agent dans le cluster]
                          │
                     applique si différence
```

### Outils GitOps pour Kubernetes

| Outil | Modèle | Points forts |
|---|---|---|
| **ArgoCD** | Pull | UI graphique, notifications, multi-cluster |
| **Flux** | Pull | Léger, intégration Helm native |
| **Tekton** | Push | Pipelines Kubernetes-natifs |

---

## 6. Drift — dérive de configuration

La **dérive** (drift) survient quand l'état réel de l'infrastructure diverge de l'état décrit dans le code.

```
État en Git : nginx version 1.24
État réel   : nginx version 1.20  (quelqu'un a fait un apt install manuellement)
                    ↑ DRIFT
```

### Détecter et corriger le drift
```bash
# Terraform : détecter le drift
terraform plan    # montre les différences entre state et réalité

# Terraform : resynchroniser le state sans changer l'infra
terraform refresh

# Ansible : mode check (dry-run)
ansible-playbook playbook.yml --check --diff
```

> **En GitOps**, la réconciliation continue corrige automatiquement le drift : toute modification manuelle est écrasée par l'état Git.

---

## 7. Flux de travail typique (Pipeline IaC/GitOps)

```
Dev
  │ 1. Modifie le code Terraform/Ansible/YAML
  │ 2. git commit && git push (branche feature)
  ▼
Git (PR / Merge Request)
  │ 3. CI : lint, validation, terraform plan (dry-run)
  │ 4. Code review par un pair
  │ 5. Merge sur main
  ▼
CD Pipeline (ou agent GitOps)
  │ 6. terraform apply / ansible-playbook / kubectl apply
  ▼
Infrastructure
  │ 7. Vérification de santé (health checks)
  │ 8. Notification Slack / alerting
  ▼
Monitoring
```

---

## 8. Bonnes pratiques

| Pratique | Pourquoi |
|---|---|
| **Ne jamais modifier l'infra manuellement** (ClickOps) | Crée du drift, rompt la traçabilité |
| **Stocker les secrets hors du code** | Vault, AWS Secrets Manager, SOPS |
| **Versionner les modules** | Évite les régressions lors des mises à jour |
| **Plan avant apply** | Toujours relire ce que Terraform va faire |
| **Environnements séparés** | dev / staging / prod dans des workspaces ou repos distincts |
| **State remote et verrouillé** | Évite les conflits en équipe |

---

## 9. À retenir pour un admin sys

- **IaC** = infrastructure décrite en code, versionnée dans Git, reproductible.
- **Terraform** provisionne, **Ansible** configure : les deux sont complémentaires.
- **GitOps** = Git comme source de vérité unique + réconciliation automatique.
- L'**idempotence** est clé : un outil IaC doit pouvoir être relancé sans effet de bord.
- La **dérive** est l'ennemi : tout changement manuel hors Git crée de l'entropie.

---

**Suite naturelle →** [[terraform_fondamentaux]] | [[ansible_fondamentaux]]
