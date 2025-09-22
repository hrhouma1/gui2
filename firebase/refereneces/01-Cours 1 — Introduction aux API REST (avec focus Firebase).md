

# Cours 1 — Introduction aux API REST (avec focus Firebase)

## Objectifs d’apprentissage

* Comprendre ce qu’est une API REST et ses principes (ressources, verbes HTTP, statelessness).
* Distinguer REST vs RPC/GraphQL, et où Firebase s’insère.
* Savoir lire/écrire une **spécification minimale** d’endpoint (URI, méthode, corps, codes HTTP).
* Préparer le terrain pour l’authentification et le CRUD sur Firestore (cours 2+).

## Pré-requis

* Bases HTTP (requête/réponse, en-têtes, corps).
* Notions JSON.
* Node.js installé (sera utilisé dès le cours 3).

## 1) Qu’est-ce qu’une API REST ?

Une API REST expose des **ressources** (ex. `/users`, `/orders/123`) via des **URI stables**, manipulées par des **verbes HTTP**:

* `GET` (lecture), `POST` (création), `PATCH` (mise à jour partielle), `PUT` (remplacement), `DELETE` (suppression).
  Caractéristiques clés :
* **Sans état (stateless)** : chaque requête contient tout ce qu’il faut (auth incluse).
* **Uniformité** : mêmes règles d’URI, de verbes, de codes HTTP partout.
* **Représentations** : JSON est courant (mais XML/CSV possibles).

## 2) Où intervient Firebase ?

Dans ce parcours, Firebase jouera plusieurs rôles :

* **Firebase Authentication** : émission de **ID tokens** que le client joindra à `Authorization: Bearer <token>`.
* **Firestore** (base NoSQL) : stockage des ressources.
* **Cloud Functions (HTTP)** : notre « couche API » (routes Express) qui valident, authentifient, lisent/écrivent Firestore et renvoient des réponses JSON.
* (Optionnel) **Realtime Database** : alternative à Firestore, utile pour certains cas temps réel.

## 3) Ressources, URI et versionnement

Design recommandé :

* **Pluriels** : `/users`, `/orders`, `/orders/{orderId}`.
* **Versionnement** (simple) : `/v1/orders` (préféré) plutôt que des en-têtes custom.
* **Filtrage/Pagination** : `/v1/orders?status=pending&limit=20&cursor=abc123`.

## 4) Verbes HTTP et sémantique

| Verbe  | Usage court                       |
| ------ | --------------------------------- |
| GET    | Lire une ou plusieurs ressources  |
| POST   | Créer (souvent sur la collection) |
| PATCH  | Modifier partiellement            |
| PUT    | Remplacer (idempotent)            |
| DELETE | Supprimer                         |

**Idempotence** : `GET`, `PUT`, `DELETE`, `HEAD` sont idempotents (répéter la requête ne change pas le résultat final), `POST` ne l’est pas.

## 5) Codes HTTP à connaître

| Code | Signification                                |
| ---- | -------------------------------------------- |
| 200  | OK (réussite)                                |
| 201  | Created (ressource créée)                    |
| 204  | No Content (suppression/réussite sans corps) |
| 400  | Bad Request (validation)                     |
| 401  | Unauthorized (auth manquante/invalide)       |
| 403  | Forbidden (droit insuffisant)                |
| 404  | Not Found                                    |
| 409  | Conflict (doublon, état)                     |
| 422  | Unprocessable Entity (validation fine)       |
| 429  | Too Many Requests (quotas)                   |
| 500  | Erreur serveur                               |

## 6) Contrats d’API (structure JSON)

Réponses claires et cohérentes : enveloppe minimaliste, champs stables, erreurs explicites.

<pre><code>{
  "data": {
    "id": "ord_123",
    "status": "pending",
    "total": 129.99
  },
  "meta": {
    "requestId": "req_abc",
    "timestamp": "2025-09-22T19:00:00Z"
  }
}
</code></pre>

Erreurs normalisées :

<pre><code>{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Le champ total est requis.",
    "details": [{"field": "total", "rule": "required"}]
  }
}
</code></pre>

## 7) Authentification — panorama (avant Firebase)

* **Bearer Token** dans l’en-tête `Authorization`.
* **Scopes/Rôles** côté serveur pour autoriser les actions.
* Avec Firebase, le **ID token** signé par Google prouve l’identité de l’utilisateur. Nos endpoints valident ce token côté serveur (librairie Admin), récupèrent `uid` et appliquent la logique d’autorisation.

## 8) Bonnes pratiques de design

* **Noms de ressources** métier, pas techniques: `customers`, `products`, `invoices`.
* **Champs stables** : ajouter plutôt que renommer/supprimer.
* **Pagination cursor-based** (plus robuste que offset).
* **Tri & filtre** stables : `?sort=-createdAt&status=paid`.
* **Idempotency-Key** (pour POST sensibles) → en-tête personnalisé pour éviter les doublons client (on y reviendra).

## 9) Exemple concret (futur endpoint Firebase)

Lecture (publique ou protégée selon les règles) :

<pre><code># Lire une collection
GET /v1/notes?limit=20&amp;cursor=abc

# Lire une ressource
GET /v1/notes/n_123
</code></pre>

Création (protégé) :

<pre><code># Créer une note (Authorization: Bearer &lt;ID_TOKEN&gt;)
POST /v1/notes
Content-Type: application/json

{
  "title": "Ma première note",
  "content": "Bonjour monde"
}
</code></pre>

Réponse (succès) :

<pre><code>201 Created
{
  "data": {
    "id": "n_123",
    "title": "Ma première note",
    "content": "Bonjour monde",
    "ownerUid": "uid_abc",
    "createdAt": "2025-09-22T19:01:00Z"
  }
}
</code></pre>

## 10) Ce que nous ferons dès le cours 2+

* Activer Firebase Auth, générer des ID tokens côté client, valider côté serveur (Cloud Functions).
* Construire une API Express déployée sur Firebase: `/v1/notes` avec **CRUD** complet.
* Règles de sécurité Firestore et tests d’autorisation.
* Pagination, tri, filtres, gestion d’erreurs et observabilité.

---

## Atelier (15–25 min)

1. **Cartographier vos ressources** pour le mini-projet « Notes » :

   * Collections: `users`, `notes`.
   * Champs `notes`: `title`, `content`, `ownerUid`, `createdAt`, `updatedAt`.
2. **Définir les endpoints v1** (URI + méthode + corps de requête) :

   * `GET /v1/notes` (liste paginée, filtrage par propriétaire).
   * `POST /v1/notes` (création).
   * `GET /v1/notes/{id}` (lecture).
   * `PATCH /v1/notes/{id}` (mise à jour partielle).
   * `DELETE /v1/notes/{id}` (suppression).
3. **Spécifier les réponses** (succès/erreurs) avec codes HTTP et schémas JSON.

> Livrable : une page (ou fichier) listant endpoints, exemples de requêtes/réponses, et codes d’erreur attendus.

## Quiz rapide (auto-évaluation)

1. Citer 3 propriétés d’une API REST bien conçue.
2. Différence principale entre `PUT` et `PATCH` ?
3. Pourquoi `POST` n’est pas idempotent ?
4. Que retourne idéalement un `POST` réussi (corps et code HTTP) ?
5. Où place-t-on un **ID token** d’authentification dans une requête ?

## Lecture/Préparation pour le cours 2

* Installer/mettre à jour **Node.js** et **Firebase CLI**.
* Créer un projet Firebase (console) que nous utiliserons pour Auth + Firestore.
* Réfléchir aux **règles d’accès** : qui peut voir/éditer/supprimer une note ?

### Prochaine étape

**Cours 2 — Mise en place de l’environnement Firebase & Auth**
On activera Firebase Authentication (email/mot de passe), on préparera Firestore, et on posera les bases de nos endpoints avec Cloud Functions + Express. 
