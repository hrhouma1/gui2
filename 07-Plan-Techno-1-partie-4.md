# Document pédagogique – Technologies vulgarisées avec définitions techniques

## 1. Backend, base de données et serverless

**Vulgarisation**

* Le **backend**, c’est la **cuisine** d’un restaurant : les clients (app mobile) commandent, et la cuisine (backend) prépare les plats (données).
* La **base de données**, c’est le **garde-manger** : on y stocke les ingrédients (informations).
* Le **serverless**, ce sont des **cuisiniers appelés uniquement quand il y a une commande**, on ne les paye pas s’ils ne cuisinent pas.

**Définition technique**

> Le backend désigne la partie serveur d’une application qui gère la logique métier, la sécurité et les interactions avec la base de données. Le serverless est un modèle d’exécution dans lequel le fournisseur cloud exécute du code à la demande, sans gestion d’infrastructure par le développeur.

---

## 2. Firebase Auth

**Vulgarisation**
C’est le **portier** à l’entrée d’un club : il vérifie si tu as ton billet (mot de passe, compte Google ou Apple) avant de te laisser entrer.

**Définition technique**

> Firebase Authentication est un service géré qui fournit des méthodes sécurisées d’identification (email/mot de passe, fournisseurs OAuth comme Google, Apple, Facebook) et gère les sessions utilisateur.

---

## 3. Firestore (base NoSQL)

**Vulgarisation**
C’est une **bibliothèque de classeurs** : chaque utilisateur a son classeur avec ses infos. On peut ranger d’autres classeurs pour les cours, les évènements, etc.

**Définition technique**

> Cloud Firestore est une base de données NoSQL orientée documents qui stocke des collections de documents JSON, avec synchronisation temps réel et gestion hors ligne.

---

## 4. Règles de sécurité Firestore

**Vulgarisation**
C’est un **agent de sécurité** qui vérifie : “Est-ce bien ton dossier ? Si oui tu peux le lire ou l’écrire, sinon non.”

**Définition technique**

> Les règles Firestore sont un langage déclaratif permettant de définir les autorisations de lecture et d’écriture sur les collections/documents en fonction de l’authentification et du contenu de la requête.

---

## 5. Cloud Functions

**Vulgarisation**
C’est comme un **cuisinier spécialisé** qu’on appelle uniquement quand on a besoin de lui (corriger un quiz, calculer des points, envoyer une notification).

**Définition technique**

> Firebase Cloud Functions est un service serverless qui exécute du code backend en réponse à des événements (HTTP, écriture dans Firestore, messages, etc.), avec facturation à l’usage.

---

## 6. FCM – Firebase Cloud Messaging

**Vulgarisation**
C’est un **haut-parleur** qui envoie un petit message à ton téléphone, même si l’application est fermée. Exemple : “Ta séance commence dans 30 minutes.”

**Définition technique**

> Firebase Cloud Messaging (FCM) est un service d’envoi de notifications push multi-plateforme (Android, iOS, Web), avec ciblage par utilisateur, par groupe ou par topic.

---

## 7. Supabase / Postgres

**Vulgarisation**
C’est comme un **cahier à colonnes (tableur)** où chaque ligne correspond à un utilisateur, un quart de travail ou une station d’essence. Parfait pour faire des totaux fiables (combien d’heures, combien payer).

**Définition technique**

> Supabase est une plateforme open source proposant une base de données Postgres managée avec API REST et WebSocket automatiques. PostgreSQL est un SGBDR relationnel ACID supportant SQL avancé, transactions, et extensions comme PostGIS.

---

## 8. RLS – Row Level Security

**Vulgarisation**
C’est comme si, dans un grand tableau Excel, chaque étudiant ne voyait **que sa propre ligne**.

**Définition technique**

> Row Level Security (RLS) est une fonctionnalité de PostgreSQL permettant de définir des politiques de filtrage des lignes visibles/modifiables pour chaque utilisateur en fonction de son identité.

---

## 9. PostGIS

**Vulgarisation**
C’est comme ajouter une **règle de géographie** à ton tableau : tu peux demander “Quelles stations sont à moins de 3 km de moi ?” et obtenir une liste triée par distance.

**Définition technique**

> PostGIS est une extension de PostgreSQL qui ajoute des types et fonctions géospatiales (points, polygones, calculs de distances, intersections), optimisées par des index GIST/BRIN.

---

## 10. Notifications push

**Vulgarisation**
Ce sont les **petits messages** qui apparaissent sur ton téléphone, même quand l’app est fermée. Exemple : “Ton cours commence dans 10 minutes.”

**Définition technique**

> Une notification push est un message court envoyé depuis un serveur vers un appareil mobile ou navigateur, via un service comme FCM ou APNs, indépendamment de l’état de l’application.

---

## 11. Offline-first

**Vulgarisation**
C’est comme **écrire dans un cahier de brouillon** quand Internet est coupé, puis recopier au propre dans le cahier principal quand la connexion revient.

**Définition technique**

> Offline-first est une stratégie de conception d’application où les données sont d’abord stockées localement (cache, base embarquée), puis synchronisées avec le backend quand la connexion est rétablie.

---

## 12. Edge Functions (Supabase)

**Vulgarisation**
C’est comme un **guichet rapide** juste à côté du cahier (base de données) : au lieu d’aller loin chercher un serveur, on exécute le code immédiatement à côté des données.

**Définition technique**

> Supabase Edge Functions est un service serverless qui exécute du code TypeScript/JavaScript proche des bases de données Postgres hébergées, avec faible latence et sécurité intégrée.

---

## 13. Google Maps, Directions, Distance Matrix

**Vulgarisation**

* **Maps** : c’est la **carte** avec les points.
* **Directions** : c’est le **GPS** qui propose un chemin.
* **Distance Matrix** : c’est le **tableau de trajets** qui dit combien de temps pour aller à plusieurs endroits.

**Définition technique**

> Google Maps Platform regroupe des APIs de cartographie. L’API Directions fournit des itinéraires, et Distance Matrix retourne les durées et distances entre plusieurs origines et destinations.


