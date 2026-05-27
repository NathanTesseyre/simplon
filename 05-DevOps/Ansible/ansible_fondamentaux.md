---
title: "Ansible — Fondamentaux"
tags:
  - ansible
  - configuration-management
  - IaC
  - devops
  - automatisation
section: 05-DevOps
domaine: Ansible
statut: actif
liens_connexes:
  - [[terraform_fondamentaux]]
  - [[shell_scripting]]
  - [[reseau_linux]]
---

# 📘 Fiche : Ansible — Fondamentaux

---

## 1. Pourquoi Ansible ?

Administrer plusieurs serveurs manuellement (connexion SSH + commandes) est lent et source d'erreurs. Les scripts shell sont mieux, mais difficiles à maintenir et pas idempotents.

**Ansible** est un outil de **gestion de configuration** : il permet d'automatiser la configuration de serveurs via des fichiers YAML lisibles, sans agent à installer sur les machines cibles.

| Caractéristique | Détail |
|---|---|
| **Agentless** | Fonctionne via SSH — rien à installer sur les serveurs cibles |
| **Idempotent** | Exécuter 2 fois le même playbook → même résultat, sans effets de bord |
| **Déclaratif** | On décrit l'état voulu, pas les étapes pour y arriver |
| **Lisible** | YAML — compréhensible sans être développeur |

### Terraform vs Ansible

| | Terraform | Ansible |
|---|---|---|
| Rôle | Créer l'infrastructure (VMs, réseaux, buckets) | Configurer les serveurs (install, config, déploiement) |
| Quand | Avant | Après |

> 💡 Les deux sont complémentaires : Terraform provisionne, Ansible configure.

---

## 2. Concepts clés

### Inventory
Liste des machines à gérer, regroupées par rôle.

```ini
# inventory.ini
[webservers]
web1.exemple.com
web2.exemple.com

[databases]
db1.exemple.com

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

### Playbook
Fichier YAML qui décrit une série de **tasks** à exécuter sur des hôtes.

### Task
Une action unitaire : installer un paquet, copier un fichier, démarrer un service...

### Module
Chaque task utilise un **module** Ansible (abstraction d'une opération) :
- `apt` / `yum` → gestion de paquets
- `copy` / `template` → copie de fichiers
- `service` → gestion de services
- `user` → gestion d'utilisateurs
- `command` / `shell` → exécution de commandes brutes

### Role
Unité réutilisable qui regroupe tasks, variables, templates et handlers liés à un même composant (ex: rôle `nginx`, rôle `postgresql`).

---

## 3. Structure d'un projet Ansible

```
projet-ansible/
├── inventory.ini          # liste des serveurs
├── playbook.yml           # playbook principal
├── group_vars/
│   └── webservers.yml     # variables pour le groupe webservers
├── host_vars/
│   └── web1.yml           # variables spécifiques à web1
└── roles/
    └── nginx/
        ├── tasks/
        │   └── main.yml
        ├── templates/
        │   └── nginx.conf.j2
        └── handlers/
            └── main.yml
```

---

## 4. Playbook — exemple complet

```yaml
# playbook.yml
---
- name: Configurer les serveurs web
  hosts: webservers
  become: true          # équivalent sudo

  vars:
    app_port: 8080

  tasks:
    - name: Mettre à jour les paquets
      apt:
        update_cache: true

    - name: Installer Nginx
      apt:
        name: nginx
        state: present   # present = installé, absent = désinstallé

    - name: Copier la configuration Nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: Redémarrer Nginx   # déclenche le handler si changement

    - name: Activer et démarrer Nginx
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Redémarrer Nginx
      service:
        name: nginx
        state: restarted
```

> 💡 Un **handler** ne s'exécute qu'une fois, à la fin du play, et seulement si une task l'a notifié.

---

## 5. Variables et templates Jinja2

### Variables
```yaml
# group_vars/webservers.yml
nginx_port: 80
server_name: monapp.exemple.com
```

Utilisation dans un playbook :
```yaml
- name: Afficher le port
  debug:
    msg: "Port configuré : {{ nginx_port }}"
```

### Templates Jinja2 (`.j2`)
Les templates permettent de générer des fichiers de config dynamiques :

```nginx
# templates/nginx.conf.j2
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};

    location / {
        proxy_pass http://localhost:{{ app_port }};
    }
}
```

---

## 6. Commandes essentielles

| Commande | Rôle |
|---|---|
| `ansible all -i inventory.ini -m ping` | Teste la connexion à tous les hôtes |
| `ansible-playbook -i inventory.ini playbook.yml` | Exécute un playbook |
| `ansible-playbook ... --check` | Mode dry-run (simule sans appliquer) |
| `ansible-playbook ... --diff` | Affiche les différences de fichiers |
| `ansible-playbook ... --tags "nginx"` | Exécute seulement les tasks taguées |
| `ansible-playbook ... --limit web1` | Limite l'exécution à un hôte |
| `ansible-vault encrypt secrets.yml` | Chiffre un fichier de secrets |
| `ansible-vault decrypt secrets.yml` | Déchiffre un fichier de secrets |
| `ansible-galaxy role install nom.role` | Installe un rôle depuis Ansible Galaxy |

---

## 7. Ansible Vault — gérer les secrets

Ansible Vault chiffre les fichiers contenant des données sensibles (mots de passe, clés API).

```bash
# Créer un fichier chiffré
ansible-vault create secrets.yml

# Exécuter un playbook avec un fichier chiffré
ansible-playbook playbook.yml --ask-vault-pass
```

---

## 8. Bonnes pratiques

- Toujours tester avec `--check` avant un premier apply en production.
- Utiliser **Ansible Vault** pour tous les secrets — ne jamais les mettre en clair dans Git.
- Préférer les **modules Ansible** aux modules `command`/`shell` (plus idempotents).
- Organiser le code en **rôles** dès que le playbook dépasse 50 lignes.
- Nommer les tasks clairement (phrase d'action : "Installer Nginx", "Copier la config").
- Versionner l'inventaire et les playbooks dans Git.

---

## 9. ✅ À retenir

- Ansible configure des serveurs via SSH, **sans agent** sur les cibles.
- Un **playbook** = liste de tasks YAML à exécuter sur des hôtes.
- Chaque task utilise un **module** (apt, service, template, copy...).
- Les playbooks sont **idempotents** : on peut les relancer sans risque.
- **Terraform provisionne**, **Ansible configure** — les deux sont complémentaires.
- Secrets → **Ansible Vault**.
