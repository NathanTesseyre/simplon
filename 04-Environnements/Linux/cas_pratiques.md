# 📘 Fiche : Cas pratiques Linux

**Sommaire**
- [Exercice 1 : Utilisateurs & sudo](#cp-u)
- [Exercice 2 : Service web](#cp-web)
- [Exercice 3 : Sécuriser SSH](#cp-ssh)
- [Exercice 4 : Logs](#cp-logs)
- [Exercice 5 : Script + cron](#cp-cron)
- [✅ À retenir](#cp-ret)

<a id="cp-u"></a>
## Exercice 1 : Utilisateurs & sudo
```bash
sudo useradd -m stagiaire && sudo passwd stagiaire
sudo usermod -aG sudo stagiaire
su - stagiaire -c 'id && sudo -v'
```

<a id="cp-web"></a>
## Exercice 2 : Service web
```bash
sudo apt update && sudo apt install -y nginx   # Debian/Ubuntu
sudo dnf install -y nginx                      # Fedora/RedHat
sudo systemctl enable --now nginx
sudo systemctl status nginx
ss -ltnp | grep :80
```

<a id="cp-ssh"></a>
## Exercice 3 : Sécuriser SSH
Modifier `/etc/ssh/sshd_config` (port, `PermitRootLogin no`) puis :
```bash
sudo systemctl restart sshd || sudo systemctl restart ssh
ss -ltnp | grep ssh
```

<a id="cp-logs"></a>
## Exercice 4 : Logs
```bash
journalctl -u ssh -f
journalctl -u ssh | grep "Failed password" | tail
```

<a id="cp-cron"></a>
## Exercice 5 : Script + cron
```bash
cat > ~/backup_home.sh <<'EOF'
#!/usr/bin/env bash
set -Eeuo pipefail
tar -czf "/tmp/home_$(date +%F).tgz" "$HOME"
EOF
chmod +x ~/backup_home.sh
# Tous les jours à 2h
( crontab -l; echo "0 2 * * * /home/$USER/backup_home.sh" ) | crontab -
```

<a id="cp-ret"></a>
## ✅ À retenir
Cas concrets : **utilisateurs, services, sécurité, logs, automation**.
