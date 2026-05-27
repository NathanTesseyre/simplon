---
title: "Trunk-based development"
tags:
  - devops
  - git
  - branching
  - CI/CD
section: 05-DevOps
domaine: CI/CD
statut: actif
liens_connexes:
  - [[fiche_git]]
  - [[semantic-release-and-versionning]]
---

# 🚀 Trunk-Based Development (TBD)

Le **Trunk-Based Development** est un modèle de gestion de branches Git basé sur une **intégration rapide et continue** dans une branche principale unique (`main`).  
Plutôt que de conserver des branches longues et isolées, on favorise des **modifications petites, fréquentes et rapidement fusionnées**.

Ce modèle est utilisé par des organisations à fort rythme de déploiement comme **Google, Meta, Netflix, Amazon, GitHub**.

---

## 🎯 Objectifs

- 💨 Réduire les conflits de merge et la dette d’intégration  
- ⚡ Livrer plus vite et plus régulièrement  
- 🪴 Obtenir un feedback plus tôt  
- 🧱 Garder le code proche de l’état déployable  
- 🔄 Faciliter le **déploiement continu (CI/CD)**  

---

## 🧭 Principes clés

| Concept | Description |
|--------|-------------|
| **Une seule branche principale** | `main` représente la version stable prête à être déployée. |
| **Branches courtes** | Durée de vie de quelques heures à 1–2 jours maximum. |
| **Intégration fréquente** | On fusionne tôt et souvent, pas uniquement en fin de développement. |
| **Feature Flags** | Les fonctionnalités incomplètes sont intégrées mais **désactivées**. |
| **CI stricte** | Les tests doivent s’exécuter automatiquement à chaque commit/pull request. |

---

## 🔄 Flow de travail

```
1. Créer une petite branche depuis main
2. Faire une petite portion de la fonctionnalité
3. Ouvrir rapidement une Merge/Pull Request
4. CI exécute automatiquement les tests
5. Revue de code courte et focus
6. Fusion dans main
7. Déploiement automatique ou semi-automatique
```

💡 **Règle d’or:** *une branche ne devrait pas durer plus de 1–2 jours.*

---

## 🏁 Feature Flags : la clé du TBD

Les **Feature Flags** permettent d’intégrer des fonctionnalités **avant qu’elles soient complètement prêtes**, en les masquant à l’utilisateur final.  
Cela garantit une branche principale **stable**, même si le développement avance par étapes.

### Pourquoi c’est essentiel

| Sans Feature Flags ❌ | Avec Feature Flags ✅ |
|----------------------|----------------------|
| Branches longues | Branches très courtes |
| Gros merges douloureux | Merges légers et continus |
| Livraison tardive | Livraison progressive et contrôlée |

---

### 🧱 Implémentation : Niveau simple (flag en configuration)

**config/features.json**
```json
{
  "enableNewCheckout": false
}
```

**services/checkoutService.js**
```js
import features from "../config/features.json" assert { type: "json" };

export function checkout(user, cart) {
  if (features.enableNewCheckout) {
    return newCheckoutFlow(user, cart);
  }
  return legacyCheckoutFlow(user, cart);
}
```

✅ Facile à mettre en place  
❗ Nécessite souvent un redeploy  

---

### 🌐 Implémentation : Niveau intermédiaire (flag dynamique)

**featureFlags.js**
```js
import db from "./db.js";

export async function isFeatureEnabled(flagName) {
  const result = await db.query(
    "SELECT enabled FROM feature_flags WHERE flag_name = $1",
    [flagName]
  );
  return result.rows[0]?.enabled === true;
}
```

**services/checkoutService.js**
```js
import { isFeatureEnabled } from "../featureFlags.js";

export async function checkout(user, cart) {
  const enabled = await isFeatureEnabled("new_checkout");
  return enabled ? newCheckoutFlow(user, cart) : legacyCheckoutFlow(user, cart);
}
```

✅ Activation/désactivation **sans redeploy**  
✅ Rollback immédiat  
✅ Permet tests internes / beta / gradual rollout  

---

### 🎚️ Activation progressive (rollout au pourcentage)

```js
function userInRollout(userId, percentage) {
  const hash = [...userId].reduce((a, c) => a + c.charCodeAt(0), 0);
  return hash % 100 < percentage;
}

if (await isFeatureEnabled("new_checkout") && userInRollout(user.id, 10)) {
  return newCheckoutFlow(user, cart); // activé pour ~10% des utilisateurs
}
```

---

## 🔥 Comparaison avec d’autres workflows

| Workflow | Avantages | Inconvénients | À adopter si… |
|---------|-----------|---------------|---------------|
| **Git Flow** | Structure, releases claires | Branches longues, merges lourds | Releases espacées, validation formelle, logiciels distribués |
| **GitHub Flow** | Simple, léger, rapide | Pas adapté aux fonctionnalités longues | Petites équipes, web continu |
| **Trunk-Based Dev** | Livraisons rapides, faible coût d’intégration | Requiert CI + discipline | Organisation orientée produit, cadence rapide |

---

## 📚 Ressources

- Site officiel TBD → https://trunkbaseddevelopment.com/  
- Martin Fowler sur les Feature Toggles → https://martinfowler.com/articles/feature-toggles.html  
- Documentation Atlassian → https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development  
- Recherche DORA / Google → https://cloud.google.com/devops  

---

## ✅ Résumé visuel

```
✔ Livraison en petits lots
✔ Branches ultra-courtes
✔ Fonctionnalités cachées via Feature Flags
✔ CI obligatoire
➡ Moins de conflits, plus de fluidité, meilleure qualité
```

🎉 *Objectif : livrer mieux, plus sereinement et plus souvent.*