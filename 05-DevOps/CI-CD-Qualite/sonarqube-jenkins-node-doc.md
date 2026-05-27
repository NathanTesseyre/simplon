---
title: "SonarQube + Jenkins — Analyse qualité"
tags:
  - sonarqube
  - jenkins
  - qualité
  - ci-cd
section: 05-DevOps
domaine: CI/CD
statut: actif
liens_connexes:
  - [[02_setup_gitea_jenkins]]
  - [[03_setup_jenkins_docker_agent]]
  - [[docker_fondamentaux]]
  - [[tests_unitaires]]
  - [[tests_integrations]]
---

# Setup SonarQube + Intégration Jenkins (Node.js + Vitest + Testcontainers)

Ce guide **simple et prêt à l’emploi** couvre :
1) **Installation / setup de SonarQube** en local
2) **Intégration SonarQube ↔ Jenkins** (serveur + webhook + scanner)
3) **Configuration du projet Node.js** (Vitest UT/IT + couverture)
4) **Jenkinsfile final** adapté à ton **agent `docker-agent`** (socket Docker déjà monté) — sans options `.inside(...)` ni installation du `docker-cli`

---

## 1) Installer SonarQube en local (Docker)

```bash
docker run -d --name sonarqube   -p 9000:9000   -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true   sonarqube:lts-community
```

- Ouvre http://localhost:9000 → connecte-toi `admin` / `admin` puis change le mot de passe.
- Va dans **My Account → Security → Generate Tokens** et crée un **token** (ex. `jenkins-local`). Garde-le de côté.
- (Optionnel) Crée un **Project** et note le **Project Key** (nous pouvons aussi le fournir côté scanner).

> Si Jenkins n’est **pas** sur la même machine que SonarQube, remplace `http://localhost:9000` par l’URL du serveur SonarQube accessible depuis Jenkins.

---

## 2) Intégration SonarQube ↔ Jenkins

### A. Installer le plugin
Dans Jenkins → **Manage Jenkins → Plugins** : installe **SonarQube Scanner for Jenkins**.

### B. Ajouter le token SonarQube en Credentials
**Manage Jenkins → Credentials → (global)** → **Add Credentials**  
- *Kind*: **Secret text**  
- *Secret*: le token SonarQube généré ci‑dessus  
- *ID*: `sq-token` (par ex.)

### C. Déclarer le serveur SonarQube
**Manage Jenkins → System → SonarQube servers**  
- *Name*: `local-sq`  
- *Server URL*: `http://localhost:9000` (ou l’URL de ton SonarQube)  
- *Server authentication token*: choisis le credentials `sq-token`

### D. Déclarer l’outil “SonarQube Scanner”
**Manage Jenkins → Global Tool Configuration → SonarQube Scanner**  
- *Name*: `SonarQubeScanner`  
- coche **Install automatically**

### E. Configurer le Webhook (Quality Gate)
Dans l’UI SonarQube → **Administration → Configuration → Webhooks**  
- *Name*: `jenkins`  
- *URL*: `http://<jenkins-host>:8080/sonarqube-webhook/`

> Le webhook est **nécessaire** pour que `waitForQualityGate` change l’état du pipeline.

---

## 3) Configuration du projet Node.js

### A. Dépendances CI
Installe la couverture LCOV + reporter JUnit + merge LCOV :
```bash
npm i -D @vitest/coverage-v8 vitest-junit lcov-result-merger
```

### B. Scripts `package.json`
Ajoute/modifie les scripts suivants :
```json
{
  "scripts": {
    "test": "npm run test:ci",
    "test:unit": "vitest run -c vitest.unit.config.ts --coverage",
    "test:integration": "vitest run -c vitest.integration.config.ts --coverage",
    "coverage:merge": "lcov-result-merger 'coverage/**/lcov.info' 'coverage/lcov.info'",
    "test:ci": "npm run test:unit && npm run test:integration && npm run coverage:merge"
  }
}
```
- Les rapports JUnit seront émis par **vitest-junit** dans les chemins configurés dans tes configs Vitest.
- La couverture fusionnée est écrite dans `coverage/lcov.info` (consommée par SonarQube).

> Si besoin, ajoute dans tes configs Vitest (UT/IT) :
> - `coverage.provider: 'v8'`, `reporter: ['lcov','text-summary']` et des **dossiers** distincts (`coverage/unit`, `coverage/integration`)
> - reporter JUnit (ex: `['vitest-junit', { outputFile: 'reports/junit/junit-unit.xml' }]` côté UT et `junit-int.xml` côté IT)

### C. Fichier `sonar-project.properties` (à la racine)
```
sonar.projectKey=taskboard-app
sonar.projectName=taskboard-app
sonar.sourceEncoding=UTF-8

# Sources & tests
sonar.sources=src
sonar.tests=tests
sonar.test.inclusions=tests/**/*.test.{js,ts}

# Couverture (merge UT + IT)
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

---

## 4) Jenkinsfile final (agent `docker-agent` avec socket déjà monté)

> Ce Jenkinsfile respecte ta contrainte : **on utilise `.inside { ... }` sans arguments**, et **on ne fait pas d’installation docker‑cli**.  
> Testcontainers utilisera l’accès Docker déjà fourni par le label `docker-agent`.

```groovy
pipeline {
  agent { label 'docker-agent' } // agent déjà configuré avec accès Docker

  options { timestamps() }

  stages {

    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Tests (Node container)') {
      steps {
        script {
          docker.image('node:20-alpine').inside {

            // Dépendances
            sh 'npm ci'

            // Unit tests
            sh 'npm run test:unit'
            junit testResults: 'reports/junit/junit-unit.xml', allowEmptyResults: true

            // Integration tests (Testcontainers)
            withEnv([
              'TESTCONTAINERS_CHECKS_DISABLE=true',
              'TESTCONTAINERS_RYUK_DISABLED=false'
            ]) {
              sh 'npm run test:integration'
            }
            junit testResults: 'reports/junit/junit-int.xml', allowEmptyResults: true

            // Merge coverage (UT+IT)
            sh 'npm run coverage:merge'

            // Analyse SonarQube
            withSonarQubeEnv('local-sq') {
              withEnv(['PATH+SONAR=${TOOL_SonarQubeScanner}/bin']) {
                sh '''
                  sonar-scanner                     -Dsonar.host.url=$SONAR_HOST_URL                     -Dsonar.login=$SONAR_AUTH_TOKEN
                '''
              }
            }
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 10, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }
}
```

---

## 5) Checklist / Dépannage rapide

- **Webhook Sonar → Jenkins** en place (`/sonarqube-webhook/`) sinon *Quality Gate* reste *Pending*.
- **Couverture = 0%** : vérifie que `coverage/lcov.info` est généré **avant** l’étape SonarQube.
- **Testcontainers** : l’agent `docker-agent` expose déjà le socket → pas besoin de monter quoi que ce soit dans `.inside { }`.
- **JUNIT** : assure les chemins utilisés dans `junit` matchent les fichiers générés par `vitest-junit`.

---

## 6) Résultat attendu

| Élément | OK quand… |
|---|---|
| UT + IT | Les deux stages passent et publient JUnit |
| Couverture | `coverage/lcov.info` existe et SonarQube l’ingère |
| Sonar | Projet visible avec métriques & Quality Gate |
| Pipeline | Se termine en SUCCESS si la Quality Gate est verte |
