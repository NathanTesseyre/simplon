---
title: "Introduction à la virtualisation"
tags:
  - virtualisation
  - VM
  - hyperviseur
section: 04-Environnements
domaine: Virtualisation
statut: actif
liens_connexes:
  - [[hyperviseurs]]
  - [[vm_vs_conteneurs]]
  - [[usages_limites]]
---

# 📘 Introduction à la virtualisation

## Définition

La virtualisation consiste à faire croire à un logiciel qu'il dispose d'un matériel dédié, alors qu'il partage en réalité des ressources physiques avec d'autres environnements. Le logiciel qui opère cette illusion s'appelle un **hyperviseur**.

Concrètement : un seul serveur physique peut héberger 10 machines virtuelles, chacune avec son propre OS, ses propres processus, son propre réseau — totalement isolées les unes des autres.

```
┌─────────────────────────────────────────────┐
│              Serveur physique               │
│  CPU: 32 cœurs │ RAM: 256 Go │ Stockage: 4 To │
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  VM #1   │  │  VM #2   │  │  VM #3   │  │
│  │ Ubuntu   │  │ Windows  │  │ Debian   │  │
│  │ 4 cœurs  │  │ 8 cœurs  │  │ 2 cœurs  │  │
│  │ 16 Go RAM│  │ 32 Go RAM│  │  8 Go RAM│  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                                             │
│              Hyperviseur                    │
└─────────────────────────────────────────────┘
```

## Virtualisation vs émulation

Ces deux concepts sont souvent confondus, mais ils sont très différents :

**Virtualisation** : le CPU de la VM est le même que celui de l'hôte. Les instructions s'exécutent directement (ou presque) sur le matériel. C'est rapide — la perte de performance est généralement inférieure à 5%.

**Émulation** : un logiciel simule un matériel complètement différent, instruction par instruction. C'est lent (10x à 100x) mais utile pour faire tourner un programme ARM sur un CPU x86, par exemple. QEMU en mode émulation, ou l'émulateur Game Boy sont des exemples.

**Paravirtualisation** : cas intermédiaire où l'OS invité est modifié pour dialoguer directement avec l'hyperviseur. Plus rapide que l'émulation, utilisé notamment par Xen.

## Pourquoi la virtualisation existe

Avant la virtualisation, chaque application critique avait son propre serveur physique dédié. Les raisons :

- **Isolation** : un bug ou une faille dans une appli ne pouvait pas affecter les autres
- **Stabilité** : pas de conflit entre dépendances
- **Sécurité** : périmètre limité

Le problème : des milliers de serveurs à 10–15% d'utilisation CPU. Énorme gaspillage. La virtualisation a permis de **consolider** ces serveurs : 10 serveurs physiques remplacés par 1, hébergeant 10 VMs.

## Ce que la virtualisation permet concrètement

**En développement :**
- Tester une appli sur plusieurs OS sans changer de machine
- Reproduire un environnement de production exact sur son laptop
- Casser une VM sans conséquence, la recréer en secondes

**En production :**
- Isoler les services (base de données, API, frontend) sur des VMs séparées
- Allouer exactement les ressources nécessaires à chaque service
- Migrer une VM d'un serveur physique à un autre à chaud (vMotion chez VMware)

**En sécurité :**
- Ouvrir des fichiers suspects dans une VM jetable (sandbox)
- Tester des malwares sans risque pour la machine hôte
- Isoler un réseau de test du réseau de production

## Les grandes familles de virtualisation

| Type | Ce qui est virtualisé | Exemples |
|------|----------------------|----------|
| Virtualisation système | CPU, RAM, stockage, réseau complets | VMware, VirtualBox, KVM |
| Virtualisation réseau | Réseaux, routeurs, switches virtuels | VPC AWS, NSX, Open vSwitch |
| Virtualisation stockage | Pools de disques, volumes logiques | LVM, SAN, EBS |
| Conteneurisation | Espace utilisateur de l'OS | Docker, Podman, LXC |

## Historique rapide

- **1960s** : IBM invente la virtualisation sur ses mainframes (IBM System/360). L'objectif : partager des machines hors de prix entre plusieurs clients.
- **1999** : VMware lance VMware Workstation — la virtualisation arrive sur PC grand public.
- **2003** : Xen, premier hyperviseur open source de type 1.
- **2007** : KVM intégré au noyau Linux — la virtualisation devient native sur Linux.
- **2013** : Docker démocratise les conteneurs — une alternative plus légère à la VM pour beaucoup de cas d'usage.
- **Aujourd'hui** : AWS, GCP, Azure font tourner des millions de VMs. La virtualisation est le socle invisible du cloud.

---

### ✅ À retenir

- La virtualisation crée des environnements isolés sur un matériel partagé.
- L'hyperviseur est la couche qui rend cela possible.
- La virtualisation ≠ émulation : pas de simulation instruction par instruction, les perfs restent bonnes.
- Deux grandes ères : serveurs physiques → VMs → conteneurs (chaque étape plus légère).

**Lire ensuite →** [[hyperviseurs]] pour comprendre comment l'hyperviseur fonctionne, puis [[vm_vs_conteneurs]] pour le choix VM vs Docker.
