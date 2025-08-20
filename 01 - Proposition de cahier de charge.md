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



# Annexe 1 -  Idées de Projets Mobiles Modernes avec Stripe

## 1. **Éducation & Apprentissage**

1. Application de cours en ligne avec abonnements (vidéos + quiz).
2. Plateforme de tutorat en visio avec paiement par session.
3. Appli mobile pour apprendre les langues avec chat IA + micro-paiements.
4. Application de préparation aux examens (QCM premium).
5. SaaS pour gestion de cours privés (professeurs indépendants).

## 2. **Finance & Paiements**

6. Application de gestion de budget avec abonnement premium.
7. Portefeuille numérique avec paiements Stripe.
8. Application de suivi des dépenses d’équipe avec facturation automatique.
9. Appli de crowdfunding (mini-Ulule) avec paiements Stripe Connect.
10. SaaS de facturation simple pour freelances.

## 3. **E-commerce & Marketplace**

11. Marketplace mobile de produits artisanaux avec paiement Stripe.
12. Application de vente de vêtements seconde main avec commissions.
13. SaaS « shop mobile » pour restaurants locaux avec commandes en ligne.
14. Appli de réservation de services à domicile (plombier, ménage).
15. Plateforme de location de matériel (caméras, drones, voitures).

## 4. **Santé & Bien-être**

16. Application de coaching sportif avec paiements récurrents.
17. Plateforme de consultation médicale en ligne (paiement par rendez-vous).
18. Appli mobile de méditation avec abonnement freemium.
19. SaaS de suivi nutritionnel avec micro-transactions.
20. Application de fitness avec challenges sponsorisés payants.

## 5. **Événements & Loisirs**

21. Billetterie mobile pour événements locaux (paiement Stripe).
22. Appli de réservation de salles de sport.
23. SaaS pour organiser des conférences (tickets, badges, paiements).
24. Application de gestion de festivals avec tickets numériques.
25. Plateforme de cours de danse ou yoga (paiement par cours).

## 6. **Services professionnels**

26. Appli de gestion de rendez-vous (coiffeur, esthéticienne, etc.).
27. SaaS de gestion de freelances avec facturation automatique.
28. Application pour avocats/notaires avec réservation et paiement.
29. Plateforme de recrutement freelance avec paiement sécurisé.
30. SaaS RH pour gérer les heures et salaires des employés.

## 7. **Immobilier & Voyage**

31. Application de location de logements (type Airbnb simplifié).
32. Plateforme mobile de colocation avec abonnement premium.
33. SaaS de gestion locative (paiement du loyer par Stripe).
34. Application de réservation de voyages sur mesure.
35. Appli de covoiturage local avec micro-paiements.

## 8. **Créateurs & Contenu**

36. Application type OnlyFans (paiements par abonnement pour créateurs).
37. SaaS de newsletters payantes (via Stripe).
38. Application mobile pour photographes (vente de presets/photos).
39. Plateforme de podcasts avec abonnements premium.
40. Application de vidéos éducatives (paiement à la demande).

## 9. **IA & Automatisation**

41. Chatbot mobile premium (paiement par nombre de tokens).
42. SaaS d’analyse de texte (CV, contrats) avec paiement par usage.
43. Application IA pour générer des images/vidéos avec crédits payants.
44. Appli mobile de transcription audio avec crédits Stripe.
45. SaaS de résumé automatique de documents (abonnement mensuel).

## 10. **Divers innovants**

46. Application de livraison locale (épiceries, restaurants).
47. SaaS de gestion d’associations (adhésions payantes en ligne).
48. Appli mobile de paris/jeux trivia (crédits virtuels achetés par Stripe).
49. Plateforme de gestion de parking avec réservation et paiement.
50. Application de garde d’animaux (paiement à l’heure).


