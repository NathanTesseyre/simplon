# Guide Agent Jenkins + VirtualBox + Terraform + Ansible (multi-OS, clair et vérifié)

Ce guide permet :
- d'utiliser **une machine locale (Windows / Linux / macOS)** comme **agent Jenkins**
- de **créer une VM VirtualBox avec Terraform**
- puis **d'exécuter un playbook Ansible** (dans Docker) depuis Jenkins.

---

## 1) Vérification avant installation

Avant d’installer, vérifiez si les outils ne sont pas déjà présents :

```
VBoxManage --version   # VirtualBox ?
terraform version      # Terraform ?
java -version          # Java ?
docker --version       # Docker ?
```

Si une commande fonctionne → **ne pas réinstaller** l’outil.

---

## 2) Installation par OS (outil par outil, clairement séparé)

### A) **Windows 10/11**

#### VirtualBox
https://www.virtualbox.org/wiki/Downloads  
→ Installer VirtualBox + Extension Pack

#### Terraform
1. Télécharger : https://developer.hashicorp.com/terraform/downloads  
2. Décompresser `terraform.exe` vers :
   ```
   C:\Program Files\terraform\
   ```
3. Ajouter au PATH :
   ```
   C:\Program Files\terraform\
   ```

#### Java (nécessaire pour l’agent Jenkins)
Télécharger Temurin 11 LTS :
https://adoptium.net/

#### Docker (pour Ansible)
Installer Docker Desktop :
https://www.docker.com/products/docker-desktop/  
→ Activer **WSL2 backend**

---

### B) **Linux (Ubuntu / Debian)**

#### VirtualBox
```bash
sudo apt update
sudo apt install virtualbox virtualbox-ext-pack -y
```

#### Terraform
```bash
sudo apt-get install wget gnupg software-properties-common -y
wget -O- https://apt.releases.hashicorp.com/gpg | sudo tee /usr/share/keyrings/hashicorp.asc
echo "deb [signed-by=/usr/share/keyrings/hashicorp.asc] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform -y
```

#### Java
```bash
sudo apt install default-jre -y
```

#### Docker (pour Ansible)
```bash
sudo apt install docker.io -y
sudo usermod -aG docker $USER
```
> Reconnectez-vous pour appliquer les groupes.

---

### C) **macOS**

#### VirtualBox
```bash
brew install --cask virtualbox
brew install --cask virtualbox-extension-pack
```

#### Terraform
```bash
brew install terraform
```

#### Java
```bash
brew install temurin
```

#### Docker
```bash
brew install --cask docker
```

---

## 3) Création de l’agent Jenkins

Dans Jenkins UI :

```
Manage Jenkins → Manage Nodes and Clouds → New Node
```

| Champ | Valeur |
|------|--------|
| Name | vb-host |
| Type | Permanent Agent |
| Remote root directory | C:\jenkins-agent (Win) / ~/jenkins-agent (Linux/Mac) |
| Labels | vb-host |
| Launch Method | **This agent will connect to the controller** |

### Où trouver le **secret de l'agent** ?
- Aller dans : `Manage Jenkins → Manage Nodes → vb-host`
- Cliquer sur **Launch agent**
- Le secret est affiché
- Ou clic : **"Click to see the agent secret"**

---

## 4) Démarrer l’agent sur la machine hôte

```powershell
mkdir C:\jenkins-agent
cd C:\jenkins-agent
Invoke-WebRequest http://<JENKINS_IP>:8080/jnlpJars/agent.jar -OutFile agent.jar

java -jar agent.jar -jnlpUrl http://<JENKINS_IP>:8080/computer/vb-host/jenkins-agent.jnlp -secret <SECRET>
```

---

## 5) Exemple Terraform minimal (VM VirtualBox)

`infra/main.tf` :
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

resource "virtualbox_vm" "demo" {
  name   = "demo-vm"
  image  = "/path/to/ubuntu-cloud.vdi"
  cpus   = 2
  memory = 2048
}

output "vm_ip" {
  value = "192.168.56.10"
}
```

---

## 6) Exemple Jenkinsfile clair (Terraform + inventaire + Ansible Docker)

```groovy
pipeline {
  agent { label 'vb-host' }

  environment {
    TF_IN_AUTOMATION = "true"
    ANSIBLE_HOST_KEY_CHECKING = "False"
  }

  stages {

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Terraform Apply') {
      steps {
        sh '''
          cd infra
          terraform init -input=false
          terraform apply -auto-approve
          terraform output -raw vm_ip > ../vm_ip.txt
        '''
      }
    }

    stage('Generate Inventory') {
      steps {
        sh '''
          echo "[vm]" > inventory.ini
          echo "$(cat vm_ip.txt) ansible_user=ubuntu ansible_ssh_private_key_file=./infra/id_rsa" >> inventory.ini
        '''
      }
    }

    stage('Configure VM (Ansible via Docker)') {
      steps {
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

## ✅ Résultat

Ce pipeline :
- Crée une VM VirtualBox avec Terraform
- Récupère automatiquement son IP
- Génère un inventaire Ansible
- Exécute le playbook via Docker (sans installer Ansible)

Fin. 🎉
