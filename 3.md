# Justification technique des stacks recommandées (5 projets)

Ce document expose, pour chacun des cinq projets mobiles (Flutter), **pourquoi** la pile technologique proposée est la plus adaptée, **dans quels cas elle ne le serait pas**, et **quels compromis** sont assumés. Une **table comparative** synthétise ensuite les choix.



## 1) Sweatly — App de mise en forme & suivi

### Stack recommandée

* **Mobile** : Flutter (GoRouter, Riverpod, Hive pour le cache hors-ligne).
* **Backend** : **Firebase** — Auth, Firestore, Storage, Cloud Functions (Gen2), Cloud Messaging (FCM), Analytics/Crashlytics.

### Pourquoi cette stack

* **Modèle de données documentaire** : contenus “exercices/programmes”, “sessions d’entraînement”, “métriques journalières”, “badges” sont naturellement des documents avec accès par utilisateur. Firestore simplifie la lecture/écriture orientée documents et les règles d’accès par `uid`.
* **Engagement & notifications** : FCM et Remote Config sont intégrés nativement, sans serveur dédié.
* **Offline-first** : Firestore et Flutter + Hive couvrent le mode **hors-ligne** (séance en métro/salle) avec synchronisation différée.
* **Faible complexité relationnelle** : peu de jointures complexes, agrégations simples déléguées à Cloud Functions (streaks, totaux).
* **Time-to-market** : DX rapide, coût d’entrée faible, outillage complet (Analytics, Crashlytics, A/B via Remote Config).

### Quand ce choix serait moins pertinent

* Besoin de **rapports SQL complexes** (BI, croisements multi-dimensions) ou de **règles métier ACID** strictes → une base relationnelle (Postgres) serait plus indiquée.



## 2) Shiftly — Horaires, pointage, paie

### Stack recommandée

* **Mobile** : Flutter (geolocator, mobile\_scanner, local\_auth).
* **Backend** : **Supabase (Postgres)** comme **source of truth** (tables `users`, `companies`, `employees`, `shifts`, `time_entries`, `payslips`, politiques RLS, triggers, vues), **Edge Functions** pour logique serveur, **Storage** pour PDF/preuves; **Firestore** en **cache lecture** (prochains quarts, annonces) si besoin de réactivité très élevée; **FCM** pour notifications.

### Pourquoi cette stack

* **Intégrité et conformité paie** : le pointage et la paie exigent des **transactions**, des **contraintes**, des **agrégations fiables** et un **audit trail** — points forts d’un SGBDR (Postgres).
* **Rapports et exports** : feuilles de temps, totaux par période, états PDF/CSV → SQL et vues matérialisées simplifient la génération.
* **RLS** (Row Level Security) : contrôle fin d’accès (RH vs employé) côté base.
* **Réactivité utilisateur** : un **cache Firestore** côté lecture (quarts imminents) peut offrir des latences très faibles sans complexifier la gouvernance des données (la vérité reste Postgres).

### Quand ce choix serait moins pertinent

* Produit **très simple** sans paie/rapports complexes → Firestore pur suffirait.
* Équipe **sans** compétence SQL/DevOps → maintenir Postgres peut être coûteux en apprentissage.



## 3) PlaneEx — Agenda intelligent (conflits, météo, trafic)

### Stack recommandée

* **Mobile** : Flutter (table\_calendar, Riverpod, GoRouter, timezone).
* **Backend** : **Firebase** — Auth, Firestore (`events`, `users`), Cloud Functions (détection de conflits, planification de rappels), FCM; **APIs externes** : Google Maps Distance Matrix (ETA), météo (OpenWeather), jours fériés (Holiday API).

### Pourquoi cette stack

* **Événements = documents** : l’agenda personnel par utilisateur correspond à des collections “events” avec règles d’accès par `uid`.
* **Rappels & détection de conflits** : logique batch/événementielle efficace via Cloud Functions et planifications.
* **Intégrations** : les APIs externes (ETA, météo, fériés) s’imbriquent bien dans des Functions serverless.
* **Notifications** : FCM natif pour rappels.

### Quand ce choix serait moins pertinent

* Besoin de **synchronisation bidirectionnelle massive** avec suites bureautiques d’entreprise (Outlook/Exchange à grande échelle) ou de **reporting** avancé → une couche intermédiaire dédiée (service custom + DB relationnelle) peut devenir nécessaire.



## 4) DevLingo — Plateforme d’apprentissage (cours, quiz, progression)

### Stack recommandée

* **Mobile** : Flutter (GoRouter, Riverpod, Hive).
* **Backend** : **Firebase** — Auth, Firestore (cours, quiz, progression, leaderboard), Storage (médias), Cloud Functions (correction quiz, MAJ points/badges), FCM (rappels/streaks).

### Pourquoi cette stack

* **Flux CRUD documentaires** : cours/quiz/progression/points/badges sont des collections naturelles.
* **Correction & gamification** : Cloud Functions sécurise la correction côté serveur et l’attribution atomique de points/badges.
* **Notifications & révision** : FCM + Functions pour rappels et sessions de révision.
* **Coûts & vélocité** : aller-retour rapide entre maquettes, contenus et app mobile.

### Quand ce choix serait moins pertinent

* Exigences d’**analytique pédagogique avancée** (requêtes croisées lourdes, cohortes détaillées) → Postgres + entrepôt analytique (BigQuery, etc.) à envisager.


## 5) GazGo — Prix de l’essence communautaires (carte, filtres, leaderboard)

### Stack recommandée

* **Mobile** : Flutter (`google_maps_flutter`, geolocator).
* **Backend** : **Supabase (Postgres + PostGIS)** — stations (POINT 4326), prix, validations, vehicles, users; **Edge Functions** (soumission/validation prix, recommandation), **Storage** (photos), **Auth**; **Google Maps Platform** (Maps, Directions, Distance Matrix).

### Pourquoi cette stack

* **Géospatial sérieux** : requêtes par **rayon**, **tri par distance**, intersections géo → **PostGIS** est la norme, avec index GIST/BRIN et `ST_DWithin/Distance`.
* **Qualité/fiabilité des données** : workflow de **validation communautaire**, **agrégations** pour leaderboard/fiabilité → transactions SQL + triggers.
* **Itinéraires et coûts** : Distance Matrix + logique serveur permettent la **recommandation de station** en combinant distance, prix et capacité réservoir.

### Quand ce choix serait moins pertinent

* Si l’usage **géospatial** se limite à “afficher des pins proches sans tri/filtre avancé”, et si l’équipe veut une stack unique **NoSQL** → Firestore + geohash peut suffire (au prix d’un code plus artisanal côté requêtes).



# Tableau comparatif — Adéquation besoins ↔ technologies

| Dimension / Besoin                                       | Sweatly (Fitness)                    | Shiftly (Horaires/Paie)                                   | PlaneEx (Agenda)                           | DevLingo (Éducation)                   | GazGo (Essence)                                         |
| -------------------------------------------------------- | ------------------------------------ | --------------------------------------------------------- | ------------------------------------------ | -------------------------------------- | ------------------------------------------------------- |
| **Nature des données**                                   | Documents personnels, médias légers  | Données métier structurées, historiques, pièces justific. | Événements par utilisateur                 | Cours/quiz/progression documentaires   | Stations/prix géolocalisés, validations communautaires  |
| **Modèle recommandé**                                    | NoSQL documentaire (Firestore)       | SQL relationnel (Postgres) + RLS                          | NoSQL documentaire (Firestore)             | NoSQL documentaire (Firestore)         | SQL relationnel + **PostGIS** (géospatial)              |
| **Transactions/contraintes fortes**                      | Faible besoin                        | **Élevé** (pointage, paie)                                | Faible                                     | Faible                                 | **Élevé** (consensus/validations, MAJ prix)             |
| **Requêtes géospatiales avancées (rayon, tri distance)** | Non                                  | Non                                                       | Optionnel (ETA)                            | Non                                    | **Oui** (PostGIS)                                       |
| **Notifications & rappels**                              | FCM natif                            | FCM                                                       | FCM                                        | FCM                                    | FCM                                                     |
| **Offline-first**                                        | Oui (Firestore + Hive)               | Cache Firestore (optionnel) + client offline              | Oui (Firestore + local notifications)      | Oui (Firestore + Hive)                 | Partiel (cache client, priorités online pour cohérence) |
| **Rapports/exports (CSV/PDF)**                           | Légers (Fonctions)                   | **Forts** (SQL + vues + PDF)                              | Légers (Fonctions)                         | Légers                                 | Moyens (leaderboards/exports simples SQL)               |
| **Time-to-market**                                       | **Très rapide** (Firebase intégrée)  | Moyen (modélisation SQL + sécurité RLS)                   | **Rapide**                                 | **Très rapide**                        | Moyen (PostGIS + logique validations)                   |
| **Équipe peu familière SQL**                             | Adapté                               | Moins adapté (apprentissage requis)                       | Adapté                                     | Adapté                                 | Moins adapté (géospatial SQL requis)                    |
| **Scalabilité fonctionnelle (ajouts)**                   | Élevée (Functions, Remote Config)    | Élevée (SQL, triggers, Edge Functions)                    | Élevée                                     | Élevée                                 | Élevée (SQL/PostGIS, fonctions)                         |
| **Coût initial & outillage**                             | **Faible** (suite Firebase)          | Moyen (DB + CI + gouvernance)                             | **Faible**                                 | **Faible**                             | Moyen (DB gérée + Maps quotas)                          |
| **Risque principal**                                     | Analytics avancée limitée en natif   | Complexité de schéma et conformité paie                   | Couverture de cas calendaires complexes    | Analyse pédagogique croisée limitée    | Gouvernance des contributions et fraude                 |
| **Mitigation**                                           | Export analytique/BigQuery si besoin | Standards SQL, RLS, audits, tests d’intégration           | Fonction serveur dédiée, tests de conflits | Data pipeline secondaire (entreposage) | Modération, consensus, heuristiques, scoring, photos    |
| **Verdict**                                              | **Firebase**                         | **Postgres (Supabase) + cache Firestore**                 | **Firebase**                               | **Firebase**                           | **Postgres + PostGIS (Supabase)**                       |



# Pourquoi **Firebase** ici et pas là — et inversement

* **Pourquoi Firebase pour Sweatly / PlaneEx / DevLingo ?**

  * Données **centrées utilisateur**, structure **documentaire**, règles d’accès simples, **mode hors-ligne** et **notifications intégrées**.
  * Parcours CRUD simples, agrégations limitées, pas de transactions complexes.
  * **Coût/rapidité** imbattables pour un MVP mobile-first avec engagement (FCM).

* **Pourquoi pas Firebase pour Shiftly / GazGo ?**

  * **Shiftly** : paie/pointage = **intégrité forte**, exports réguliers, règles d’accès fines → **SQL + RLS**.
  * **GazGo** : carte + requêtes **géospatiales** (rayon, tri distance), **consensus/validations** → **PostGIS** pour performance et expressivité.
  * Ces deux domaines bénéficient de **transactions, contraintes, triggers, vues** difficiles à reproduire proprement en pur NoSQL.

* **Pourquoi Supabase/Postgres pour Shiftly / GazGo ?**

  * **Modèle relationnel** naturel, **transactions** et **intégrité** métier.
  * **PostGIS** pour GazGo.
  * **RLS** pour une sécurité fine côté données.
  * **Edge Functions** pour logique serveur proche des données et WebSocket/Realtime si nécessaire.

* **Pourquoi un mélange (Shiftly)** :

  * Garder **Postgres** comme **source de vérité** (comptabilité/paie), tout en offrant une **UX très réactive** via un **cache Firestore** pour la lecture “prochains quarts” et les annonces.
  * On bénéficie du **meilleur des deux mondes** sans dupliquer la logique critique.



# Risques, arbitrages et mesures de maîtrise

* **Vendor lock-in Firebase** : acceptable au regard du time-to-market et de l’intégration mobile; **mitigation** par une couche repository côté app et Functions modulaires exportables.
* **Compétences SQL/PostGIS** (Shiftly, GazGo) : montée en compétence nécessaire; **mitigation** via gabarits de schémas, migrations outillées (Sqitch/Prisma), tests d’intégration.
* **Coûts Google Maps** (GazGo, PlaneEx) : surveiller quotas; **mitigation** par cache, bounding boxes, progressif fallback (tri approximatif client si quota atteint).
* **Sécurité & conformité** :

  * Firebase : règles Firestore et vérifications serveur (Functions) pour écritures sensibles.
  * Supabase : RLS systématique, journaux d’audit, chiffrage au repos/transport, gouvernance accès.



## Conclusion

* **Firebase** est privilégié pour les apps à **données documentaires par utilisateur**, nécessitant **offline-first**, **notifications** et **livraison rapide** (Sweatly, PlaneEx, DevLingo).
* **Supabase/Postgres (+ PostGIS)** est requis lorsque la **cohérence transactionnelle**, le **reporting SQL** et/ou les **requêtes géospatiales** sont des exigences structurelles (Shiftly, GazGo).
* Le cas **hybride** (Shiftly) tire parti d’un **SGBDR** pour la vérité métier et d’un **cache NoSQL** pour l’UX temps réel, sans compromettre la conformité.

