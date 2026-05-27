---
title: "VM vs conteneurs"
tags:
  - virtualisation
  - Docker
  - VM
  - conteneurs
section: 04-Environnements
domaine: Virtualisation
statut: actif
liens_connexes:
  - [[Docker]]
  - [[hyperviseurs]]
  - [[usages_limites]]
---

# 📘 VM vs Conteneurs

## L'isolation : deux approches différentes

Les VMs et les conteneurs répondent au même besoin fondamental — isoler des applications — mais à des niveaux différents de la pile logicielle.

**VM : isolation au niveau matériel**

Chaque VM embarque un OS complet. L'hyperviseur virtualise le CPU, la RAM, le réseau, le stockage. La VM ne sait même pas qu'elle tourne sur du matériel partagé.

```
┌─────────────────────────────────────────────┐
│              Matériel physique              │
├─────────────────────────────────────────────┤
│               Hyperviseur                   │
├────────────────┬────────────────────────────┤
│     VM #1      │          VM #2             │
│  ┌──────────┐  │      ┌──────────┐          │
│  │   App    │  │      │   App    │          │
│  ├──────────┤  │      ├──────────┤          │
│  │ OS Linux │  │      │OS Windows│          │
│  │ (kernel) │  │      │ (kernel) │          │
│  └──────────┘  │      └──────────┘          │
└────────────────┴────────────────────────────┘
```

**Conteneurs : isolation au niveau OS**

Les conteneurs partagent le noyau de l'OS hôte. Chaque conteneur a son propre espace utilisateur (fichiers, processus, réseau) mais utilise le même kernel Linux.

```
┌─────────────────────────────────────────────┐
│              Matériel physique              │
├─────────────────────────────────────────────┤
│            OS hôte (Linux)                  │
│              Noyau partagé                  │
├───────────┬───────────┬─────────────────────┤
│ Conteneur │ Conteneur │     Conteneur        │
│    #1     │    #2     │        #3            │
│  App Node │  App Python│   App Java          │
│  + libs   │  + libs   │   + libs            │
└───────────┴───────────┴─────────────────────┘
```

## Comparatif détaillé

| Critère | Machine virtuelle | Conteneur |
|---------|------------------|-----------|
| **Isolation** | Complète (noyau séparé) | Partielle (noyau partagé) |
| **OS** | OS complet embarqué | Uniquement les libs nécessaires |
| **Taille** | Plusieurs Go (image OS) | Quelques Mo à centaines de Mo |
| **Démarrage** | 30s à plusieurs minutes | Quelques secondes, parfois ms |
| **RAM utilisée** | Plusieurs Go par VM | Quelques dizaines de Mo |
| **Densité** | ~10 VMs par serveur physique | Centaines de conteneurs |
| **Sécurité** | Très forte (kernel isolé) | Bonne, mais failles kernel partagées |
| **Portabilité** | Bonne (OVA, VMDK) | Excellente (image Docker) |
| **Compatibilité OS** | N'importe quel OS invité | Linux uniquement (nativement) |
| **Persistance** | Disque virtuel persistant | Éphémère par défaut |

## Quand choisir une VM

**Applications nécessitant un OS différent de l'hôte**
Faire tourner Windows sur un hôte Linux (ou l'inverse). Les conteneurs ne peuvent pas faire ça.

**Isolation de sécurité maximale**
Un conteneur compromis peut potentiellement attaquer le kernel hôte. Une VM compromise reste confinée à son propre kernel.

**Tests de configuration système**
Tester des scripts d'installation, des configs réseau, des règles iptables — sans risque pour la machine hôte.

**Environnements de formation**
Chaque apprenant a sa propre VM isolée. Si elle est cassée, on la recrée depuis un snapshot propre.

**Applications legacy**
Une vieille appli qui tourne uniquement sur Windows Server 2008 ou sur une vieille distribution Linux.

## Quand choisir des conteneurs

**Microservices**
Chaque service (API, worker, scheduler) dans son propre conteneur. Déploiement, mise à l'échelle et mise à jour indépendants.

**CI/CD**
Les pipelines Jenkins/GitLab CI créent des conteneurs jetables pour chaque build. Propres, reproductibles, rapides.

**Développement local**
`docker-compose up` démarre toute la stack (API + BDD + cache) en quelques secondes. Fini le "ça marche sur ma machine".

**Scalabilité horizontale**
Dupliquer 50 conteneurs d'une API derrière un load balancer est trivial. Dupliquer 50 VMs est lourd.

**Cloud-native**
Kubernetes orchestre des conteneurs, pas des VMs. Si tu vises du Kubernetes, pense conteneurs.

## Peuvent coexister

VM et conteneurs ne s'opposent pas — ils se complètent souvent :

```
┌─────────────────────────────┐
│      VM (Ubuntu Server)     │
│                             │
│  ┌──────┐ ┌──────┐ ┌──────┐│
│  │Cont. │ │Cont. │ │Cont. ││
│  │ API  │ │  DB  │ │Redis ││
│  └──────┘ └──────┘ └──────┘│
│         Docker Engine       │
└─────────────────────────────┘
```

C'est exactement ce que font AWS, GCP et Azure : tes conteneurs Docker tournent dans des VMs (EC2, Compute Engine) gérées par le cloud provider. Tu ne vois que les conteneurs, mais les VMs sont là en dessous.

## Les namespaces et cgroups — comment Docker isole sans VM

Docker utilise deux mécanismes du noyau Linux :

**Namespaces** : isolent ce que voit le conteneur.
- `pid` : le conteneur a sa propre liste de processus (PID 1 = son processus principal)
- `net` : interface réseau virtuelle dédiée
- `mnt` : système de fichiers isolé
- `uts` : hostname propre
- `user` : utilisateurs isolés

**cgroups** (control groups) : limitent ce que consomme le conteneur.
```bash
# Limiter un conteneur à 512 Mo de RAM et 1 CPU
docker run --memory="512m" --cpus="1.0" nginx
```

Sans ces deux mécanismes, Docker n'existerait pas. Ce sont des fonctionnalités du noyau Linux, pas de Docker lui-même.

---

### ✅ À retenir

- VM = OS complet isolé → isolation maximale, démarrage lent, lourd
- Conteneur = processus isolé → léger, rapide, moins isolé
- Conteneurs = Linux uniquement nativement (Docker Desktop émule sur Mac/Windows)
- En production : souvent les deux — conteneurs dans des VMs
- Choisir selon le besoin : sécurité maximale → VM ; scalabilité et rapidité → conteneurs

**Lire ensuite →** [[Docker]] pour la prise en main pratique des conteneurs.
