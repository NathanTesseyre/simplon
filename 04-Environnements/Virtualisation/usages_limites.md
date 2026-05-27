---
title: "Usages et limites de la virtualisation"
tags:
  - virtualisation
  - cas-usage
  - limites
  - choix-technique
section: 04-Environnements
domaine: Virtualisation
statut: actif
liens_connexes:
  - [[vm_vs_conteneurs]]
  - [[cloud_virtualisation]]
  - [[hyperviseurs]]
---

# 📘 Usages et limites de la virtualisation

## Cas d'usage en production

### Consolidation de serveurs

Avant la virtualisation, chaque application avait son serveur physique dédié. Un serveur web tournait à 8% de CPU, un serveur mail à 12%, etc. Énorme gaspillage.

Avec la virtualisation, un seul serveur physique héberge plusieurs VMs. En pratique, un ratio de 10:1 à 20:1 est courant (10 à 20 VMs par hôte physique). Les économies sont massives : matériel, électricité, climatisation, espace datacenter.

### Isolation des services

Séparer les services en VMs indépendantes réduit la surface d'attaque et facilite la maintenance :

```
Avant :         Après :
┌──────────┐    ┌────────┐ ┌────────┐ ┌────────┐
│  Serveur │    │VM: Web │ │VM: DB  │ │VM: Mail│
│  Web     │    │        │ │        │ │        │
│  DB      │    └────────┘ └────────┘ └────────┘
│  Mail    │         ↕         ↕
│  ...     │    Un bug/faille dans Web
└──────────┘    n'affecte pas DB
```

### Haute disponibilité et reprise après sinistre

Les hyperviseurs de type 1 (VMware ESXi, KVM via oVirt) permettent :
- **vMotion / Live migration** : déplacer une VM d'un hôte physique à un autre sans coupure (pour maintenance matérielle)
- **HA (High Availability)** : si un hôte tombe en panne, ses VMs redémarrent automatiquement sur un autre hôte
- **Snapshots** : sauvegarder l'état complet d'une VM avant une mise à jour risquée

### Environnements de développement et test

Chaque développeur peut avoir une VM identique à la production. Plus de "ça marche sur ma machine" — si la VM est identique, le comportement le sera aussi.

Les équipes QA testent sur des VMs clonées depuis la production. Les tests sont reproductibles et n'affectent pas la prod.

### Sandbox sécurité

Ouvrir un fichier suspect (malware potentiel, pièce jointe douteuse) dans une VM jetable. Si la VM est compromise, on la supprime. L'hôte est intact.

Certains outils de sécurité (Cuckoo Sandbox, Any.run) analysent automatiquement des fichiers dans des VMs éphémères.

### Formation et labs

Fournir à chaque apprenant un environnement Linux isolé, reproductible, et réinitialisable en cas de manipulation catastrophique. C'est précisément l'usage de ce cours.

## Limites et inconvénients

### Overhead de ressources

Chaque VM embarque un OS complet : noyau, services système, démons de fond. Une VM Ubuntu minimale consomme environ 500 Mo de RAM avant même de lancer une application.

Avec 10 VMs sur un hôte, c'est 5 Go de RAM "gaspillés" en OS. Sur un hôte avec 32 Go, c'est ~15% des ressources consommées par les OS invités eux-mêmes.

Les conteneurs n'ont pas ce problème : ils partagent le noyau hôte.

### Latence I/O

Les opérations disque et réseau passent par une couche de virtualisation supplémentaire. Pour des applications très sensibles à la latence (bases de données haute performance, trading), cet overhead peut poser problème.

Solutions : pass-through PCI (accès direct au matériel pour une VM), stockage NVMe virtuel, SR-IOV pour le réseau.

### Complexité de gestion à l'échelle

Gérer 5 VMs est simple. Gérer 500 VMs nécessite des outils d'orchestration (vCenter, oVirt, OpenStack, Proxmox avec cluster). La configuration réseau (VLANs, bridges virtuels) devient vite complexe.

### Temps de démarrage

Une VM met 30 secondes à plusieurs minutes à démarrer. Pour des architectures qui doivent scaler rapidement (ex: API qui reçoit un pic de trafic), c'est trop lent. Les conteneurs démarrent en secondes.

### Licences OS

Chaque VM Windows nécessite une licence Windows. Sur 50 VMs Windows, c'est 50 licences. Les modèles de licence Microsoft pour la virtualisation sont complexes.

## Quand ne pas utiliser de VMs

| Situation | Mieux adapté |
|-----------|-------------|
| Scaler rapidement une API | Conteneurs (Docker + Kubernetes) |
| Développement quotidien d'une app web | Docker Compose |
| Déployer des microservices | Conteneurs |
| Très faibles ressources disponibles | Conteneurs ou bare metal |
| Applications stateless cloud-native | Conteneurs + orchestrateur |

## Tendances actuelles

La virtualisation classique (VM) reste dominante en entreprise pour les workloads legacy et les applications avec des exigences d'isolation strictes.

Pour les nouveaux développements, les conteneurs (Docker, Kubernetes) ont largement pris le dessus pour les applications cloud-native.

La ligne de partage tend à se faire ainsi :
- **Infrastructure système** (serveurs, bases de données, applis legacy) → VMs
- **Applications modernes, microservices, CI/CD** → conteneurs

Les deux coexistent souvent : des conteneurs Docker qui tournent dans des VMs hébergées dans le cloud.

---

### ✅ À retenir

- VMs = idéal pour l'isolation forte, la haute disponibilité, les environnements de test reproductibles
- Principales limites : overhead RAM/CPU, temps de démarrage, complexité à l'échelle
- Conteneurs = complémentaires, pas en remplacement total, selon le cas d'usage
- En entreprise : souvent les deux selon les besoins

**Voir aussi →** [[vm_vs_conteneurs]] pour le détail du choix VM vs Docker, [[cloud_virtualisation]] pour la dimension cloud.
