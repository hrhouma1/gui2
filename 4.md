
# 1) Notions générales

## 1.1 Backend, base de données, serverless

* **Backend** : la partie “cachée” qui exécute la logique (sécurité, calculs, envoi de notifications), lit/écrit la base de données.
* **Base de données (BD)** : l’endroit où l’on stocke les informations (utilisateurs, évènements, stations d’essence, etc.).
* **Serverless** (Cloud/Edge/Functions) : du code “à la demande”, exécuté par le cloud quand un événement se produit (pas de serveur à maintenir).

**Exemple**
Quand un usager soumet un quiz, une **Function** (serverless) corrige la copie et met à jour ses points en BD.

---

# 2) Firebase – services principaux

## 2.1 Firebase Auth (authentification)

**À quoi ça sert ?** Créer/identifier les utilisateurs (email, Google, Apple…).
**Image mentale** : un **vigile** à l’entrée du bâtiment.

**Exemple (côté mobile, pseudo-code)**

```dart
final user = await FirebaseAuth.instance
  .createUserWithEmailAndPassword(email, password);
```

---

## 2.2 Firestore (base NoSQL documentaire)

**À quoi ça sert ?** Stocker des **documents** (ex.: `users/uid`, `events/id`, `workouts/id`).
**Image mentale** : des **dossiers** classés par catégories, facile à consulter.

**Exemple de structure**

```
users/{uid}
  displayName, level, points
events/{eventId}
  title, date, location
```

---

## 2.3 Règles de sécurité Firestore (Rules)

**À quoi ça sert ?** Autoriser/empêcher l’accès à des documents selon l’utilisateur.
**Image mentale** : un **agent** qui vérifie “as-tu le droit de voir/modifier ce dossier ?”.

**Exemple (règle simple)**

```
// Un utilisateur peut lire/écrire uniquement son profil
match /databases/{db}/documents {
  match /users/{uid} {
    allow read, write: if request.auth != null && request.auth.uid == uid;
  }
}
```

---

## 2.4 Cloud Functions (Functions, Gen2)

**À quoi ça sert ?** Exécuter du code côté serveur pour les opérations **sensibles** (notes, points, badges, agrégats).
**Image mentale** : un **bureau interne** où l’on fait les calculs sérieux, à l’abri du public.

**Exemple (pseudo-code JS)**

```js
exports.submitQuiz = onCall(async (ctx) => {
  // 1) Vérifier l’identité
  const uid = ctx.auth?.uid; if (!uid) throw new Error("unauthenticated");
  // 2) Corriger
  const score = grade(ctx.data.answers);
  // 3) Mettre à jour progression + points
  await db.collection('progress').doc(uid).set({ lastScore: score }, { merge: true });
  return { score };
});
```

---

## 2.5 FCM (Firebase Cloud Messaging)

> Parfois orthographié “FCR” par erreur : **le bon sigle est FCM**.
> **À quoi ça sert ?** Envoyer des **notifications push** (rappels d’entraînement, alerte de quart, etc.).
> **Image mentale** : un **haut-parleur** qui envoie un message court à l’app.

**Exemple (payload minimal)**

```json
{
  "to": "<deviceToken>",
  "notification": { "title": "Rappel", "body": "Séance à 18h" }
}
```

---

## 2.6 Storage (fichiers)

**À quoi ça sert ?** Stocker **images, vidéos, PDF**.
**Image mentale** : un **dépôt** d’images et documents, séparé de la BD.

---

## 2.7 Crashlytics / Analytics / Remote Config

* **Crashlytics** : rapports d’anomalies (crash) — “boîte noire” de l’app.
* **Analytics** : statistiques d’usage (écrans visités, conversions).
* **Remote Config** : activer/désactiver une fonctionnalité **sans publier** une nouvelle version.

---

# 3) Supabase / Postgres – notions clés

## 3.1 Postgres (SGBDR relationnel)

**À quoi ça sert ?** Modéliser des données **structurées** avec **contraintes** et **transactions** (paie, pointage, validations).
**Image mentale** : un **tableur géant** avec des liens (clefs étrangères), des règles et des totaux solides.

**Exemple de tables**

```
users(id, email, display_name)
shifts(id, user_id, start_at, end_at)
time_entries(id, user_id, shift_id, check_in, check_out)
```

---

## 3.2 RLS — Row Level Security

**À quoi ça sert ?** Règles **au niveau des lignes** pour contrôler qui voit/modifie quoi.
**Image mentale** : des **lunettes filtrantes** ; chaque utilisateur ne voit que ses lignes.

**Exemple (politique RLS)**

```sql
-- Activer RLS
ALTER TABLE time_entries ENABLE ROW LEVEL SECURITY;

-- Politique : un employé ne voit que ses entrées
CREATE POLICY user_can_see_their_entries
ON time_entries
FOR SELECT
USING (user_id = auth.uid());
```

---

## 3.3 PostGIS (géospatial)

**À quoi ça sert ?** Requêtes **par distance/rayon**, tri par proximité (stations d’essence).
**Image mentale** : une **carte** avec un **compas** pour mesurer les distances.

**Exemple (stations dans un rayon de 3 km, triées par distance)**

```sql
SELECT id, name,
       ST_Distance(location, ST_MakePoint(:lon, :lat)::geography) AS meters
FROM stations
WHERE ST_DWithin(location, ST_MakePoint(:lon, :lat)::geography, 3000)
ORDER BY meters ASC
LIMIT 20;
```

---

## 3.4 Edge Functions (Supabase)

**À quoi ça sert ?** Logique serveur proche de la base (soumission/validation d’un prix, calcul de leaderboard).
**Image mentale** : un **guichet** collé à l’entrepôt des données, rapide et sécurisé.

---

# 4) Google Maps Platform (cartographie)

## 4.1 Maps, Directions, Distance Matrix

* **Maps** : afficher la carte et des marqueurs.
* **Directions** : proposer un **itinéraire** (voiture, marche).
* **Distance Matrix** : calculer **temps/distance** vers **plusieurs** destinations.

**Exemple (Distance Matrix, idée de réponse)**

```json
{
  "rows": [{ "elements": [
    { "distance": { "value": 1950 }, "duration": { "value": 360 } },
    { "distance": { "value": 3200 }, "duration": { "value": 540 } }
  ]}]
}
```

---

# 5) Géolocalisation côté mobile

## 5.1 Position, permissions, rayon

* Récupérer la position du téléphone (avec consentement).
* Filtrer les résultats par **rayon** (ex.: 2 km).
* Afficher les **stations** proches sur la carte.

**Exemple (logique simple)**

```dart
final pos = await Geolocator.getCurrentPosition();
final nearby = await api.stationsNear(pos.latitude, pos.longitude, radiusKm: 2);
```

---

# 6) Offline-first et cache local (Hive)

## 6.1 Pourquoi

* L’utilisateur doit **continuer** à utiliser l’app **sans Internet** (métro, sous-sol).
* On **enregistre** localement puis on **synchronise** quand le réseau revient.

**Exemple (principe write-behind)**

```dart
// 1) Écrire localement
await hiveBox.put('session-2025-09-17', sessionData);
// 2) En tâche de fond, tenter la sync Firestore
try { await firestore.collection('sessions').add(sessionData); }
catch (_) { retryLater(); }
```

---

# 7) Notifications : rappel, alerte, engagement

## 7.1 FCM en pratique

* **Planification** d’un rappel (ex.: “Séance à 18h”, “Quart à 7h”) via une Function.
* **Envoi ciblé** à un appareil (token) ou à un **topic** (ex.: “ville\:montreal”).

**Exemple (idée de fonction d’envoi)**

```js
await messaging.send({
  token: deviceToken,
  notification: { title: "Rappel", body: "Votre séance débute dans 30 minutes." }
});
```

---

# 8) Sécurité et conformité

## 8.1 Firebase Rules vs RLS

* **Firebase Rules** : filtre d’accès **documentaire** (côté Firestore).
* **RLS (Postgres)** : filtre d’accès **au niveau des lignes** en SQL, **dans la base**.

**Règle de sélection**

* Si vous gérez des **pièces justificatives/paie** : privilégier **RLS** et transactions SQL.
* Si vos données sont **centrées utilisateur** en mode **documentaire** (cours, agenda, workouts) : **Firestore Rules** suffit souvent.

---

# 9) Exemples concrets par cas d’usage

## 9.1 DevLingo (cours/quiz/progression) — Firebase

* Firestore : `courses`, `lessons`, `quizzes`, `users`, `progress`.
* Function `submitQuiz()` : corrige, met à jour points/badges.
* FCM : rappel de révision.

**Pourquoi Firebase ?** Données par utilisateur, opérations CRUD simples, **notifications** intégrées, **mode hors-ligne** facile.

---

## 9.2 Sweatly (workouts/progression) — Firebase

* Firestore : `exercises`, `workouts`, `sessions`, `metrics_daily`.
* Function `onSessionWrite()` : calcule volume/durée, streaks.
* Hive : exécuter un workout **sans réseau**.

**Pourquoi Firebase ?** Documents, synchronisation simple, **Functions** pour agrégats, **Analytics** pour l’engagement.

---

## 9.3 PlaneEx (agenda) — Firebase + APIs

* Firestore : `events` (par utilisateur).
* Function `detectConflicts()` : repère chevauchements d’évènements.
* Distance Matrix : ETA et heure de départ recommandée.

**Pourquoi Firebase ?** Évènements = documents; **FCM** pour rappels; **Functions** pour conflits.

---

## 9.4 Shiftly (horaires/pointage/paie) — Postgres + RLS (+ cache Firestore optionnel)

* Postgres : `users`, `shifts`, `time_entries`, `payslips`.
* RLS : employés ne voient que leurs données.
* Exports : **rapports** fiables (SQL, PDF).
* Firestore (option) : **cache** “prochains quarts”.

**Pourquoi Postgres ?** **Transactions**, **contraintes**, **rapports** — base **relationnelle** indispensable.

---

## 9.5 GazGo (stations/prix géolocalisés) — Postgres + PostGIS

* Postgres : `stations(geography)`, `prices`, `validations`, `users`.
* PostGIS : stations **dans un rayon** + **tri par distance**.
* Edge Functions : soumission/validation, **leaderboard**, recommandations.

**Pourquoi PostGIS ?** **Géospatial** avancé (performant, expressif), **transactions** pour la fiabilité des prix.

---

# 10) CI/CD et qualité

## 10.1 CI/CD

* **CI** : exécuter tests et lints à chaque push.
* **CD** : produire automatiquement des builds Android/iOS (staging → prod).

**Exemple (idées de jobs GitHub Actions)**

* `flutter test`
* build Android App Bundle
* upload TestFlight (iOS)

## 10.2 Observabilité

* **Crashlytics** : détecter et prioriser les crashs.
* **Analytics** : suivre l’adoption des fonctionnalités et éliminer les frictions.

---

# 11) Lexique rapide

* **FCM** : Firebase Cloud Messaging (notifications).
* **(FCR)** : confusion fréquente avec FCM ; peut aussi désigner “Firebase Crashlytics” ou “Firebase Remote Config”.
* **RLS** : Row Level Security (sécurité par ligne en SQL).
* **Edge/Cloud Functions** : fonctions serverless côté backend.
* **PostGIS** : extension géospatiale de Postgres.
* **ETA** : Estimated Time of Arrival (heure d’arrivée estimée).
* **ICS** : format standard de calendrier (import/export).
* **Offline-first** : l’app reste utilisable sans réseau, synchronise ensuite.
* **Geohash/GeoPoint** : codage de positions géo (utile avec Firestore).
* **Leaderboard** : classement des utilisateurs.
* **Streak** : nombre de jours consécutifs d’activité.

---

## Conclusion — Règle de décision simple

* **Données documentaires centrées utilisateur**, CRUD simples, **notifications** et **offline** → **Firebase** (Auth, Firestore, Functions, FCM).
* **Données métier relationnelles**, **transactions**, **rapports** SQL, **géospatial avancé** → **Postgres** (Supabase) + **RLS** (et **PostGIS** si carte/distance).

Si vous voulez, je peux livrer ce contenu **en DOCX** avec une mise en page pédagogique (titres numérotés, table des matières) et y ajouter des **exercices pratiques** (mini-lab : écrire une règle Firestore, créer une RLS, requêter PostGIS sur un rayon).
