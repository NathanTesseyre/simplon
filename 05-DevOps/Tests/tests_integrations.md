---
title: "Tests d'intégration"
tags:
  - tests
  - intégration
  - API
section: 05-DevOps
domaine: Tests
statut: actif
liens_connexes:
  - [[tests_unitaires]]
  - [[tests_integration_guide]]
  - [[docker_compose]]
  - [[fiche_database]]
---

# 🧪 Tests d’Intégration
**Stack : Node.js • Express • PostgreSQL**

---

## 🎯 Objectifs pédagogiques

Vous allez apprendre à :

- Comprendre **ce qu’est un test d’intégration**
- Différencier **test unitaire** et **test d’intégration**
- Utiliser un **cas métier concret** pour concevoir les tests
- Identifier **ce qui doit être testé**
- Structurer des **tests lisibles, cohérents et maintenables**

---

## 1) Qu’est-ce qu’un test d’intégration ?

Un **test d’intégration** vérifie que plusieurs parties du système **fonctionnent correctement ensemble**, dans un contexte proche du réel :

```
(HTTP) Express → Service métier → PostgreSQL → Réponse JSON
```

### ✅ Ce que l’on teste
- Le **parcours des données**
- Le **comportement réel** du système
- Les **interactions** entre composants

### ❌ Ce que ce n’est pas
- Un test unitaire (on ne teste pas une fonction isolée)
- Un test mocké (la base doit être réelle)

> **Un test d’intégration vérifie des interactions, pas du calcul isolé.**

---

## 2) 🆚 Test Unitaire vs Test d’Intégration

| Aspect | **Test Unitaire (TU)** | **Test d’Intégration (TI)** |
|---|---|---|
| Portée | Une seule fonction | Plusieurs modules ensemble |
| Base de données | ❌ simulée / mockée | ✅ réelle |
| Objectif | Vérifier la logique interne | Vérifier le comportement global |
| Vitesse | ⚡ Très rapide | 🐢 Plus lent mais réaliste |
| Ce qu’il répond | “Est-ce que mon code **calcule juste** ?” | “Est-ce que le système fonctionne **pour de vrai** ?” |

**À retenir :**  
> **TU = on isole**  
> **TI = on connecte**

---

## 3) 🧱 Contexte métier utilisé

Nous utilisons un exemple simple de **gestion de commandes** :

```
Commande : order-001

Articles :
• (SKU-001, 10€, qty 2) → 20€
• (SKU-002, 5€, qty 1)  →  5€

Total attendu = 25€
```

Cet exemple permet de couvrir :

| Aspect | Pourquoi c'est utile |
|---|---|
| Insertion & lecture SQL | On valide la persistance |
| Relation commande ↔ articles | On teste les jointures |
| Calcul métier (total) | On teste la logique appliquée aux données |
| Appels HTTP Express | On teste le comportement observable |

---

## 4) Exemples de comportements à tester

| Cas métier | Attendu |
|---|---|
| Création + récupération d’une commande | Les données envoyées sont réellement stockées et retrouvées |
| Calcul du total | Le total dépend bien de SUM(price × qty) en base |
| Données invalides | L’API renvoie une erreur claire (pas une stack SQL) |
| Suppression de commande | Les articles liés sont supprimés (intégrité) |
| Commande inexistante | L’API renvoie 404 propre |
| Plusieurs créations simultanées | La base reste cohérente |

> Chaque test vérifie **un scénario métier**, pas une ligne de code.

---

## 5) Modèles de tests d’intégration (avec exemples condensés)

> On suppose que `server` (Express) et `pool` (PostgreSQL) sont disponibles

### 🟢 Création d’une commande

```js
test("crée une commande", async () => {
  await request(server)
    .post("/orders")
    .send({
      id: "order-001",
      items: [{ sku: "SKU1", price: 10, qty: 2 }]
    })
    .expect(201);

  const result = await pool.query("SELECT * FROM orders WHERE id = $1", ["order-001"]);
  expect(result.rowCount).toBe(1);
});
```

---

### 🟢 Calcul du total

```js
test("calcule correctement le total", async () => {
  const res = await request(server).get("/orders/order-001/total");
  expect(res.body.total).toBe(25);
});
```

---

### 🟢 Données invalides rejetées proprement

```js
test("rejette une quantité invalide", async () => {
  const res = await request(server)
    .post("/orders")
    .send({ id: "bad", items: [{ sku: "X", price: 10, qty: 0 }] });

  expect(res.status).toBe(400);
  expect(res.body.error).toMatch(/quantité/);
});
```

---

### 🟢 Suppression + cascade

```js
test("supprime la commande et ses items", async () => {
  await request(server).delete("/orders/order-001").expect(204);

  const items = await pool.query("SELECT * FROM order_items WHERE order_id = $1", ["order-001"]);
  expect(items.rowCount).toBe(0);
});
```

---

### 🟢 Ressource inexistante

```js
test("renvoie 404 si la commande n'existe pas", async () => {
  const res = await request(server).get("/orders/inconnue/total");
  expect(res.status).toBe(404);
});
```

---

## 🧠 À retenir

> **Un test d’intégration teste un comportement métier réel.**  
> Si tu **mockes la base**, ce n’est **plus** un test d’intégration.
