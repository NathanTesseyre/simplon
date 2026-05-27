# 🧪 Série d’exercices — Tests Unitaires pour Node.js

## 🎯 Objectif
Écrire des tests unitaires pour des fonctions données, en distinguant :
- Les tests de **fonctions pures** (sans mock)
- Les tests de **fonctions avec dépendances** (avec mock)

---

## Consignes générales
- 1 test = 1 comportement clair
- Pas d'accès à la base, réseau, fichiers → **test unitaire = isolation**
- Utiliser Jest ou Vitest (`vitest` recommandé)
- Nommer les fichiers `*.test.js`

Exemple bloc de test :
```js
test("décrit précisément le comportement attendu", () => {
  // Arrange
  // Act
  // Assert
});
```

---

## Partie A — Exercices sans mock (fonctions pures)

### A1 — `add(a, b)`
```js
// add.js
export function add(a, b) {
  return a + b;
}
```
**À tester**
- `add(2, 3) === 5`
- Nombres négatifs, zéro
- (optionnel) comportement si types invalides

---

### A2 — `applyPercentage(price, pct)`
```js
// applyPercentage.js
export function applyPercentage(price, pct) {
  if (typeof price !== 'number' || typeof pct !== 'number') throw new TypeError('numbers expected');
  if (pct < 0 || pct > 100) throw new RangeError('pct out of range');
  const result = price * (1 - pct / 100);
  return Math.round(result * 100) / 100;
}
```
**À tester**
- Cas nominal
- `pct = 0`, `pct = 100`
- Arrondis
- Erreurs types / range

---

### A3 — `isValidSku(sku)`
```js
// isValidSku.js
export function isValidSku(sku) {
  return /^[A-Z]{3}-\d{4}$/.test(sku);
}
```
**À tester**
- Valides / invalides (minuscules, longueurs, caractères)

---

### A4 — `calcOrderTotal(items)`
```js
// calcOrderTotal.js
export function calcOrderTotal(items) {
  if (!Array.isArray(items)) throw new TypeError('items array required');
  return items.reduce((sum, it) => {
    const price = Number(it?.price ?? 0);
    const qty = Number(it?.qty ?? 0);
    if (!Number.isFinite(price) || !Number.isFinite(qty)) {
      throw new TypeError('invalid item');
    }
    return sum + price * qty;
  }, 0);
}
```
**À tester**
- Cas nominal
- Tableau vide
- Erreurs si éléments invalides

---

### A5 — `formatMoney(amount, currency='EUR', locale='fr-FR')`
```js
// formatMoney.js
export function formatMoney(amount, currency = 'EUR', locale = 'fr-FR') {
  if (typeof amount !== 'number') throw new TypeError('amount number expected');
  return new Intl.NumberFormat(locale, { style: 'currency', currency }).format(amount);
}
```
**À tester**
- Valeurs + / 0 / négatives
- Autres devises / locales

---

## Partie B — Exercices avec dépendances (introduction au mock)

### B1 — `OrderService.totalPrice(orderId)`
```js
// order.service.js
export class OrderService {
  constructor(repo) { this.repo = repo; }
  async totalPrice(orderId) {
    const order = await this.repo.get(orderId);
    return order.items.reduce((sum, it) => sum + it.price * it.qty, 0);
  }
}
```
**À tester**
- Mock repo.get
- Vérifier que `repo.get` est appelé avec le bon ID
- Total correct
- Items vide → total 0

---

### B2 — `OrderService.totalWithDiscount(orderId, pct)`
**À tester avec un repo mocké**
- Pourcentage valide (0%, 10%, 100%)
- Erreur si `pct` hors limite
- Vérifier arrondis

---

### B3 — `InventoryService.isOrderFulfillable(orderId)`
```js
// inventory.service.js
export class InventoryService {
  constructor(repo) { this.repo = repo; }
  async isOrderFulfillable(orderId) {
    const order = await this.repo.getOrder(orderId);
    for (const item of order.items) {
      const available = await this.repo.getStock(item.sku);
      if (available < item.qty) return false;
    }
    return true;
  }
}
```
**À tester**
- Tous les stocks suffisants → true
- Un stock insuffisant → false
- Vérifier appel `getOrder(orderId)`

---

## Partie C — Exercices design pour testabilité

### C1 — Injection de dépendance (refactor)
Refactorer pour remplacer `Date.now()` par une fonction injectée.  
Tester avec une valeur contrôlée.

### C2 — Séparation logique pure vs I/O
Isoler `computeShipping(weightKg)` et le tester sur plusieurs seuils.

---

## Bonus — Vitest (mock rapide)
```js
import { vi } from 'vitest';
const repoMock = {
  get: vi.fn().mockResolvedValue({ items: [ { price: 10, qty: 2 } ] })
};
```

---

## ✅ Checklist d’auto-évaluation
- Tests lisibles et nommés par comportement
- Cas nominaux + cas bord + erreurs couverts
- Pas de mock quand la fonction est pure
- Mock minimal quand dépendance externe
- Tests indépendants entre eux

