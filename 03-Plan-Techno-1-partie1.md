

# 1) Sweatly — Description produit (exhaustive)

**Vision.** Aider l’utilisateur à s’entraîner régulièrement avec des séances adaptées, un suivi clair de sa progression, et un petit côté social (partage des réussites) — le tout **mobile-first, offline-first**.

**Cibles.** Débutants à intermédiaires; coachs qui publient des circuits; étudiants qui veulent une routine simple.

**Fonctionnalités cœur (MVP)**

* **Onboarding & Profil** : objectifs (perte de poids, force, mobilité), niveau, matériel disponible.
* **Bibliothèque d’exercices** : fiches normalisées (gif/image, muscles ciblés, consignes, variantes, contre-indications).
* **Séances & Programmes** : circuits (ex: 3×10, AMRAP, EMOM), timers, repos, alternatives par niveau.
* **Progression** : séries, répétitions, charges, temps; graphiques d’évolution par exercice/séance.
* **Favoris & Historique** : enregistrer des séances, rejouer un “workout du jour”.
* **Notifications** : rappels, relances de streak, reprise après inactivité.
* **Partage discret** : capture/résumé “workout complété” (sans données sensibles), optionnel.
* **Mode hors-ligne** : exécuter une séance sans réseau; synchro à la reconnexion.

**Fonctionnalités “+1” (post-MVP)**

* **Recommandations** (séances adaptées aux dernières perfs & fatigue auto-déclarée).
* **Plan hebdo automatique** (alternance haut/bas du corps, intensité).
* **Badges & Gamification** (streaks, records personnels).
* **Espace coach** (console web pour créer programmes).

---

## Parcours utilisateurs (Happy paths)

1. **Nouveau·elle** : crée un compte → répond à 4–6 questions d’onboarding → reçoit un **Plan de 7 jours** prêt à lancer → démarre “Séance 01”.
2. **Retour après pause** : l’app détecte >7 jours d’inactivité → propose une **séance de reprise** + rappel progressif des charges.
3. **Routine régulière** : ouvre l’app → “Workout du jour” en 1 tap → timer intégré → saisie ultra-rapide des perfs → feedback immédiat.
4. **Offline** : commute métro/salle → lance la séance en cache (médias + consignes locales) → synchro quand réseau revient.

---

## Modèle de données (Firestore + Storage)

Collections principales :

* `users/{uid}`
  `displayName, goal, level, equipment[], height, weight, lastActiveAt, streak, badges[]`
* `exercises/{exerciseId}`
  `title, muscles[], equipment[], level, mediaUrls{image, gif}, cues[], contraindications[]`
* `workouts/{workoutId}`
  `title, blocks[]` (chaque block = liste d’exercices avec schéma: sets/reps/timer/tempo/alternatives), `level`, `tags[]`
* `programs/{programId}`
  `title, weeks[], level, targetGoal, author`
* `sessions/{sessionId}` (journal)
  `userId, workoutId, performedAt, entries[]` (par exercice : `sets, reps/repsDone, weight, time, rpe`)
* `favorites/{userId}/items/{workoutId}`
* `metrics_daily/{userId}_{yyyyMMdd}`
  `completedCount, volumeTotalKg, durationMin, streakDay`

**Stockage (Cloud Storage)**

* `/exercises/{id}/media/*` (images/gifs courtes optimisées, webp/mp4)
* `/users/{uid}/share/*` (images de partage générées)

**Règles clé**

* Lecture publique **limitée** aux exercices/programmes **publiés**; tout le reste scoped par `uid`.
* Écritures critiques (streaks, badges, recalculs) via **Cloud Functions**.

---

## Pile technique (mobile & backend)

### Mobile (Flutter)

* **Architecture** :

  * State management : **Riverpod** (ou Bloc)
  * Navigation : **GoRouter**
  * Caching local : **Hive** (sessions à venir, fiches d’exercices consultées, dernier “workout du jour”)
  * Media : `cached_network_image`, pré-fetch des médias du prochain workout
* **UX séance** :

  * Timers intégrés (intra-exercice & repos)
  * Saisie perfs **ultra-rapide** (tap sliders, suggestions automatiques à partir de la dernière séance)
  * Haptics & TTS (consignes courtes) optionnels
* **Packages Flutter (MVP)**
  `firebase_core, firebase_auth, cloud_firestore, firebase_storage, cloud_functions, firebase_messaging, flutter_riverpod, go_router, hive, hive_flutter, cached_network_image, intl`

### Backend (Firebase)

* **Auth** : email/mot de passe + providers (Apple/Google)
* **Firestore** : documents/collections pour contenu & progression
* **Cloud Functions (Gen2, Node 20)** :

  * `onSessionWrite` : agrège volume/temps, met à jour `metrics_daily` & `streak`
  * `awardBadges` : décerne badges selon règles (ex: 7 jours d’affilée)
  * `recommendNextWorkout` : simple heuristique initiale (niveau + muscles sous-travaillés)
  * `generateShareImage` : génère image sociale (Cloud Run/Functions + canvas node) → URL signée
* **Cloud Messaging (FCM)** : rappels personnalisés (fenêtre préférée), relance post-inactivité
* **Cloud Storage** : médias optimisés; gestion d’URLs signées si contenus premium/privés
* **Analytics** : Firebase Analytics + events customs (`workout_started`, `set_logged`, `workout_completed`, `streak_incremented`)

### Sécurité & confidentialité

* Règles Firestore : accès **par utilisateur** à ses journaux; contenu public en lecture seule (exercices publiés).
* **Aucune donnée santé sensible** (type médical) sans consentement explicite.
* **Export/effacement compte** (GDPR-like) : endpoint/Function pour récupérer/supprimer données.

### Performances & offline

* Pré-chargement **J+1** : workout du lendemain + médias.
* Mise en cache Hive des fiches d’exercices récemment ouvertes.
* Stratégie **write-behind** : journal écrit localement, puis synchro Firestore (retry/backoff).
* Image & gif : variantes en 3 tailles; choisir dynamiquement selon réseau.

### Observabilité & QA

* Crash reporting : Firebase Crashlytics
* Traces essentielles (durée affichage écran séance, latence lecture médias)
* Feature flags via `remote_config` (activer badges, recommandations)

### CI/CD

* GitHub Actions ou Codemagic : build Android App Bundle + iOS (TestFlight)
* **Flavors** : dev / staging / prod (configs Firebase distinctes)
* Tests unitaires (logic perfs/reco), golden tests UI pour écrans clés

---

## Recommandation : quoi faire **en premier** (ordre priorisé)

### Semaine 0 — Préparation

1. **Création des projets Firebase** (dev, staging, prod) + activation Auth/Firestore/Storage/Functions/FCM.
2. **Skeleton Flutter** (flavors, GoRouter, Riverpod, thème, design tokens).
3. **Schéma Firestore** minimal + règles lecture publique des `exercises` publiés; tout le reste privé.

### Sprint 1 — **MVP séance hors-ligne**

1. **Onboarding court** (objectif, niveau, matériel) → stockage `users/{uid}`.
2. **Écran “Workout du jour”** (mock statique d’abord) avec **timer** et **saisie perfs**.
3. **Cache local (Hive)** pour le workout courant + médias pré-téléchargés.
4. **Journal simple** : écrire `sessions/{id}` (local→sync).
   **Sortie sprint** : on peut faire une séance **sans réseau**, la terminer, et la synchro se fait ensuite.

### Sprint 2 — **Bibliothèque & progression**

1. **Liste/fiche exercices** (Firestore + images Storage).
2. **Workouts dynamiques** depuis Firestore (fini le mock).
3. **Fonction Cloud `onSessionWrite`** : agrégation volume/temps + mise à jour `metrics_daily`, `streak`.
4. **Historique & favoris** (rejouer séance).

### Sprint 3 — **Notifications & qualité**

1. **FCM** : rappels personnalisés (créneau préféré).
2. **Relance post-inactivité** (J+3/J+7).
3. **Crashlytics + Analytics** : instrumentation des événements clés.
4. **Optim médias** : images/gifs en plusieurs tailles; pré-fetch J+1.

### Sprint 4 — **Recommandations & partage**

1. **`awardBadges`** (badges basiques : 3 séances/semaine, 7 jours d’affilée).
2. **`recommendNextWorkout`** (heuristique niveau + alternance muscles).
3. **Image de partage** (Function/Run) avec stats de la dernière séance.

> **Point d’attention** : ne complexifie pas la reco dès le départ. Commence par une heuristique transparente. Les modèles ML pourront venir **après** avoir collecté des journaux suffisants.

---

## Critères d’acceptation (MVP)

* L’utilisateur peut **créer un compte**, passer l’onboarding, lancer et terminer **une séance**.
* La séance fonctionne **sans réseau** (timer + saisie), puis **se synchronise** correctement.
* Son **historique** affiche la séance dans la journée, avec **volume total** et **durée**.
* Il reçoit un **rappel** le lendemain dans son créneau préféré.
* Aucune donnée autre que les exercices publiés n’est visible sans authentification.



<br/>

# Planification agile omplète 

# Cadence & rituels

* **Itération**: 3 semaines.
* **Rituels**: Planning (2 h), Daily (15 min, async si possible), Revue (30–45 min démo), Rétro (45 min).
* **Artefacts**: Product Backlog (épopées → US), Sprint Backlog, Burndown, Definition of Ready (DoR) & Definition of Done (DoD).

---

# Sprint 0 — Préparation (17–21 sept. 2025)

**Objectif**: Environnements prêts, pipelines en place, bases de code créées.

* **Comptes & projets**

  * Firebase (dev/staging/prod) pour Sweatly & PlaneEx (Auth, Firestore, Storage, Messaging, Functions).
  * Supabase (Postgres + PostGIS) pour Shiftly (ou backend Postgres géré), Storage, Edge Functions.
  * Clés APIs (Google Maps, Distance Matrix; OpenWeather; Holiday API).
* **Code & CI/CD**

  * Monorepo ou 3 repos (Flutter) + **flavors** (dev/staging/prod).
  * GitHub Actions/Codemagic: build Android (AAB) + iOS (TestFlight).
  * Qualité: lints, format, tests unitaires skeleton, Crashlytics/Analytics intégrés.
* **Design Systèmes**

  * Thème Flutter commun (typographie, couleurs), routing GoRouter, state Riverpod.
  * Charte d’API (naming, versioning), conventions Firestore/SQL.
* **DoD Sprint 0**

  * Builds CI verts pour les 3 apps en **dev**.
  * Secrets gérés (dotenv/Actions secrets).
  * Docs README “comment builder/lancer” à jour.

---

# Sprint 1 — 22 sept. → 12 oct. 2025

**Objectif global**: Onboarding & Auth fonctionnels, premiers écrans utiles.

### Sweatly (fitness — Firebase)

* **US clés**:

  * Création/connexion compte (email + provider).
  * Onboarding court (objectif, niveau, matériel).
  * Écran Accueil avec “Workout du jour” (contenu statique initial).
* **Tech**: Firebase Auth; Firestore `users`; Storage prêt; FCM configuré (topic test).
* **DoD**: Un utilisateur peut s’inscrire, compléter l’onboarding, voir un workout placeholder.

### Shiftly (horaires/pointage — Postgres/Supabase + cache Firebase)

* **US clés**:

  * Auth (email/Google) centralisée (choisir **Supabase Auth** ou **Firebase Auth**, un seul système).
  * CRUD minimal **shifts** côté Postgres; affichage “Mes 5 prochains quarts” mobile.
* **Tech**: Schéma Postgres: `users`, `shifts`; index temporels; 1 Edge Function `get_next_shifts(user_id)`; (option) Firestore cache lecture “prochains quarts”.
* **DoD**: Un utilisateur connecté voit ses quarts à venir issus de Postgres.

### PlaneEx (agenda — Firebase + APIs)

* **US clés**:

  * Auth + création d’un **évènement** (titre, date, lieu).
  * Vue Calendrier (liste ou monthly grid) en lecture.
* **Tech**: Firestore `events`; structure `users/{uid}/events`; modèle de données simple; timezone gérée.
* **DoD**: Créer/éditer/supprimer un évènement et l’afficher dans la vue calendrier.

**Livrables Sprint 1**: APK/IPA internes installables, 1 démo par app.

---

# Sprint 2 — 13 oct. → 2 nov. 2025

**Objectif global**: Contenu réel et 1 parcours “cœur de métier” utilisable par app.

### Sweatly

* **US clés**:

  * Bibliothèque d’exercices (liste + fiche) chargée depuis Firestore.
  * Workout du jour réel (depuis `workouts`) avec timer local.
* **Tech**: Firestore `exercises`, `workouts`; Storage images/gifs (3 tailles); cache Hive pour “J+1”.
* **DoD**: Lancer et exécuter un workout complet (timer, navigation d’exercices).

### Shiftly

* **US clés**:

  * Création/modif quart (rôle manager) + notifications assignation.
  * Écran “Mon quart du jour” (détails, notes).
* **Tech**: Tables `companies`, `employees`, `shifts`; politiques RLS/ACL; FCM push assignations.
* **DoD**: Un manager assigne un quart; l’employé reçoit une notif et voit le détail.

### PlaneEx

* **US clés**:

  * Modèle récurrence simple (hebdomadaire).
  * Rappels locaux/push à N minutes/heures.
* **Tech**: Functions planification rappels; FCM; champs `reminderOffset`.
* **DoD**: L’utilisateur reçoit un rappel avant un évènement; récurrence hebdo opérationnelle.

**Livrables Sprint 2**: flux clé **utilisable** pour chaque app (workout, assignation de quart, rappel d’évènement).

---

# Sprint 3 — 3 nov. → 23 nov. 2025

**Objectif global**: Historique, conflits/logique serveur, début d’intelligence.

### Sweatly

* **US clés**:

  * Journal de séance (sets/reps/poids/temps) + **historique** et **statistiques** (volume/durée).
  * Streak automatique et badges basiques (1er workout, 3 séances/semaine).
* **Tech**: Function `onSessionWrite` (agrégats `metrics_daily`, streak); vues historiques; Analytics événements.
* **DoD**: Séance enregistrée ⇒ stats & streak mis à jour automatiquement; badges visibles.

### Shiftly

* **US clés**:

  * **Pointage**: GPS/QR ou photo horodatée (stockée) + règles de validation.
  * **Rapport** d’heures par période (CSV/PDF simple).
* **Tech**: Table `time_entries`; contraintes anti-fraude; génération PDF (server); stockage pièces jointes.
* **DoD**: Un employé pointe et son entrée est auditée; un rapport hebdo est exportable.

### PlaneEx

* **US clés**:

  * **Détection de conflits** (chevauchements) + propositions alternatives simples.
  * Intégration **Distance Matrix**: ETA proposé (option départ anticipé).
* **Tech**: Function `detect_conflicts(uid, date_range)`; intégration Maps (quota, backoff).
* **DoD**: Un conflit est signalé et une alternative est suggérée; ETA affichée sur évènement avec lieu.

**Livrables Sprint 3**: valeur “pro” visible (statistiques, rapports d’heures, gestion de conflits/ETA).

---

# Sprint 4 — 24 nov. → 14 déc. 2025 (revue finale 15 déc.)

**Objectif global**: Engagement, robustesse, finitions et préparation démo.

### Sweatly

* **US clés**:

  * Recommandations heuristiques (proposer la prochaine séance).
  * Partage d’un résumé de séance (image générée).
* **Tech**: Function `recommendNextWorkout`; Function de génération image partage (Canvas/Cloud Run).
* **DoD**: Après un workout, une prochaine séance est suggérée; image partage disponible.

### Shiftly

* **US clés**:

  * Notifications “rappel de quart” et “changement de quart”.
  * Règles de confidentialité/consentement (présence, localisation) claires dans l’app.
* **Tech**: FCM scénarisé; politiques RLS affinées; pages légales in-app.
* **DoD**: Notifications fiables; consentement géré; check final sécurité.

### PlaneEx

* **US clés**:

  * Météo/jours fériés dans l’agenda; vue hebdo améliorée.
  * Export ICS/PDF simple pour partage.
* **Tech**: APIs OpenWeather/Holidays; Function export ICS; rendu PDF server.
* **DoD**: Évènements enrichis météo/feriés; export opérationnel.

**Livrables Sprint 4**: APK/IPA de démo, notes de version, scripts de démo, backlog “post-demo”.

---

## Definition of Ready (DoR) & Definition of Done (DoD)

* **DoR**: US a une valeur métier claire, critères d’acceptation, maquettes ou wire simples, dépendances identifiées.
* **DoD**: code revu, tests passants, analytics/crashlytics instrumentés si pertinent, docs courtes, déployé en staging, démo réalisable.

---

## Backlog priorisé (extraits)

* **Sweatly**: Auth → Onboarding → Exercices/Workouts → Timer → Journal/Stats → Streak/Badges → Notifications → Reco/Partage.
* **Shiftly**: Auth → Shifts CRUD/Assignation → Vue “Prochains quarts” → Pointage → Rapports → Notifications → Consentement.
* **PlaneEx**: Auth → Events CRUD/Calendrier → Rappels → Conflits → ETA → Météo/Fériés → Export ICS/PDF.

---

## Indicateurs (par sprint)

* **Livrables**: 3 APK/IPA installables + changelog.
* **Qualité**: taux de crash < 1 %, TTI écran principal < 2 s, transactions clés < 500 ms server.
* **Engagement**: Sweatly — % séances terminées; Shiftly — % pointages valides; PlaneEx — % évènements avec rappel configuré.

---

## Risques & parades

* **Empilement techno**: limiter à Firebase (S/Pl) + Supabase (Sh) + APIs Google; pas d’options exotiques en sprint.
* **Quota APIs Maps**: clé restreinte (domaines/paquet), monitoring quotas, fallback “estimation simple”.
* **iOS provisioning**: créer profils/cerfs dès Sprint 0; TestFlight actif Sprint 1.
* **Dérive périmètre**: une seule “feature star” par app et par sprint.

---

### Ce que tu fais **dès aujourd’hui** (ordre)

1. Clés & projets (Firebase/Supabase/Google APIs) + 3 repos Flutter avec flavors.
2. CI/CD dev en vert pour les 3 apps (Android d’abord).
3. Sprint 1: implémenter **Auth** + 1 écran d’accueil par app.
4. Démo interne (12 oct.), ajuster backlog Sprint 2 selon retours.





<br/>
<br/>


# Feedback – Projet EcoMap

## 1. Points forts

* **Pertinence du thème** : le projet répond à un enjeu actuel (mobilité durable, électrification des transports, vélo-partage). C’est un domaine **innovant et à fort impact social**, qui peut séduire un jury gouvernemental.
* **Clarté fonctionnelle** : la description des cas d’utilisation (EV, BIXI, filtrage, favoris, statistiques CO₂) est bien définie. L’équipe a pris soin de penser à l’**expérience utilisateur** et aux bénéfices écologiques.
* **Multilinguisme** : interface prévue en français et en anglais, un bon point pour l’accessibilité au Québec.
* **Utilisation de données ouvertes** : réutiliser les API de la Ville de Montréal et de BIXI est une approche réaliste et pragmatique.
* **Valeur ajoutée** : les statistiques écologiques (CO₂ évité) apportent une dimension motivationnelle, qui va au-delà du simple repérage de bornes.

---

## 2. Améliorations possibles

* **Priorisation Agile** : le projet couvre beaucoup de fonctionnalités (carte, météo, favoris, stats). Il faudra découper en **MVP** :

  * Sprint 1 : Authentification + carte avec bornes EV et BIXI.
  * Sprint 2 : Filtrage + itinéraires.
  * Sprint 3 : Notifications de disponibilité.
  * Sprint 4 : Météo + statistiques CO₂.
* **Données en temps réel** : les APIs publiques (ex. BIXI) doivent être testées pour vérifier la fréquence de mise à jour et la fiabilité des données.
* **Performances et consommation** : afficher plusieurs centaines de bornes peut être lourd sur mobile. Prévoir un système de **clustering** ou de **chargement par zones**.
* **Notifications** : penser aux conditions de batterie des utilisateurs. Une notification “borne disponible” peut vite devenir intrusive si mal calibrée.
* **Accessibilité et inclusion** : prévoir que l’application soit utilisable par des personnes non-voyantes (support VoiceOver / TalkBack).
* **Backend/API** : l’API définie est claire mais encore générique. Il faudrait choisir la **stack technique** (par ex. Firebase vs Supabase/Postgres). Pour ce projet, **Supabase/Postgres + PostGIS** semble idéal car il y a beaucoup de calculs **géographiques** (bornes proches, filtrage par distance).

---

## 3. Recommandation technologique

* **Mobile** : Flutter (multiplateforme, rapide pour prototypage).
* **Backend** :

  * **Supabase/Postgres + PostGIS** : parfait pour la gestion géospatiale (bornes, distance, filtrage).
  * **Auth Supabase** : gestion des utilisateurs, JWT simple.
  * **Edge Functions** : gestion des notifications, calculs CO₂.
* **APIs externes** : Ville de Montréal (bornes EV), BIXI (temps réel), OpenWeather (météo).
* **Notifications** : Firebase Cloud Messaging (push).

---

## 4. Risques et points de vigilance

* **Dépendance aux données ouvertes** : si l’API change ou tombe en panne, l’application perd une partie de sa valeur. Prévoir un **cache local**.
* **Exactitude des stats CO₂** : il faudra justifier la formule de calcul (par ex. 1 km vélo = x grammes CO₂ évités).
* **Complexité du backend** : combiner météo, bornes EV, BIXI et stats peut surcharger la première version. Il est conseillé de livrer un **MVP simple** et d’ajouter les extras par itérations.

---

## 5. Évaluation pédagogique

* **Bonne maîtrise** des concepts vus en cours : géolocalisation, cartes, notifications, APIs.
* **Travail complet** : diagrammes, récits utilisateurs, API backend.
* **Pour aller plus loin** : démontrer la **justification des choix techniques** (par exemple : pourquoi Postgres/PostGIS plutôt que Firestore). Cela donnera plus de profondeur à la présentation.





<br/>
<br/>


# 1) Mobile (Flutter) — App first class

## 1.1 Packages principaux

* **Carte & géoloc** : `google_maps_flutter`, `geolocator` (position en temps réel; centrage initial sur Montréal).&#x20;
* **Clustering & performance** : `google_maps_cluster_manager` (regrouper des centaines de bornes), `flutter_cache_manager` (cache tuiles/images).
* **Routage & état** : `go_router`, `flutter_riverpod` (ou Bloc).
* **HTTP & JSON** : `dio` (timeouts/retry), `freezed` + `json_serializable` (modèles sûrs).
* **Localisation & i18n** : `flutter_localizations`, `intl` (FR/EN comme prévu).&#x20;
* **Notifications** : `firebase_messaging` (push), `flutter_local_notifications` (rappels locaux – ex. borne redevenue dispo).&#x20;
* **Offline** : `hive`/`isar` pour cache persistant (favoris, dernières bornes, météo récente).
* **Accessibilité** : `flutter_screenutil` (tailles adaptatives), support TalkBack/VoiceOver.

## 1.2 Architecture app

* **Clean Architecture** (layers `data`/`domain`/`presentation`), Repository pattern.
* **Feature toggles** via Remote Config (activer météo, CO₂).
* **Sécurité client** : aucune clé sensible en dur; configs par variables d’environnement et/ou RC.

## 1.3 UX clés

* **Recherche** par adresse (barre de recherche → appel Geocoding), **itinéraire** via deep-link Google/Apple Maps.&#x20;
* **Filtres** (type: EV vs BIXI; disponibilité; rayon), **fiche** borne (adresse, ports, type, statut, distance).&#x20;
* **Favoris** + **bilingue** FR/EN. &#x20;

---

# 2) Backend — Supabase (Postgres + PostGIS) + Functions

> Pourquoi **Postgres + PostGIS** ? Les besoins “**bornes à proximité**”, “**rayon**”, “**tri par distance**” et “**filtres**” sont natifs et performants avec PostGIS (index GIST + `ST_DWithin/Distance`). Votre API côté serveur prévoit déjà des routes “à proximité” et des filtres, ce qui cadre parfaitement avec du SQL géospatial. &#x20;

## 2.1 Services

* **DB managée** : Supabase (Postgres 15 + **PostGIS**).
* **Auth** : Supabase Auth (email/password; OAuth optionnel).
* **Stockage** : Supabase Storage (icônes custom, éventuelles images).
* **Edge Functions** (Deno/TS) : façade d’agrégation API publiques (BIXI, Ville de Montréal), webhooks notifications.
* **Realtime** (optionnel) : canal de “disponibilité” si une source temps réel fiable est accessible.

## 2.2 Schéma SQL (proposé)

```
users(id uuid pk, email text unique, display_name text, created_at timestamptz)
ecospots(                    -- bornes EV + stations BIXI
  id bigserial pk,
  type text check (type in ('ev','bixi')),
  name text,
  provider text,             -- 'VilleMTL', 'BIXI'
  address text,
  location geography(Point,4326),
  capabilities jsonb,        -- ex: {level: 2, ports: 6} pour EV ; {docks: 35} pour BIXI
  is_free boolean,           -- gratuité si fourni par la source
  status text,               -- 'available' | 'busy' | 'unknown'
  last_seen_at timestamptz,
  metadata jsonb             -- bruts de l’API source
)
favorites(                   -- favoris par utilisateur
  user_id uuid references users(id) on delete cascade,
  ecospot_id bigint references ecospots(id) on delete cascade,
  added_at timestamptz default now(),
  primary key(user_id, ecospot_id)
)
eco_actions(                 -- actions de mobilité verte pour stats CO₂
  id bigserial pk,
  user_id uuid references users(id) on delete cascade,
  kind text check (kind in ('bixi_km','ev_charge_kwh','walk_km','transit_trip')),
  value numeric not null,    -- nombre de km, kWh, etc.
  at timestamptz default now()
)
user_stats_daily(            -- matérialisation journalière
  user_id uuid,
  day date,
  km numeric default 0,
  co2_saved_g numeric default 0,
  primary key(user_id, day)
)
```

### Index & géospatial

```sql
CREATE INDEX ecospots_location_gix ON ecospots USING GIST (location);
-- Requête type: spots dans un rayon
-- SELECT ... WHERE ST_DWithin(location, ST_MakePoint($lon,$lat)::geography, $radius_m)
-- ORDER BY ST_Distance(location, ST_MakePoint($lon,$lat)::geography)
```

### RLS (Row-Level Security)

* `favorites`, `eco_actions`, `user_stats_daily` : **RLS ON**.

  * **Policy SELECT/INSERT** : `user_id = auth.uid()` (chaque usager ne voit que ses données).
* `ecospots` : **lecture publique** (read-only) si données ouvertes; **INSERT/UPDATE** limité aux **ETL/Functions** signées.

## 2.3 Pipelines & ETL

* **BIXI** : Edge Function `etl_bixi` (cron) → fetch temps réel → upsert `ecospots` (type `bixi`, `status`, `last_seen_at`).&#x20;
* **Ville de Montréal (bornes EV)** : Edge Function `etl_ev` (cron) → normalise capacités/ports/type.&#x20;
* **Météo** : `weather_cache` table (clé lat/lon/heure) pour limiter les appels OpenWeather; servi à l’app.&#x20;

## 2.4 API (Gateway)

> Vos méthodes d’API proposées s’alignent très bien sur cette façade : “/bornes”, “/à-proximité”, “/favorites”, “/stats”… &#x20;

* `GET /ecospots?type=&lat=&lng=&radius=&free=&status=` → requête PostGIS paginée.
* `GET /ecospots/{id}` → détail.
* `GET /ecospots/nearby?lat=&lng=&radius=` → **ST\_DWithin** + tri distance.
* `POST /favorites` / `GET /favorites` → gérés par RLS (uid).
* `POST /actions` → enregistre action verte; **trigger** met à jour `user_stats_daily`.
* `GET /stats?period=day|week|month` → somme CO₂ évité.

## 2.5 Logique serveur (triggers & jobs)

* **Trigger** `on_eco_action_insert` → calcule `co2_saved_g` par type (coef configurable) → upsert dans `user_stats_daily`.
* **Job** quotidien → recalcul consolidé (sécurité).
* **Webhook** “spot re-available” → si une station passe `busy→available` et qu’un utilisateur l’a en favori dans un rayon, envoi d’une **notification** (via FCM topic/device).

---

# 3) Intégrations externes

## 3.1 Cartographie & Itinéraires

* **Google Maps SDK** (mobile) pour affichage des points + **Directions** pour itinéraire depuis un point (deep-link si vous préférez l’app Maps).&#x20;
* **Distance Matrix** (option) si besoin de temps vers **plusieurs** spots pour classer les plus rapides.

## 3.2 Données sources

* **BIXI** : API temps réel officielle (stations, disponibilité docks/électrifiées).&#x20;
* **Ville de Montréal** : données ouvertes bornes de recharge EV.&#x20;
* **OpenWeather** : météo actuelle + prévisions (affichage + alertes).&#x20;

---

# 4) Sécurité, conformité, clés

* **RLS** stricte sur données personnelles (favoris, actions, stats).
* **Lecture publique** possible sur `ecospots` (open data).
* **Clés API** : restreindre clés Google (SHA-1 iOS/Android; si passerelle HTTP, restreindre IP); jamais en clair côté app.
* **Auth** : JWT Supabase sur l’app; rotation de refresh tokens standard.

---

# 5) Observabilité, qualité, CI/CD

* **Logs & traces** : Supabase logs + Edge Functions console; **Sentry** (app) + Crashlytics.
* **Analytics** : événements clés (`map_loaded`, `filter_changed`, `spot_tapped`, `route_opened`).
* **CI/CD** : GitHub Actions/Codemagic → build Android (AAB) + iOS (TestFlight) par branche; **flavors** dev/staging/prod.
* **Tests** :

  * Unitaires (mappers, repos, calcul CO₂)
  * Golden tests UI (fiche spot, liste filtrée)
  * Intégration (requêtes PostGIS sur dataset seed)

---

# 6) Performance et UX à grande échelle

* **Clustering** cartes (éviter des centaines de marqueurs simultanés).
* **Chargement par fenêtre spatiale** (bounds de la carte → requête serveur limitée).
* **Pagination** côté API (`limit/offset` ou `id > last_id`) pour points d’intérêt.
* **Cache** : météo, réponses “nearby”, favoris (TTL court).
* **Accessibilité** : contraste, labels TalkBack, tailles dynamiques.

---

# 7) Plan d’exécution priorisé (3 sprints de 3 semaines)

* **Sprint 1 — Fondations** : Auth + carte + **GET /ecospots/nearby** (PostGIS) + FR/EN + favoris (local).
* **Sprint 2 — Données réelles** : ETL BIXI + Ville MTL (cron) + filtres (type/rayon) + favoris (serveur/RLS) + itinéraire. &#x20;
* **Sprint 3 — Valeur ajoutée** : météo + notifications “re-available” + stats CO₂ (actions + daily stats) + optimisation cluster.  &#x20;

---

# 8) Pourquoi cette stack (résumé)

* **Postgres + PostGIS (Supabase)** pour le **géospatial** rapide et exact (rayon/tri distance), les **filtres** et l’**ETL** open data; **RLS** pour la protection des données utilisateur. Vos routes “à proximité”, “favoris”, “stats” cadrent parfaitement avec ce modèle. &#x20;
* **Flutter** simplifie la livraison **mobile-first** avec carte, filtres, bilingue, et notifications locales/push comme décrit.  &#x20;
* **Edge Functions** pour agréger **BIXI / Ville MTL / Météo** côté serveur, éviter les CORS/clé exposée, et déclencher les **notifications**. &#x20;


