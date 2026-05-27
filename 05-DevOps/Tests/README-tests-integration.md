# Mise en place des tests d’intégration avec Vitest + Testcontainers

Ce guide documente la migration depuis un projet **sans tests d’intégration** vers une configuration complète, maintenable et automatisée basée sur :

- **Vitest** (tests unitaires + intégration)
- **Testcontainers** pour lancer un **PostgreSQL réel** pendant les tests
- **Schéma unique** appliqué depuis `db/init/0001_init.sql`
- **Setup global** (pas de `beforeAll` à répéter)
- **Injection automatique des services** dans `globalThis`

---

## 1) Installation des dépendances

```bash
npm i -D vitest @testcontainers/postgresql
npm i pg
```

Assurez-vous également que **Docker** est installé et actif.

---

## 2) Structure finale du projet

```
taskboard/
  db/
    init/
      0001_init.sql         ← Schéma réel utilisé pour la base de test
  src/
    db/
      adapter.js
    repositories/
      tasks.repo.js
    services/
      tasks.service.js
  tests/
    helpers/
      global-setup.js       ← Lance/stops PostgreSQL + applique le schéma
      per-worker-setup.js   ← Prépare tasksService → globalThis
    integration/
      tasks.test.js         ← Exemple de test d’intégration
  vitest.common.ts
  vitest.unit.config.ts
  vitest.integration.config.ts
```

---

## 3) Fichiers helpers

### `tests/helpers/global-setup.js`

```js
import { PostgreSqlContainer } from "@testcontainers/postgresql";
import fs from "node:fs";
import pg from "pg";
import path from "node:path";

export default async function () {
  const container = await new PostgreSqlContainer("postgres:16-alpine")
    .withDatabase("taskboard")
    .withUsername("taskboard")
    .withPassword("taskboard")
    .start();

  process.env.NODE_ENV = "test";
  process.env.DATABASE_URL = container.getConnectionUri();

  const INIT_SQL = path.resolve(process.cwd(), "db/init/0001_init.sql");
  const ddl = fs.readFileSync(INIT_SQL, "utf8");

  const client = new pg.Client({ connectionString: process.env.DATABASE_URL });
  await client.connect();
  await client.query(ddl);
  await client.end();

  return async () => {
    try {
      const { database } = await import("~db/adapter.js");
      await database.close?.();
    } catch {}
    await container.stop({ remove: true });
  };
}
```

---

### `tests/helpers/per-worker-setup.js`

```js
const { database } = await import("~db/adapter.js");
const { makeTasksRepo } = await import("~repositories/tasks.repo.js");
const { makeTasksService } = await import("~services/tasks.service.js");

globalThis.tasksService = makeTasksService(makeTasksRepo(database));
```

---

## 4) Exemple de test d’intégration

### `tests/integration/tasks.test.js`

```js
import { describe, it, expect } from "vitest";

describe("Tasks Service (Integration)", () => {
  it("crée, lit, met à jour, supprime une tâche", async () => {
    const service = globalThis.tasksService;

    const created = await service.create({ title: "Hello Testcontainers" });
    expect(created.title).toBe("Hello Testcontainers");

    const updated = await service.updateStatus(created.id, "doing");
    expect(updated.status).toBe("doing");

    const removed = await service.remove(created.id);
    expect(removed).toBe(true);
  });
});
```

---

## 5) Configuration Vitest

### `vitest.common.ts`

```ts
import type { UserConfig as ViteUserConfig } from "vite";
import path from "node:path";

export const common: ViteUserConfig = {
  resolve: {
    alias: {
      "~db": path.resolve(__dirname, "src/db"),
      "~repositories": path.resolve(__dirname, "src/repositories"),
      "~services": path.resolve(__dirname, "src/services"),
      "~app": path.resolve(__dirname, "src"),
    },
  },
};
```

### `vitest.unit.config.ts`

```ts
import { defineConfig } from "vitest/config";
import { common } from "./vitest.common";

export default defineConfig({
  ...common,
  test: {
    name: "unit",
    include: ["tests/unit/**/*.test.[jt]s"],
  },
});
```

### `vitest.integration.config.ts`

```ts
import { defineConfig } from "vitest/config";
import { common } from "./vitest.common";

export default defineConfig({
  ...common,
  test: {
    name: "integration",
    include: ["tests/integration/**/*.test.[jt]s"],
    globalSetup: ["tests/helpers/global-setup.js"],
    setupFiles: ["tests/helpers/per-worker-setup.js"],
    testTimeout: 120000,
    hookTimeout: 120000
  },
});
```

---

## 6) Scripts `package.json`

```json
{
  "scripts": {
    "test": "vitest run -c vitest.unit.config.ts",
    "test:unit": "vitest run -c vitest.unit.config.ts",
    "test:integration": "vitest run -c vitest.integration.config.ts",
    "test:all": "npm run test:unit && npm run test:integration"
  }
}
```

---

## ✅ Résultat

| Avantage | Détail |
|---|---|
| Tests unitaires **rapides** | aucune dépendance Docker |
| Tests d’intégration **réalistes** | vraie base PostgreSQL lancée automatiquement |
| Zéro duplication de schéma | `db/init/0001_init.sql` = **source unique** |
| Pas de boilerplate | services injectés via `globalThis` |
| Structure claire | `tests/unit` vs `tests/integration` |

---

Fin 🎉
Bon tests !
