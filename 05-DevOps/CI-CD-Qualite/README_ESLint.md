# 🧹 Linting (ESLint)

Ce guide explique comment ajouter et configurer **ESLint** à ton projet TaskBoard pour garantir une qualité de code homogène et détecter les erreurs courantes.

---

## ⚙️ Installation rapide

1️⃣ **Installer les dépendances**
```bash
npm i -D eslint @eslint/js globals eslint-plugin-import eslint-plugin-n eslint-plugin-promise
```

2️⃣ **Créer le fichier de configuration** `eslint.config.js` à la racine du projet :
```js
// eslint.config.js (Flat config, ESLint ≥ 9)
import js from '@eslint/js'
import globals from 'globals'
import importPlugin from 'eslint-plugin-import'
import n from 'eslint-plugin-n'
import promise from 'eslint-plugin-promise'

export default [
  {
    files: ['**/*.js'],
    ignores: ['node_modules/**', 'coverage/**', 'reports/**'],
    languageOptions: {
      ecmaVersion: 2022,
      sourceType: 'module',
      globals: {
        ...globals.node,
      },
    },
    plugins: {
      import: importPlugin,
      n,
      promise,
    },
    rules: {
      ...js.configs.recommended.rules,
      'import/order': ['warn', { 'newlines-between': 'always' }],
      'no-console': 'off'
    },
  },
  // Reconnaître les globals des tests (Vitest)
  {
    files: ['tests/**/*.js'],
    languageOptions: {
      globals: {
        ...globals.node,
        vi: 'readonly',
        describe: 'readonly',
        it: 'readonly',
        expect: 'readonly',
        beforeAll: 'readonly',
        afterAll: 'readonly',
        beforeEach: 'readonly',
        afterEach: 'readonly'
      },
    },
  },
]
```

3️⃣ **Ajouter les scripts dans `package.json`**
```jsonc
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix"
  }
}
```

4️⃣ **Utilisation**
```bash
npm run lint
npm run lint:fix
```

---

## 💡 Conseils utiles

- Ajoute le dossier `node_modules` et les fichiers générés (`coverage`, `reports`, etc.) à `.eslintignore` si tu utilises encore une config non-flat.
- Tu peux personnaliser les règles selon ton style de code :  
  → https://eslint.org/docs/latest/rules/
- Si tu veux un affichage plus lisible dans le terminal, installe aussi :  
  ```bash
  npm i -D eslint-formatter-pretty
  ```
  et exécute :
  ```bash
  npx eslint . -f pretty
  ```

---

## 🧩 Intégration avec Vitest

Le fichier de config ci-dessus reconnaît déjà les variables globales des tests Vitest (`describe`, `it`, `expect`, etc.).  
Cela permet d'éviter les faux positifs “variable non définie” dans les fichiers de test.

---

## 🧾 Option CI (Intégration continue)

Pour intégrer le linting dans ta **pipeline Jenkins**, ajoute simplement une étape dans ton `Jenkinsfile` après le checkout et avant les tests :

```groovy
pipeline {
  agent { label 'docker-agent' }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Lint') {
      steps {
        script {
          docker.image('node:20-alpine').inside {
            sh '''
              echo "🔍 Running ESLint..."
              npm ci
              npm run lint || true
            '''
          }
        }
      }
    }

    stage('Tests') {
      steps {
        script {
          docker.image('node:20-alpine').inside {
            sh 'npm test'
          }
        }
      }
    }
  }

  post {
    always {
      echo 'Pipeline terminée (lint + tests).'
    }
  }
}
```

### 🧠 Explications
- `npm run lint` est exécuté dans le même conteneur Node que les tests.  
- Le `|| true` permet de **ne pas bloquer le pipeline** si des erreurs de style sont détectées (tu peux le retirer si tu veux rendre le lint bloquant).  
- Tu peux utiliser `eslint . -f json -o reports/eslint-report.json` si tu veux publier le résultat en artefact.

---

## ✅ Résumé

| Élément | Fichier / Commande |
|----------|--------------------|
| Config ESLint | `eslint.config.js` |
| Lancer le lint | `npm run lint` |
| Corriger automatiquement | `npm run lint:fix` |
| Compatible Vitest | ✅ Oui |
| Compatible ESM / Node ≥ 18 | ✅ Oui |
| Intégration CI Jenkins | ✅ Étape “Lint” dans le pipeline |

---

🎯 **Objectif** : un code propre, cohérent, et validé automatiquement dans ta CI avant l’exécution des tests.
