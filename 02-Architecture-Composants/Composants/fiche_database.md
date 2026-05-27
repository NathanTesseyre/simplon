---
title: "Les bases de données"
tags:
  - composants
  - BDD
  - SQL
  - NoSQL
section: 02-Architecture-Composants
domaine: Composants
statut: actif
liens_connexes:
  - [[fiche_backend]]
  - [[fiche_securite_dev]]
---

# 🗄️ Fiche Bases de données

## 🔎 Qu’est-ce qu’une base de données (DB) ?
Une **base de données** est un système permettant de **stocker, organiser et interroger des informations** de manière fiable et efficace.
Le **backend** s’appuie sur la DB pour persister et récupérer ses données.

---

## 🧱 Rôles principaux d’une DB
- **Stockage** : conserver les données de façon durable.
- **Organisation** : structurer les données (tables, documents, graphes).
- **Accès rapide** : requêtes optimisées grâce aux index.
- **Intégrité** : garantir cohérence et règles (contraintes, transactions).
- **Sécurité** : contrôler l’accès aux données sensibles.

---

## ❌ Pourquoi le frontend ne parle pas directement à la base ?
Le frontend **n’interagit jamais directement** avec la DB, car :
- **Sécurité** : la DB contient des données sensibles. L’exposer serait catastrophique.
- **Logique métier** : le **backend** applique les règles (plafonds bancaires, validation commande…). Sans lui, elles peuvent être contournées.
- **Cohérence & validation** : le backend valide les données avant de les stocker.
- **Évolutivité** : le backend expose une API stable, le frontend n’a pas besoin de connaître le schéma interne.

👉 *Exemple* : sans backend, un utilisateur pourrait modifier un prix à 1 €. Avec backend, la règle « le prix vient du catalogue » l’empêche.

---

## ⚙️ Types de bases de données



### 🖼️ SQL vs NoSQL — critères de choix
![SQL vs NoSQL](img/sql_vs_nosql.png)


### 1) Bases relationnelles (SQL)
- Modèle : tables, lignes, colonnes.
- Langage : **SQL**.
- Exemples : MySQL, PostgreSQL, Oracle, SQL Server.
- ✅ Avantages : cohérence forte, transactions **ACID**, normalisation.
- ⚠️ Limites : rigidité du schéma, scaling horizontal plus complexe.

![Aperçu SQL](img/sql_overview.png)

#### 📊 Principales BDD SQL et quand les choisir

| SGBD | Points forts | Limites | Cas d’usage typiques |
|------|--------------|---------|----------------------|
| **PostgreSQL** | Très robuste, riche (JSON, extensions), open source | Administration plus exigeante que MySQL | SaaS, finance, apps critiques |
| **MySQL/MariaDB** | Simple, rapide, très répandu | Moins avancé sur certaines fonctions | Sites web, e‑commerce |
| **Oracle** | Performance extrême, support entreprise | Très cher, propriétaire | Banque, assurance, grands comptes |
| **SQL Server** | Intégration Microsoft | Licence coûteuse | Entreprises Microsoft, ERP |

---

### 2) Bases NoSQL
- Familles : **Clés‑Valeurs**, **Documents**, **Colonnes**, **Graphes**.
- ✅ Avantages : flexibles, scalables horizontalement, gros volumes.
- ⚠️ Limites : cohérence plus faible selon les moteurs.

![Guide de décision NoSQL](img/nosql_decision.png)

#### 📊 Familles NoSQL et choix

| Famille | Exemples | Points forts | Cas d’usage |
|------|----------|--------------|-------------|
| **Clé‑Valeur** | Redis | Ultra rapide, simple | Cache, sessions, files |
| **Documents** | MongoDB, CouchDB | Schéma flexible, JSON | Réseaux sociaux, apps mobiles |
| **Colonnes** | Cassandra, HBase | Scalabilité massive | Big Data, logs, IoT |
| **Graphes** | Neo4j | Relations complexes | Social, recommandations |
|

### 🖼️ Critères de choix (SQL vs NoSQL)

En résumé, le choix entre **SQL et NoSQL** dépend surtout de :  
- La **cohérence** immédiate nécessaire (finance, transactions critiques → SQL).  
- La **flexibilité du schéma** (données variées, évolutives → NoSQL).  
- La **scalabilité horizontale** (volumétrie énorme, faible latence → NoSQL).  
- Les **relations complexes** (graphes sociaux → NoSQL Graphes).  

![SQL vs NoSQL](img/sql_vs_nosql_criteria.png)

---

## 🧩 Transactions & cohérence — pourquoi c’est essentiel
Lorsqu’une appli manipule des données, il faut garantir leur **cohérence**.

**Exemples métier**
- **Banque** : un virement doit débiter *et* créditer (jamais l’un sans l’autre).
- **E‑commerce** : valider une commande = débit + stock – facture.
- **Réseau social** : un like doit être enregistré malgré la concurrence.

**Deux approches**
- **ACID** (souvent SQL) : **Atomicité** (tout ou rien), **Cohérence**, **Isolation**, **Durabilité**.
- **BASE** (souvent NoSQL distribué) : **Basically Available**, **Soft state**, **Eventually consistent** (cohérence atteinte avec délai).  
  👉 Adapté quand la **vitesse et la scalabilité priment** (analytics, timelines).

---

## 🔄 Communication Backend ↔ DB
- Le **Frontend** interroge le **Backend** via une API.
- Le **Backend** applique logique métier et sécurité.
- Puis il interagit avec la **DB** via un **driver** ou un **ORM** (TypeORM, Hibernate, Eloquent).

![Flux Backend ↔ DB](img/backend_db_comm.png)

> Ainsi, **toutes les requêtes passent par le Backend**, garantissant règles métiers et sécurité.

---

## 🧩 Exemples concrets
- **E‑commerce** : PostgreSQL pour catalogue/commandes, Redis pour cache panier.
- **Réseau social** : MongoDB pour posts, Neo4j pour relations.
- **Bancaire** : Oracle pour transactions, PostgreSQL pour audit.

---

## 🛠️ Bonnes pratiques
- **Bien concevoir sa base dès le départ**
  - Modéliser entités/relations (UML, Merise).
  - **Normaliser** en SQL (éviter duplications), **dénormaliser** en NoSQL si nécessaire.
  - Anticiper les requêtes critiques → **index adaptés**.
- **Indexation** : accélère les lectures (coût en écriture à surveiller).
- **Sécurité** : utilisateurs DB à droits minimaux, chiffrement des données sensibles.
- **Redondance et haute disponibilité**
  - Éviter le **SPOF** : pas d’instance unique.
  - Réplication maître‑esclave / primaire‑répliques (lectures/écritures séparées).
  - **Clusters** (PostgreSQL + Patroni ; MongoDB ReplicaSet).
  - **Réplication géographique** pour résilience datacenter.
- **Backups & restauration testée** : un backup non testé ne sert à rien.
- **Monitoring** : tables qui grossissent, requêtes lentes, erreurs.

---

## 🧪 À vérifier après déploiement
- **Connexion** : DB accessible uniquement depuis le backend.
- **Performances** : `EXPLAIN` (SQL), index manquants.
- **Sécurité** : pas de comptes admin exposés, TLS activé.
- **Backups** : planifiés et testés régulièrement.
- **Logs** : erreurs et lenteurs surveillées.



---