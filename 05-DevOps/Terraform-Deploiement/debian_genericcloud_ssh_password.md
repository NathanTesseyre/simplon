# Debian *genericcloud* & SSH par mot de passe (Terraform + VirtualBox)

## 1) C’est quoi une image Debian *genericcloud* ?
Une image **Debian “genericcloud”** est un disque prêt à l’emploi pour machines virtuelles/cloud :
- minimal (sans GUI), drivers virtuels déjà présents ;
- **intègre `cloud-init`** pour configurer à la 1ère boot : utilisateur, mot de passe/clé SSH, hostname, réseau, etc. ;
- format courant : `qcow2` (convertissable en `vdi`/`vmdk` pour VirtualBox).

> Avantage : vous **décrivez** l’utilisateur, le réseau et l’accès SSH via un simple fichier *cloud-init* — pas besoin d’ouvrir la VM manuellement.

---

## 2) Je **ne veux pas** de clé SSH : activer l’accès SSH **par mot de passe**
C’est possible et simple avec `cloud-init`. On crée un utilisateur avec un **mot de passe hashé** et on autorise l’authentification par mot de passe côté SSH.

### 2.1 Générer un mot de passe **haché SHA‑512**
Choisissez l’une des méthodes ci-dessous (toutes donnent un hash compatible `/etc/shadow`).

**Linux/macOS (OpenSSL) :**
```bash
openssl passwd -6 'VotreMotDePasseFort'
```
**Linux (mkpasswd du paquet whois) :**
```bash
mkpasswd -m sha-512
```
**Windows (via Docker) :**
```powershell
docker run --rm alpine sh -lc "apk add --no-cache openssl >/dev/null && openssl passwd -6 'VotreMotDePasseFort'"
```
> Gardez **le hash** (il commence par `$6$...`). Ne mettez jamais le mot de passe en clair dans le dépôt.

### 2.2 `cloud-init` pour activer SSH par mot de passe
Créez `infra/cloud-init.yaml` :

```yaml
#cloud-config
ssh_pwauth: true   # autorise l'authentification par mot de passe
users:
  - name: admin
    gecos: CI Admin
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    groups: [adm, sudo]
    shell: /bin/bash
    lock_passwd: false
    passwd: $6$REMPLACEZ_PAR_VOTRE_HASH$xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

package_update: true
packages: [openssh-server, curl]

# (Optionnel) force le paramètre côté sshd si l'image le désactive par défaut
write_files:
  - path: /etc/ssh/sshd_config.d/10-passwordauth.conf
    permissions: "0644"
    content: |
      PasswordAuthentication yes

runcmd:
  - systemctl restart ssh || systemctl restart sshd
```

---

## 3) Terraform (VirtualBox) avec **réseau Bridge** + IP en *output*
On part d’une image Debian genericcloud convertie en **VDI** (recommandé).

**Convertir QCOW2 → VDI :**
```bash
qemu-img convert -f qcow2 -O vdi debian-12-genericcloud-amd64.qcow2 debian-12.vdi
```

**`infra/main.tf` :**
```hcl
terraform {
  required_providers {
    virtualbox = {
      source  = "terra-farm/virtualbox"
      version = "~> 0.3"
    }
  }
}

provider "virtualbox" {}

variable "image_path" { default = "C:/images/debian-12.vdi" }  # ou /opt/images/...
variable "vm_name"    { default = "debian-bridge" }
variable "vm_cpu"     { default = 2 }
variable "vm_mem"     { default = 2048 }
variable "ssh_user"   { default = "admin" }
variable "bridge_if"  { default = "Intel(R) Ethernet..." }     # nom exact de l’interface hôte

variable "ssh_pubkey_path" { default = "" } # non utilisé ici

data "template_file" "user_data" {
  template = file("${path.module}/cloud-init.yaml")
}

resource "virtualbox_vm" "vm" {
  name   = var.vm_name
  image  = var.image_path
  cpus   = var.vm_cpu
  memory = var.vm_mem

  # NIC0 en BRIDGE → obtiendra une IP par le DHCP de votre LAN
  network_adapter {
    type           = "bridged"
    host_interface = var.bridge_if
  }

  # Monte l'ISO seed de cloud-init (créé par le provider) avec vos paramètres ci-dessus
  user_data = data.template_file.user_data.rendered

  wait_for_guest_net_timeout = 300

  # Infos SSH (optionnel, utile si vous utilisez ensuite des provisioners)
  ssh_username     = var.ssh_user
  ssh_wait_timeout = "5m"
}

output "vm_ip" {
  description = "Adresse IPv4 Bridge (DHCP)"
  value       = virtualbox_vm.vm.network_adapter[0].ipv4_address
}

output "ssh_user" {
  value = var.ssh_user
}
```

> ⚠️ **`bridge_if`** doit correspondre **exactement** au nom de l’interface réseau de l’hôte (ex. *“Intel(R) Ethernet I219‑V”*, *“en0: Wi‑Fi (AirPort)”*, *“eth0”*…).

---

## 4) Connexion SSH après `terraform apply`
Une fois l’apply terminé :
```bash
terraform output -raw vm_ip
# → ex: 192.168.1.57

ssh admin@192.168.1.57
# Mot de passe : celui dont vous avez calculé le hash
```

---

## 5) (Option) Lancer un playbook Ansible avec mot de passe
**Inventaire temporaire :**
```bash
cat > inventory.ini <<EOF
[debian]
$(terraform output -raw vm_ip) ansible_user=$(terraform output -raw ssh_user) ansible_password=VOTRE_MDP ansible_become_password=VOTRE_MDP
EOF
```
**Via Docker (sans Ansible installé localement) :**
```bash
docker run --rm -v "$PWD":/ws -w /ws cytopia/ansible:latest   ansible-playbook -i inventory.ini ansible/site.yml
```

> En production, **ne stockez pas** le mot de passe en clair : passez-le via les **Credentials Jenkins** ou un coffre (Vault).

---

## 6) Pourquoi pas un OVA ?
- Un **OVA** est une *VM complète importée* (CPU/RAM/NIC pré‑définis). Terraform gère mieux une image **disque** (`VDI`/`VMDK`) qu’il **attache** en laissant à Terraform le contrôle complet (réseau bridge, cloud‑init, etc.).
- Ça réduit les surprises et garde votre infra **déclarative** et **reproductible**.

Bon déploiement ! 🙌
