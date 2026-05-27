---
title: "Tests unitaires"
tags:
  - tests
  - TDD
  - vitest
  - jest
section: 05-DevOps
domaine: Tests
statut: actif
liens_connexes:
  - [[tests_integrations]]
  - [[exercices_tests_unitaires]]
---

# 🧪 Fiche pédagogique — Tests Unitaires (TU) — Node.js Débutants

## 🎯 Objectif
Comprendre ce qu’est un **test unitaire**, pourquoi il est essentiel, comment en écrire un **simple**, puis comment gérer le cas où une fonction dépend d’une **ressource externe** grâce au **mock**.

---

## 1) Qu’est-ce qu’un test unitaire ?

Un **test unitaire** sert à vérifier **une seule fonction**, **en isolation**.

### ✅ Un bon test unitaire est :
- **Rapide** (exécution en millisecondes)
- **Stable** (même résultat à chaque fois)
- **Simple** (une seule idée testée)
- **Sans dépendance externe** (pas de base de données, pas de réseau, pas de fichiers)

> 💡 **Idée clé** : On teste **la logique**, pas l’environnement autour.

---

## 2) 🟢 Exemple de test unitaire simple (sans dépendance externe)

Ici, la fonction ne dépend que de ses **paramètres** → elle peut être testée directement.

```js
// add.js
export function add(a, b) {
  return a + b;
}
```

```js
// add.test.js
import { add } from './add.js';

test("additionne deux nombres", () => {
  expect(add(2, 3)).toBe(5);
});
```

### 💡 Pourquoi ce test est unitaire ?
- Pas de base de données
- Pas d’appel à un service externe
- Juste des paramètres → résultat → vérification

➡️ **Aucun besoin de mock ici.**

---

## 3) 🟠 Quand un test simple ne suffit plus

Dans une vraie application, une fonction ne possède pas toujours **les données** dont elle a besoin.

Exemple typique :

```
service.totalPrice(orderId) → doit d’abord récupérer la commande
```

Cela signifie que la fonction **dépend d’une ressource externe**, souvent :
- une **base de données**
- une **API externe**
- un **système de fichiers**
- ou un **autre service**

### ⚠️ Tester avec la vraie ressource pose problème :
| Problème | Conséquence |
|---|---|
| La base peut être vide, différente, cassée | Le test échoue sans rapport avec la logique |
| Accéder à la base prend du temps | Le test devient lent |
| Le comportement varie selon l’environnement | Le test devient instable |

➡️ On ne testerait plus **la logique**, mais **l’environnement**.

---

## 4) 🧩 Introduction au **Mock**

Un **mock** est une **version contrôlée** d’une dépendance externe.

### 🎯 Son rôle :
- **remplacer** la dépendance réelle (ex : base de données)
- **renvoyer une valeur définie dans le test**
- permettre de tester **uniquement la logique**

> 💡 Le mock n’est pas là pour imiter toute la base.  
Il fournit **juste ce que la fonction attend** pour travailler.

---

## 5) 🟣 Exemple de Test Unitaire **avec mock**

### Code métier
```js
// order.service.js
export class OrderService {
  constructor(repo) {        // repo = dépendance externe (ex : base de données)
    this.repo = repo;
  }

  async totalPrice(orderId) {
    const order = await this.repo.get(orderId);
    return order.items.reduce(
      (sum, item) => sum + item.price * item.qty,
      0
    );
  }
}
```

### Test unitaire
```js
// order.service.test.js
import { OrderService } from './order.service.js';

test("calcule le total d'une commande à partir de données connues", async () => {
  // On remplace la base par une version contrôlée
  const repoMock = {
    get: (orderId) => {
      // Le test vérifie que la dépendance est appelée correctement
      expect(orderId).toBe("order-001");

      return Promise.resolve({
        items: [
          { price: 10, qty: 2 }, // 20
          { price: 5, qty: 1 }   // +5 = 25
        ]
      });
    }
  };

  const service = new OrderService(repoMock);

  const total = await service.totalPrice("order-001");

  expect(total).toBe(25);
});
```

### 💡 Ce que montre ce test :
| Élément | Explication |
|---|---|
| `repoMock` | remplace la base de données |
| Le test contrôle l’entrée | plus de surprise |
| Le test mesure **uniquement** le calcul | pas l’accès aux données |
| Le comportement est stable | résultat toujours identique |

---

## 6) 🧭 Quand utiliser un mock dans un test unitaire ?

| Situation | Faut-il mocker ? | Pourquoi ? |
|---|---|---|
| La fonction dépend uniquement de ses paramètres | ❌ Non | Test direct → simple |
| La fonction dépend d’une ressource externe | ✅ Oui | On veut isoler la logique |

### Règle à retenir
> **Si la fonction peut être testée seule → pas de mock.  
Si elle dépend d’autre chose → mock.**

---

## 🧠 Synthèse

> **Un test unitaire teste une logique interne.  
Dès que cette logique dépend d’une ressource externe, on remplace cette ressource par un mock.**

