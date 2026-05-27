---
title: "Cloud et virtualisation"
tags:
  - virtualisation
  - cloud
  - IaaS
  - PaaS
  - SaaS
  - AWS
  - GCP
  - Azure
section: 04-Environnements
domaine: Virtualisation
statut: actif
liens_connexes:
  - [[fiche_architecture_infra]]
  - [[Hosting]]
  - [[hyperviseurs]]
  - [[usages_limites]]
---

# 📘 Cloud et virtualisation

## Le cloud repose entièrement sur la virtualisation

Quand tu lances une instance EC2 sur AWS ou une VM sur GCP, tu loues en réalité une machine virtuelle qui tourne sur un serveur physique appartenant au cloud provider. La virtualisation est la technologie qui rend cela possible et économiquement viable.

```
┌──────────────────────────────────────────────────────┐
│              Datacenter AWS (ex: eu-west-1)          │
│                                                      │
│  ┌──────────────┐  ┌──────────────┐                  │
│  │ Serveur phy. │  │ Serveur phy. │  ...             │
│  │              │  │              │                  │
│  │ ┌──┐ ┌──┐   │  │ ┌──┐ ┌──┐   │                  │
│  │ │VM│ │VM│   │  │ │VM│ │VM│   │                  │
│  │ └──┘ └──┘   │  │ └──┘ └──┘   │                  │
│  │  KVM / Xen  │  │  KVM / Xen  │                  │
│  └──────────────┘  └──────────────┘                  │
│                                                      │
│  Tu vois ça →  [EC2 Instance]  ← toi               │
└──────────────────────────────────────────────────────┘
```

AWS utilise historiquement Xen, puis a migré vers son propre hyperviseur basé sur KVM (AWS Nitro). GCP et Azure utilisent KVM et Hyper-V respectivement.

## Les modèles de service cloud

### IaaS — Infrastructure as a Service

Tu loues de l'infrastructure brute : machines virtuelles, stockage, réseau. Tu gères tout ce qui est au-dessus : OS, runtime, applis.

**Exemples :** AWS EC2, Google Compute Engine, Azure Virtual Machines, OVHcloud VPS

**Tu gères :** OS, mises à jour, sécurité, runtime, appli
**Le provider gère :** matériel physique, hyperviseur, réseau physique, datacenter

**Cas d'usage :** migration "lift and shift" depuis un datacenter on-premise, contrôle total de l'environnement, workloads qui ne rentrent pas dans les cases PaaS.

### PaaS — Platform as a Service

Tu fournis le code, le provider gère tout le reste (OS, runtime, scaling, déploiement).

**Exemples :** Heroku, Google App Engine, AWS Elastic Beanstalk, Vercel, Railway

**Tu gères :** le code de ton appli et sa configuration
**Le provider gère :** tout le reste (infra, OS, runtime, déploiement, scaling)

**Cas d'usage :** déploiement rapide sans vouloir gérer l'infra, startups qui veulent se concentrer sur le produit.

### SaaS — Software as a Service

Tu utilises directement une application hébergée. Aucune gestion d'infra.

**Exemples :** Gmail, Notion, Slack, GitHub, Figma

**Tu gères :** tes données et ta configuration dans l'appli
**Le provider gère :** tout

**Cas d'usage :** outils de productivité, collaboration — pas du tout du hosting applicatif.

### FaaS — Function as a Service (serverless)

Tu déploies des fonctions individuelles, exécutées à la demande. Tu paies à l'exécution, pas à l'heure.

**Exemples :** AWS Lambda, Google Cloud Functions, Azure Functions, Cloudflare Workers

**Avantages :** scaling automatique de 0 à l'infini, paiement à l'usage, aucune gestion de serveur
**Limites :** cold start (latence au premier appel), durée d'exécution limitée, stateless obligatoire

```javascript
// Exemple de fonction AWS Lambda
exports.handler = async (event) => {
    const name = event.queryStringParameters?.name || 'World';
    return {
        statusCode: 200,
        body: JSON.stringify({ message: `Hello, ${name}!` })
    };
};
```

## Virtualisation réseau dans le cloud

Les clouds ne virtualisent pas que le compute — le réseau aussi.

**VPC (Virtual Private Cloud)** : réseau privé virtuel isolé dans le cloud. Tes ressources (VMs, bases de données) sont dans ce réseau, inaccessibles de l'extérieur sauf ce que tu exposes explicitement.

```
Internet
    │
    ▼
┌─────────────────────────────────────┐
│             VPC (10.0.0.0/16)       │
│                                     │
│  ┌──────────────────────────────┐   │
│  │  Subnet public (10.0.1.0/24) │   │
│  │  [Load Balancer] [Bastion]   │   │
│  └──────────────┬───────────────┘   │
│                 │                   │
│  ┌──────────────▼───────────────┐   │
│  │  Subnet privé (10.0.2.0/24) │   │
│  │  [API] [Worker] [BDD]       │   │
│  └──────────────────────────────┘   │
└─────────────────────────────────────┘
```

**Security Groups / Firewall rules** : règles réseau attachées aux VMs. Définissent quels ports sont accessibles depuis quelles sources.

**Load Balancer managé** : le provider gère la haute disponibilité et la répartition du trafic entre tes instances.

## Virtualisation du stockage

**Volumes persistants (EBS sur AWS, Persistent Disk sur GCP)** : disques virtuels attachables à une VM. Le volume persiste même si la VM est arrêtée ou supprimée.

**Object storage (S3, GCS)** : stockage de fichiers accessible via API HTTP. Pas un système de fichiers traditionnel. Idéal pour les médias, backups, assets statiques.

**Snapshots** : copie d'un volume à un instant T. Utilisé pour les sauvegardes et pour créer de nouvelles VMs depuis une image existante.

## Vers le serverless et le cloud-native

L'évolution logique de la virtualisation :

```
Bare metal → VM → Conteneurs → Serverless
   (100%)    (85%)    (40%)      (5% overhead)
  overhead  overhead  overhead
```

Chaque étape réduit l'overhead opérationnel et augmente l'abstraction. Le serverless représente la virtualisation poussée à l'extrême : tu ne vois même plus le serveur.

Mais chaque niveau a ses cas d'usage. Le bare metal reste pertinent pour les bases de données haute performance. Les VMs restent pertinentes pour les applis legacy. Les conteneurs dominent le cloud-native. Le serverless est idéal pour les tâches événementielles.

## Régions et zones de disponibilité

Un concept clé du cloud lié à la virtualisation :

**Région** : zone géographique (ex: `eu-west-1` = Irlande chez AWS). Choisir une région proche de tes utilisateurs réduit la latence.

**Zone de disponibilité (AZ)** : datacenter indépendant au sein d'une région. Déployer tes VMs sur plusieurs AZs garantit qu'une panne dans un datacenter n'affecte pas ton service.

---

### ✅ À retenir

- Le cloud = virtualisation à grande échelle, mutualisée, facturée à l'usage
- IaaS = VMs louées (EC2, GCP, Azure VM) → tu gères l'OS
- PaaS = plateforme de déploiement (Heroku, App Engine) → tu gères le code
- SaaS = application clé en main → tu gères tes données
- Serverless = functions à la demande, scaling automatique, paiement à l'exécution
- VPC = ton réseau privé virtuel dans le cloud

**Voir aussi →** [[Hosting]] pour le comparatif hébergement mutualisé / VPS / cloud.
