# Partager une image Debian pour VirtualBox sous 500 Mo (Windows/Linux/macOS)

Vous n’avez **pas besoin** de partager un `.vdi` lourd. Partagez plutôt une **image compacte** et laissez chaque machine **générer le disque final localement**.

## ✅ Recommandation
- Distribuer **l’image source** officielle **Debian *genericcloud* en QCOW2** (≈ 300–450 Mo) **ou** une **VMDK *streamOptimized*** (très compacte).  https://cloud.debian.org/images/cloud/bookworm/latest/ 
- Fournir un **script de conversion local** (Windows/WSL/PowerShell, Linux, macOS) qui produit un **VDI** ou **VMDK** utilisable par Terraform/VirtualBox.

> Le provider VirtualBox accepte **VDI** *et* **VMDK**. Le format **VMDK streamOptimized** est idéal pour partager (< 500 Mo) et VirtualBox peut l’utiliser directement.

---

## Partager le **QCOW2 officiel** et convertir localement
Le QCOW2 de Debian est compact. Fournissez un script **cross‑platform** pour convertir en **VDI** *ou* **VMDK** sur la machine cible.

### 1) Script **Bash** (Linux & macOS & WSL)
Enregistrez sous `tools/convert-debian.sh` :
```bash
#!/usr/bin/env bash
set -euo pipefail

SRC_QCOW2="${1:-debian-12-genericcloud-amd64.qcow2}"
OUT_FMT="${2:-vdi}"             # vdi | vmdk
OUT_NAME="${3:-debian-12.${OUT_FMT}}"

command -v qemu-img >/dev/null 2>&1 || { echo "qemu-img manquant. Installez-le : apt install qemu-utils | brew install qemu"; exit 1; }

case "$OUT_FMT" in
  vdi)  qemu-img convert -f qcow2 -O vdi  "$SRC_QCOW2" "$OUT_NAME" ;;
  vmdk) qemu-img convert -f qcow2 -O vmdk -o subformat=streamOptimized "$SRC_QCOW2" "$OUT_NAME" ;;
  *)    echo "Format inconnu: $OUT_FMT (attendu: vdi|vmdk)"; exit 1 ;;
esac

echo "OK -> $OUT_NAME"
```
Rendre exécutable :
```bash
chmod +x tools/convert-debian.sh
```

### 2) Script **PowerShell** (Windows natif ou PowerShell Core)
Enregistrez sous `tools/convert-debian.ps1` :
```powershell
param(
  [string]$SrcQcow2 = "debian-12-genericcloud-amd64.qcow2",
  [ValidateSet("vdi","vmdk")]
  [string]$OutFmt = "vdi",
  [string]$OutName
)

if (-not $OutName) { $OutName = "debian-12.$OutFmt" }

function Have($cmd){ $null -ne (Get-Command $cmd -ErrorAction SilentlyContinue) }

# 1) Essayer qemu-img natif (Chocolatey)
if (-not (Have "qemu-img")) {
  Write-Host "qemu-img introuvable. Vous pouvez l'installer via Chocolatey : 'choco install qemu'."
  # 2) Sinon, tenter via WSL
  if (Have "wsl") {
    Write-Host "Tentative via WSL…"
    $fmtArg = ($OutFmt -eq "vmdk") ? "-O vmdk -o subformat=streamOptimized" : "-O vdi"
    wsl bash -lc "qemu-img --version || (sudo apt update && sudo apt install -y qemu-utils); qemu-img convert -f qcow2 $fmtArg `"$SrcQcow2`" `"$OutName`""
    if ($LASTEXITCODE -eq 0) { Write-Host "OK -> $OutName"; exit 0 }
  }
  Write-Error "qemu-img non disponible. Installez-le (Chocolatey) ou activez WSL."
  exit 1
} else {
  if ($OutFmt -eq "vmdk") {
    & qemu-img convert -f qcow2 -O vmdk -o subformat=streamOptimized $SrcQcow2 $OutName
  } else {
    & qemu-img convert -f qcow2 -O vdi $SrcQcow2 $OutName
  }
  if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }
  Write-Host "OK -> $OutName"
}
```

**Usage :**
```powershell
# Windows (PowerShell)
powershell -ExecutionPolicy Bypass -File .\tools\convert-debian.ps1 -SrcQcow2 .\debian-12-genericcloud-amd64.qcow2 -OutFmt vmdk -OutName debian-12-stream.vmdk
# ou
bash ./tools/convert-debian.sh ./debian-12-genericcloud-amd64.qcow2 vdi debian-12.vdi   # via WSL/git-bash
```

---

## Intégration Terraform (VirtualBox)

Dans `infra/main.tf`, vous pouvez pointer :
```hcl
# Soit VDI
image = "${path.module}/images/debian-12.vdi"

# Soit VMDK streamOptimized (léger à partager)
image = "${path.module}/images/debian-12-stream.vmdk"
```

Le reste (network bridge, cloud-init, SSH) ne change pas.

---

## FAQ

**Q. On n’a pas réussi à convertir QCOW2→VDI sur Windows.**  
R. Utilisez **le script PowerShell** ci-dessus. Il tentera **qemu-img** natif ; à défaut, il passe **via WSL** automatiquement. Sinon, convertissez **une fois** sur une machine Linux/macOS et partagez le **VMDK streamOptimized** (≤ 500 Mo).

**Q. Je dois lancer le script depuis WSL ou depuis PowerShell ?**  
R. **Comme vous voulez.** Il existe un script **Bash** (WSL/Linux/macOS) et un script **PowerShell** (Windows). Les deux produisent les mêmes sorties.

**Q. Est-ce que le script marche sur Linux et macOS ?**  
R. Oui : `tools/convert-debian.sh` fonctionne sur **Linux et macOS**. Installez `qemu-img` (`apt install qemu-utils` ou `brew install qemu`).

**Q. Et si je veux quand même un `.vdi` < 500 Mo ?**  
R. Compressez pour le partage (ex. `.vdi.xz` / `.7z`). À la réception, **décompressez** et utilisez le fichier décompressé comme `image`. Mais **le plus simple** reste de partager **VMDK streamOptimized** directement utilisable par VirtualBox.

---

## Exemple récapitulatif

1. Téléchargez le QCOW2 Debian (≈ 400 Mo).  
2. Convertissez **localement** en :  
   - **VMDK streamOptimized** `debian-12-stream.vmdk` (à partager), **ou**
   - **VDI** `debian-12.vdi` (à garder local).
3. Dans Terraform, mettez `image = "…/debian-12-stream.vmdk"` (ou `.vdi`).

Bonne route ! 🚀
