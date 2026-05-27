---
title: "Terraform — Fondamentaux"
tags:
  - terraform
  - IaC
  - infrastructure
  - devops
section: 05-DevOps
domaine: Terraform
statut: actif
liens_connexes:
  - [[terraform_virtualbox_ova_vs_vdi]]
  - [[fiche_architecture_infra]]
  - [[cloud_virtualisation]]
---

# 📘 Fiche : Terraform — Fondamentaux

---

## 1. Qu'est-ce que Terraform ?

**Terraform** (par HashiCorp) est un outil d'**Infrastructure as Code (IaC)** : il permet de décrire une infrastructure dans des fichiers texte, puis de la créer, modifier ou détruire de manière automatisée et reproductible.

### Pourquoi IaC ?

| Sans IaC | Avec IaC |
|---|---|
| Infra créée à la main (console, CLI) | Infra décrite dans des fichiers versionnés |
| Difficile à reproduire | Reproductible à l'identique |
| Pas d'historique des changements | Versionnable avec Git |
| Risque d'erreur humaine | Automatisable en CI/CD |

---

## 2. Concepts clés

### Provider
Plugin qui permet à Terraform de communiquer avec une plateforme (AWS, Azure, GCP, Docker, VirtualBox...).

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "eu-west-3"
}
```

### Resource
Un objet d'infrastructure à créer (VM, bucket, réseau, DNS...).

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}
```

### State
Terraform maintient un **fichier d'état** (`terraform.tfstate`) qui représente l'infrastructure réelle telle qu'il la connaît.  
👉 C'est ce fichier qui permet à Terraform de savoir quoi créer, modifier ou détruire lors du prochain `apply`.

> ⚠️ Ne jamais modifier `terraform.tfstate` à la main.

### Plan
Aperçu des changements que Terraform va appliquer, **sans toucher à l'infrastructure**.

### Apply
Exécute les changements décrits dans le plan.

---

## 3. Structure d'un projet Terraform

```
projet/
├── main.tf           # ressources principales
├── variables.tf      # déclaration des variables
├── outputs.tf        # valeurs exportées après apply
├── terraform.tfvars  # valeurs des variables (à gitignorer si secrets)
└── providers.tf      # configuration des providers
```

> 💡 Pour les petits projets, tout peut tenir dans un seul `main.tf`.

---

## 4. Variables et Outputs

### Variables (input)
```hcl
# variables.tf
variable "region" {
  description = "Région AWS"
  type        = string
  default     = "eu-west-3"
}

variable "instance_type" {
  type = string
}
```

```hcl
# terraform.tfvars
instance_type = "t3.micro"
```

Utilisation dans le code :
```hcl
provider "aws" {
  region = var.region
}
```

### Outputs (valeurs exportées)
```hcl
# outputs.tf
output "public_ip" {
  description = "IP publique de l'instance"
  value       = aws_instance.web.public_ip
}
```

Après un `apply`, les outputs s'affichent dans le terminal. Utiles pour récupérer une IP, une URL, un identifiant.

---

## 5. Workflow de base

```
terraform init       → télécharge les providers, initialise le projet
terraform validate   → vérifie la syntaxe des fichiers .tf
terraform plan       → aperçu des changements (rien n'est créé)
terraform apply      → applique les changements (demande confirmation)
terraform destroy    → détruit toute l'infrastructure gérée
```

### Cycle typique

```
Écrire .tf → init → plan → relire le plan → apply
```

> 💡 `terraform apply -auto-approve` saute la confirmation — à réserver aux pipelines CI/CD.

---

## 6. State local vs State distant

### State local (défaut)
Le fichier `terraform.tfstate` est stocké localement.  
⚠️ Problème en équipe : chaque personne a son propre state → conflits.

### State distant (recommandé en équipe)
Le state est stocké dans un backend partagé (S3 + DynamoDB, Terraform Cloud, GitLab...).

```hcl
terraform {
  backend "s3" {
    bucket         = "mon-terraform-state"
    key            = "projet/terraform.tfstate"
    region         = "eu-west-3"
    dynamodb_table = "terraform-locks"
  }
}
```

Le **verrouillage** (DynamoDB ou équivalent) empêche deux personnes d'appliquer en même temps.

---

## 7. Commandes utiles

| Commande | Rôle |
|---|---|
| `terraform init` | Initialise le projet, télécharge les providers |
| `terraform validate` | Vérifie la syntaxe |
| `terraform fmt` | Formate les fichiers `.tf` |
| `terraform plan` | Aperçu des changements |
| `terraform apply` | Applique les changements |
| `terraform destroy` | Détruit l'infrastructure |
| `terraform output` | Affiche les outputs |
| `terraform state list` | Liste les ressources dans le state |
| `terraform import` | Importe une ressource existante dans le state |
| `terraform taint` | Force la recréation d'une ressource au prochain apply |

---

## 8. Bonnes pratiques

- **Toujours relire le plan** avant d'appliquer — vérifier ce qui sera détruit.
- **Versionner les fichiers `.tf`** avec Git, mais **ignorer** `terraform.tfstate`, `terraform.tfstate.backup`, `.terraform/` et `terraform.tfvars` (si secrets).
- **Utiliser un state distant** dès qu'on travaille en équipe.
- **Découper en modules** pour réutiliser du code Terraform.
- Garder les ressources dans des fichiers séparés par domaine (`network.tf`, `compute.tf`, `storage.tf`).

`.gitignore` recommandé :
```
.terraform/
*.tfstate
*.tfstate.backup
terraform.tfvars
```

---

## 9. ✅ À retenir

- Terraform décrit l'infrastructure dans des fichiers `.tf` → **reproductible et versionnable**.
- **Provider** = connecteur vers une plateforme ; **Resource** = objet à créer.
- **State** = mémoire de Terraform sur l'état réel de l'infra.
- Workflow : `init` → `plan` → `apply`.
- En équipe : **state distant** obligatoire pour éviter les conflits.
