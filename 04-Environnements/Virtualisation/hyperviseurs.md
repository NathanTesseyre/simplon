---
title: "Les hyperviseurs"
tags:
  - virtualisation
  - hyperviseur
  - VMware
  - KVM
  - VirtualBox
section: 04-Environnements
domaine: Virtualisation
statut: actif
liens_connexes:
  - [[intro_virtualisation]]
  - [[vm_vs_conteneurs]]
  - [[cloud_virtualisation]]
---

# 📘 Les hyperviseurs

## Rôle de l'hyperviseur

L'hyperviseur est le logiciel qui crée et gère les machines virtuelles. Il intercepte les instructions des VMs et les traduit pour le matériel physique. Il arbitre aussi l'accès aux ressources partagées : CPU, RAM, réseau, stockage.

Sans hyperviseur, plusieurs OS ne peuvent pas coexister sur le même matériel sans se marcher dessus.

## Type 1 — Bare metal

L'hyperviseur s'installe **directement sur le matériel**, sans OS hôte en dessous. Il est lui-même l'OS de la machine.

```
┌──────────────────────────────────┐
│  VM Linux  │  VM Windows  │ VM BSD│
├──────────────────────────────────┤
│         Hyperviseur Type 1       │
├──────────────────────────────────┤
│         Matériel physique        │
└──────────────────────────────────┘
```

**Avantages :** performances maximales, accès direct au matériel, pas d'OS hôte à maintenir.

**Inconvénients :** configuration plus complexe, moins pratique pour un usage desktop.

**Usage typique :** datacenters, serveurs de production, cloud providers.

**Exemples :**
- **VMware ESXi** — standard de l'entreprise, très mature, payant
- **Microsoft Hyper-V Server** — gratuit, intégré à Windows Server
- **Xen** — open source, utilisé par AWS (historiquement) et Citrix
- **KVM** (Kernel-based Virtual Machine) — intégré au noyau Linux depuis 2007, utilisé par GCP, OpenStack, Proxmox

## Type 2 — Hosted

L'hyperviseur s'installe **comme une application** sur un OS hôte existant (Windows, macOS, Linux). Il dépend de l'OS hôte pour accéder au matériel.

```
┌──────────────────────────────────┐
│  VM Linux  │  VM Windows  │ VM BSD│
├──────────────────────────────────┤
│         Hyperviseur Type 2       │
├──────────────────────────────────┤
│    OS hôte (Windows / macOS)     │
├──────────────────────────────────┤
│         Matériel physique        │
└──────────────────────────────────┘
```

**Avantages :** installation simple, pratique sur un laptop, bonne intégration avec l'OS hôte.

**Inconvénients :** overhead supplémentaire dû à l'OS hôte, performances légèrement inférieures.

**Usage typique :** développement local, tests, formations, sandbox.

**Exemples :**
- **VirtualBox** — open source, gratuit, multiplateforme. Le plus utilisé en formation.
- **VMware Workstation / Fusion** — plus performant que VirtualBox, payant
- **Parallels Desktop** — macOS uniquement, excellent pour faire tourner Windows sur Mac

## KVM — focus pratique

KVM est aujourd'hui le standard open source. Il transforme le noyau Linux en hyperviseur de type 1 (même s'il tourne sur Linux, il s'appuie sur des extensions CPU matérielles — Intel VT-x ou AMD-V — pour une isolation quasi-native).

**Vérifier si KVM est disponible sur ta machine :**
```bash
# Vérifier les extensions CPU
grep -E 'vmx|svm' /proc/cpuinfo

# vmx = Intel VT-x
# svm = AMD-V
# Aucun résultat = virtualisation matérielle désactivée (vérifier le BIOS)
```

**Installer KVM sur Debian/Ubuntu :**
```bash
sudo apt install qemu-kvm libvirt-daemon-system virt-manager bridge-utils

# Ajouter l'utilisateur au groupe libvirt
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER

# Vérifier que le service tourne
sudo systemctl status libvirtd
```

**Gérer les VMs en ligne de commande avec virsh :**
```bash
# Lister les VMs
virsh list --all

# Démarrer / arrêter une VM
virsh start nom-vm
virsh shutdown nom-vm
virsh destroy nom-vm   # arrêt forcé

# Infos sur une VM
virsh dominfo nom-vm

# Snapshot
virsh snapshot-create-as nom-vm snap1 "avant mise à jour"
virsh snapshot-list nom-vm
virsh snapshot-revert nom-vm snap1
```

## VirtualBox — focus pratique

Le plus accessible pour débuter. Interface graphique claire, gestion des snapshots simple.

**Installer les Guest Additions** (à faire dans chaque VM pour de meilleures perfs et le copier-coller) :
```bash
# Dans la VM, après avoir monté le CD Guest Additions (menu Périphériques)
sudo apt install build-essential dkms linux-headers-$(uname -r)
sudo sh /media/cdrom/VBoxLinuxAdditions.run
```

**Gestion en ligne de commande avec VBoxManage :**
```bash
# Lister les VMs
VBoxManage list vms
VBoxManage list runningvms

# Démarrer en mode headless (sans interface graphique)
VBoxManage startvm "nom-vm" --type headless

# Snapshot
VBoxManage snapshot "nom-vm" take "snap-avant-install"
VBoxManage snapshot "nom-vm" restore "snap-avant-install"

# Modifier les ressources
VBoxManage modifyvm "nom-vm" --memory 4096 --cpus 2
```

## Comparatif rapide

| Critère | VirtualBox | VMware Workstation | KVM |
|---------|-----------|-------------------|-----|
| Prix | Gratuit | Payant | Gratuit |
| OS hôte | Win/Mac/Linux | Win/Linux | Linux uniquement |
| Performances | Bonnes | Très bonnes | Excellentes |
| Facilité | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Usage prod | Non | Non | Oui |
| Snapshots | Oui | Oui | Oui (virsh) |

## Fonctionnalités clés à connaître

**Snapshots** : photo de l'état d'une VM à un instant T. Permet de revenir en arrière si une manipulation casse quelque chose. Indispensable avant toute opération risquée.

**Clones** : copie complète d'une VM. Utile pour déployer rapidement plusieurs VMs identiques.

**Réseaux virtuels** : chaque hyperviseur propose plusieurs modes réseau :
- **NAT** : la VM accède à internet via l'IP de l'hôte (simple, mais pas accessible depuis l'extérieur)
- **Bridge** : la VM a sa propre IP sur le réseau local (accessible comme une vraie machine)
- **Host-only** : réseau isolé entre VMs et hôte uniquement (sans internet)
- **Internal** : réseau isolé entre VMs uniquement

**Migration à chaud** (type 1 uniquement) : déplacer une VM d'un serveur physique à un autre sans l'éteindre. Fonctionnalité clé en production pour la maintenance sans interruption de service.

---

### ✅ À retenir

- **Type 1** (bare metal) = production, serveurs. KVM, ESXi, Hyper-V.
- **Type 2** (hosted) = dev, tests, formation. VirtualBox, VMware Workstation.
- KVM est le standard Linux open source — utilisé par la plupart des clouds.
- VirtualBox est le plus accessible pour commencer.
- Les snapshots sont indispensables : toujours en faire avant une manipulation risquée.

**Lire ensuite →** [[vm_vs_conteneurs]] pour comprendre quand choisir une VM plutôt que Docker.
