# OVA vs VDI pour Terraform + VirtualBox (Debian, réseau bridge, SSH)

## TL;DR
- **Choix recommandé : `VDI`** (ou `VMDK`) comme **disque de base**.
- **Éviter `OVA`** pour Terraform : un OVA contient une *VM complète* (réseau, CPU, etc.). Le provider VirtualBox de Terraform attend surtout **un disque** à attacher et **contrôle lui-même** le reste (CPU, RAM, cartes réseau, cloud-init…). Les OVA importées ajoutent de la complexité et peuvent contredire la config Terraform.

---

## Pourquoi VDI est mieux ici
1. **Compatibilité provider** : `virtualbox_vm.image` pointe sur un **fichier disque** (VDI/VMDK). Pas besoin d’« importer » une VM toute faite.
2. **Idempotence** : Terraform définit la VM (CPU, mémoire, adaptateurs réseau, cloud-init). Un OVA risque d’introduire des écarts (noms d’interface, contrôleurs, etc.).
3. **Cloud‑init** : les images *cloud* Debian fonctionnent très bien en VDI/VMDK, et le provider peut leur **monter un ISO cloud‑init** via `user_data` pour créer l’utilisateur, injecter la clé SSH, configurer le réseau, etc.

---

## Obtenir une image Debian *cloud* en VDI
Les images Debian *genericcloud* sont souvent en **QCOW2**. Convertissez‑la en **VDI** avec `qemu-img` :

```bash
qemu-img convert -f qcow2 -O vdi debian-12-genericcloud-amd64.qcow2 debian-12.vdi
```

Placez ensuite `debian-12.vdi` sur la machine agent (celle qui a VirtualBox).

> Alternative : `VMDK` fonctionne aussi : `-O vmdk`.

---

## Exemple Terraform : Debian + Bridge + SSH + IP en output

**Arborescence rapide**
```
infra/
├─ main.tf
├─ variables.tf
├─ outputs.tf
├─ cloud-init.yaml
└─ keys/
   ├─ id_ed25519
   └─ id_ed25519.pub
```

**`infra/variables.tf`**
```hcl
variable "image_path"   { default = "C:/images/debian-12.vdi" } # ou /opt/images/...
variable "vm_name"      { default = "debian-bridge" }
variable "vm_cpu"       { default = 2 }
variable "vm_memory"    { default = 2048 }
variable "ssh_user"     { default = "debian" } # utilisateur cloud (ou 'admin')
variable "bridge_if"    { default = "en0: Wi-Fi (AirPort)" }   # macOS exemple
# Windows: "Intel(R) Ethernet Controller I225-V"
# Linux : "eth0" / "wlan0" / "enp3s0" ... (voir `ip link`)

variable "ssh_pubkey_path" {
  default = "${path.module}/keys/id_ed25519.pub"
}
```

**`infra/cloud-init.yaml`**
```yaml
#cloud-config
users:
  - name: ${ssh_user}
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    groups: [adm, sudo]
    shell: /bin/bash
    ssh_authorized_keys:
      - ${ssh_authorized_key}

package_update: true
packages:
  - openssh-server
  - curl

# Le bridge prend une IP par DHCP (par défaut), rien à forcer ici.
# Si vous voulez une IP fixe, vous pouvez écrire un fichier netplan.
```

**`infra/main.tf`**
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

data "template_file" "user_data" {
  template = file("${path.module}/cloud-init.yaml")
  vars = {
    ssh_user           = var.ssh_user
    ssh_authorized_key = chomp(file(var.ssh_pubkey_path))
  }
}

resource "virtualbox_vm" "debian" {
  name   = var.vm_name
  image  = var.image_path
  cpus   = var.vm_cpu
  memory = var.vm_memory

  # NIC0 : BRIDGED → l’IP est délivrée par votre DHCP du LAN
  network_adapter {
    type           = "bridged"
    host_interface = var.bridge_if
  }

  # (Optionnel) NIC1 : NAT pour une sortie internet séparée
  # network_adapter { type = "nat" }

  # cloud-init (ISO de seed montée automatiquement par le provider)
  user_data = data.template_file.user_data.rendered

  # Attendre que l’OS soit up et connecté au réseau
  wait_for_guest_net_timeout = 300

  # Infos SSH (facultatif mais utile pour `terraform apply` qui attend la VM up)
  ssh_username     = var.ssh_user
  ssh_private_key  = file("${path.module}/keys/id_ed25519")
  ssh_wait_timeout = "5m"
}
```

**`infra/outputs.tf`**
```hcl
# Selon le provider, l’IP bridgée découverte est exposée sur l’adaptateur 0
# (après wait_for_guest_net_timeout).

output "vm_ip" {
  description = "Adresse IPv4 (bridge) de la VM"
  value       = virtualbox_vm.debian.network_adapter[0].ipv4_address
}

output "ssh_user" {
  value = var.ssh_user
}
```

> Remarques :
> - Sur **Windows/macOS**, la valeur de `bridge_if` doit **exactement** correspondre au nom de l’interface réseau hôte que vous voulez pont-er (ex. *"Intel(R) Ethernet..."* ou *"en0: Wi-Fi (AirPort)"*).  
> - L’**IP en bridge** dépend de votre **DHCP**. L’output `network_adapter[0].ipv4_address` n’est renseigné qu’une fois la VM up ET l’adresse attribuée.  
> - Debian *genericcloud* embarque **cloud-init** et **OpenSSH**. Le `packages: [openssh-server]` est une ceinture/suspenders.

---

## Inventaire Ansible basé sur l’output

Après `terraform apply` :
```bash
terraform output -raw vm_ip > vm_ip.txt
terraform output -raw ssh_user > ssh_user.txt
```

**Inventory minimal :**
```bash
echo "[debian]" > inventory.ini
echo "$(cat vm_ip.txt) ansible_user=$(cat ssh_user.txt) ansible_ssh_private_key_file=./infra/keys/id_ed25519" >> inventory.ini
```

---

## Jenkinsfile ultra‑concis (apply + playbook)

```groovy
pipeline {
  agent { label 'vb-host' }
  environment {
    TF_IN_AUTOMATION = "true"
    ANSIBLE_HOST_KEY_CHECKING = "False"
  }
  stages {
    stage('Checkout'){ steps{ checkout scm }}

    stage('Terraform apply'){
      steps {
        sh '''
          cd infra
          terraform init -input=false
          terraform apply -auto-approve
          terraform output -raw vm_ip > ../vm_ip.txt
          terraform output -raw ssh_user > ../ssh_user.txt
        '''
      }
    }

    stage('Ansible (Docker)'){
      steps {
        sh '''
          echo "[debian]" > inventory.ini
          echo "$(cat vm_ip.txt) ansible_user=$(cat ssh_user.txt) ansible_ssh_private_key_file=./infra/keys/id_ed25519" >> inventory.ini
        '''
        script {
          docker.image('cytopia/ansible:latest').inside('-v $WORKSPACE:/ws -w /ws') {
            sh "ansible-playbook ansible/site.yml -i inventory.ini"
          }
        }
      }
    }
  }
}
```

---

## Conclusion
- Pour Terraform + VirtualBox, **préférez un `VDI` (ou `VMDK`)** issu d’une image *cloud* Debian.  
- Utilisez le **réseau en bridge** sur l’adaptateur 0 pour obtenir une **IP DHCP** que vous exposez en **output**.  
- Activez **SSH** via cloud‑init et connectez‑vous avec Ansible depuis Jenkins.

Bon déploiement !
