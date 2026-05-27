---
title: "cloud-init — Configuration au démarrage des VMs"
tags:
  - cloud-init
  - cloud
  - vm
  - automatisation
  - infrastructure
section: 04-Environnements
domaine: Cloud
statut: actif
liens_connexes:
  - [[cloud_aws_fondamentaux]]
  - [[terraform_fondamentaux]]
  - [[ansible_fondamentaux]]
  - [[ssh_avance]]
  - [[utilisateurs_permissions]]
---

# 📘 Fiche : cloud-init

---

## 1. Qu'est-ce que cloud-init ?

**cloud-init** est le standard de configuration au **premier démarrage** d'une VM ou instance cloud. Il est intégré dans la quasi-totalité des images cloud (AWS AMI, Azure, GCP, images Debian/Ubuntu cloud...).

### Cas d'usage
- Créer des utilisateurs et injecter des clés SSH
- Installer des paquets au boot
- Écrire des fichiers de configuration
- Exécuter des scripts d'initialisation
- Configurer le hostname, le réseau

### Pourquoi pas juste un script bash ?

| Script bash seul | cloud-init |
|---|---|
| Exécuté à chaque démarrage si mal configuré | Exécuté **une seule fois** (premier boot) |
| Pas de modules standard | Modules declaratifs (users, packages, files...) |
| Pas de logging standardisé | Logs dans `/var/log/cloud-init.log` |
| Spécifique à chaque OS | Portable sur tous les cloud providers |

---

## 2. Comment ça fonctionne

```
Instance démarre
      ↓
cloud-init lit le "user-data" (fourni par le provider)
      ↓
Exécute les modules dans l'ordre : network → users → packages → files → runcmd
      ↓
Instance prête, cloud-init ne se réexécute plus
```

Le **user-data** est le fichier de configuration passé à cloud-init. Sur AWS, il se passe dans le champ "User data" au lancement d'une instance EC2. Avec Terraform, via `user_data`.

---

## 3. Format du fichier user-data

Le fichier commence obligatoirement par `#cloud-config` :

```yaml
#cloud-config

# Le reste est du YAML
```

---

## 4. Modules essentiels

### Hostname
```yaml
#cloud-config
hostname: mon-serveur
fqdn: mon-serveur.exemple.com
```

### Utilisateurs et clés SSH
```yaml
#cloud-config
users:
  - name: deploy
    groups: sudo
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3... cle-publique-ici

  - name: alice
    groups: [sudo, docker]
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3... cle-alice
```

### Installation de paquets
```yaml
#cloud-config
package_update: true
package_upgrade: true

packages:
  - nginx
  - git
  - curl
  - ufw
  - fail2ban
```

### Écriture de fichiers
```yaml
#cloud-config
write_files:
  - path: /etc/nginx/sites-available/default
    content: |
      server {
          listen 80;
          server_name _;
          root /var/www/html;
      }
    owner: root:root
    permissions: '0644'

  - path: /home/deploy/.ssh/config
    content: |
      Host *
        ServerAliveInterval 60
    owner: deploy:deploy
    permissions: '0600'
```

### Exécuter des commandes (runcmd)
```yaml
#cloud-config
runcmd:
  - systemctl enable nginx
  - systemctl start nginx
  - ufw allow 80/tcp
  - ufw allow 443/tcp
  - ufw allow ssh
  - ufw --force enable
```

> ⚠️ `runcmd` s'exécute après les autres modules. Les commandes sont exécutées dans l'ordre, dans un shell.

---

## 5. Exemple complet — Serveur web prêt à l'emploi

```yaml
#cloud-config

hostname: web-01

users:
  - name: deploy
    groups: [sudo, www-data]
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5... ma-cle-publique

package_update: true
package_upgrade: true

packages:
  - nginx
  - git
  - curl
  - ufw
  - fail2ban
  - unattended-upgrades

write_files:
  - path: /var/www/html/index.html
    content: |
      <h1>Serveur opérationnel</h1>
    permissions: '0644'

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
  - systemctl enable fail2ban
  - systemctl start fail2ban
  - ufw allow ssh
  - ufw allow 80/tcp
  - ufw allow 443/tcp
  - ufw --force enable
  - dpkg-reconfigure -f noninteractive unattended-upgrades
```

---

## 6. Utilisation avec Terraform

```hcl
# main.tf
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  key_name      = aws_key_pair.deployer.key_name

  user_data = file("cloud-init.yaml")

  tags = {
    Name = "web-01"
  }
}
```

Ou en passant le contenu inline avec `templatefile` pour injecter des variables :

```hcl
user_data = templatefile("cloud-init.yaml.tpl", {
  ssh_key  = var.ssh_public_key
  hostname = "web-${var.env}"
})
```

---

## 7. Debugging

```bash
# Statut d'exécution de cloud-init
cloud-init status

# Logs détaillés
cat /var/log/cloud-init.log
cat /var/log/cloud-init-output.log

# Vérifier ce qui a été exécuté
cloud-init query userdata

# Forcer une réexécution (dev/test uniquement)
cloud-init clean
cloud-init init
```

---

## 8. Bonnes pratiques

- Toujours tester le fichier cloud-config avec `cloud-init devel schema --config-file mon-fichier.yaml` avant de déployer.
- Utiliser `runcmd` en dernier recours — préférer les modules natifs (`packages`, `write_files`...) qui sont idempotents.
- Ne pas mettre de secrets en clair dans user-data — il est lisible depuis l'instance (`curl http://169.254.169.254/latest/user-data` sur AWS).
- Pour des configurations complexes, préférer **Ansible** après le boot plutôt que tout faire dans cloud-init.
- Versionner les fichiers cloud-init dans Git avec le reste de l'infra.

---

## 9. ✅ À retenir

- **cloud-init** s'exécute **une seule fois** au premier démarrage d'une VM.
- Standard universel : fonctionne sur AWS, Azure, GCP, VPS (si image cloud).
- Fichier commence par `#cloud-config`, format YAML.
- Modules clés : `users`, `packages`, `write_files`, `runcmd`.
- Avec Terraform : passer via `user_data = file("cloud-init.yaml")`.
- Logs : `/var/log/cloud-init.log` et `/var/log/cloud-init-output.log`.
