# 📘 Fiche : Outils développeur sous Linux

**Sommaire**
- [Compilation & debug](#dev-build)
- [Git](#dev-git)
- [HTTP & transferts](#dev-http)
- [Éditeurs](#dev-edit)
- [Multiplexeurs](#dev-mux)
- [Environnements de langage](#dev-lang)
- [SSH & synchronisation](#dev-ssh)
- [Cas pratiques](#dev-cas)
- [✅ À retenir](#dev-ret)

<a id="dev-build"></a>
## Compilation & debug
```bash
gcc main.c -o app
make
gdb app
```

<a id="dev-git"></a>
## Git
```bash
git init && git add . && git commit -m "init"
git branch -M main && git remote add origin <url>
git push -u origin main
```

<a id="dev-http"></a>
## HTTP & transferts
```bash
curl -v https://example.com
wget https://example.com/arch.tar.gz
```

<a id="dev-edit"></a>
## Éditeurs
vim, nano, emacs

<a id="dev-mux"></a>
## Multiplexeurs
tmux (`tmux`, `Ctrl-b d`), screen

<a id="dev-lang"></a>
## Environnements de langage
- Python venv :
```bash
python -m venv .venv && source .venv/bin/activate
```
- Node.js (nvm) :
```bash
nvm install --lts && nvm use --lts
```

<a id="dev-ssh"></a>
## SSH & synchronisation
```bash
ssh user@serveur -p 22
scp fichier user@serveur:/tmp/
rsync -avh /src/ user@serveur:/dst/
```

<a id="dev-cas"></a>
## Cas pratiques
1. Compiler et déboguer un mini-prog C.  
2. Créer un dépôt Git et pousser.  
3. Ouvrir une session tmux, se détacher puis revenir.

<a id="dev-ret"></a>
## ✅ À retenir
Linux = outillage complet : build, VCS, HTTP, éditeurs, multiplexeurs.
