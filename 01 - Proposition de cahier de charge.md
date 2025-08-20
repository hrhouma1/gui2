# Cahier des charges – Projet Étudiants (Application Android + Backend)

## 1) Contexte & objectif

Concevoir une **application Android** connectée à un **backend** moderne (REST ou GraphQL) avec base de données relationnelle ou NoSQL. Le choix des technologies est **ouvert** et doit être **justifié**.

## 2) Périmètre fonctionnel

### 2.1 Application mobile (Android obligatoire)

Les étudiants choisissent **un** des frameworks suivants :

* **Android natif (Java/Kotlin)**
* **React Native (TypeScript/JavaScript)**
* **Flutter (Dart)**

L’app doit exploiter **au moins une** fonctionnalité du téléphone : caméra, GPS, notifications push, accéléromètre/gyroscope, baromètre, capteur de proximité, partage vers d’autres apps, intégration contacts/agenda, etc.

### 2.2 Backend (libre)

Backend exposant une **API REST** ou **GraphQL**. Exemples possibles (non limitatifs) :

* Node.js : **Express**, **NestJS**, **Next.js (API Routes)** avec **Prisma** ou **Knex**
* NoSQL : **Express/Nest + Mongoose (MongoDB)**
* Python : **Django/DRF**, **FastAPI** (+ ORM/ODM)
* Autres solutions équivalentes acceptées si motivation technique

**Base de données** : **MySQL / PostgreSQL / MSSQL** (relationnel) ou **MongoDB** (NoSQL).

### 2.3 Application web (optionnelle)

Si la portée Android est trop réduite pour le **sprint 3**, livrer une **application web** consommant le **même backend**.

## 3) Aide au choix – Comparatif rapide

### 3.1 Mobile

| Option            | Points forts                                                           | Limites                                                | Quand la choisir                                 |
| ----------------- | ---------------------------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------ |
| **Android natif** | Perf maximale, accès direct aux API Android, écosystème Android Studio | Code 100% Android, moins réutilisable multi-plateforme | Besoin d’APIs bas niveau/capteurs, perf critique |
| **React Native**  | JS/TS, large écosystème web, hot reload, partages de libs UI           | Modules natifs parfois requis, fragmentation libs      | Équipe JS/TS, time-to-market, UI réactive        |
| **Flutter**       | UI cohérente, très bon tooling, perf proche natif                      | Apprendre Dart, bundles plus lourds                    | UI custom riche, cadence de dev rapide           |

### 3.2 Backend

| Stack                       | Points forts                             | Points d’attention             | Cas typique                        |
| --------------------------- | ---------------------------------------- | ------------------------------ | ---------------------------------- |
| **Express + Knex**          | Simple, SQL explicite                    | Boilerplate, peu structuré     | Démo pédagogique, contrôle SQL     |
| **NestJS**                  | Architecture claire, TypeScript, modules | Courbe d’apprentissage         | Projet structuré moyen/long terme  |
| **Next.js API + Prisma**    | Très demandé, Prisma productif           | Couplage à Next, schema Prisma | Full-stack moderne, SaaS           |
| **Express/Nest + Mongoose** | Agile en NoSQL                           | Schéma moins strict            | Données flexibles/évolutives       |
| **Django/DRF / FastAPI**    | Mûr, batteries incluses                  | Stack Python côté équipe       | Data apps, APIs rapides et propres |

## 4) Contraintes techniques

* **Cible** : Android (APK/AAB installable sur device ou émulateur). iOS facultatif.
* **Sécurité** : gestion des permissions Android, auth basique (JWT/session), variables d’environnement, secrets non committés.
* **Qualité** : gestion des erreurs réseau, états de chargement, tests minimum (unitaire ou d’intégration).

## 5) Organisation & livrables

* **Sprints**

  * À venir !
 
* **Livrables**

  * Code mobile + backend
  * Schéma BD (relationnel ou documents)
  * Documentation API (OpenAPI/README GraphQL) + instructions d’installation
  * Démo finale

## 6) Évaluation

* Justification du **choix de la pile** (ADR) et architecture
* Intégration mobile ↔ backend ↔ base de données
* Usage pertinent des capteurs/permissions Android
* Qualité du code, UX, robustesse (erreurs/offline)
* Documentation et respect des jalons


