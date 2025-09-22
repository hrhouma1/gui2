# Plan du cours : Création complète d’une API REST avec Firebase (Auth + Firestore + Functions + Express)

## Bloc 1 — Fondations et mise en place

1. **Introduction aux API REST et à Firebase**

   * Principes REST (statelessness, verbes HTTP, codes de statut).
   * Comparaison REST vs RPC vs GraphQL.
   * Rôle de Firebase : Auth, Firestore, Functions, Storage.
   * Exemple d’architecture cible.

2. **Installation et configuration de l’environnement**

   * Node.js et Firebase CLI.
   * Création du projet Firebase.
   * Initialisation des Cloud Functions.
   * Plan payant Blaze et limitations du plan gratuit.

3. **Découverte de Firestore**

   * Concepts clés (collections, documents, sous-collections).
   * Modélisation NoSQL (normalisation vs dénormalisation).
   * Requêtes de base et REST API Firestore.
   * Premiers tests avec Postman.

4. **Mise en place de Firebase Authentication**

   * Activation email/mot de passe.
   * Inscription, connexion, réinitialisation.
   * Obtention et utilisation des ID tokens.
   * Vérification des tokens côté serveur.

---

## Bloc 2 — Construction des endpoints CRUD

5. **Initialisation d’Express et Cloud Functions**

   * Installation d’Express.
   * Création de la fonction HTTPS.
   * Middleware simple et route racine.
   * Déploiement initial.

6. **Implémentation des opérations CRUD**

   * POST /entries : création de document.
   * GET /entries : récupération d’une collection.
   * PATCH /entries/\:id : mise à jour partielle.
   * DELETE /entries/\:id : suppression.
   * Gestion des erreurs basiques.

7. **Sécurisation avec les règles Firestore**

   * Introduction aux règles de sécurité.
   * Restriction aux utilisateurs connectés.
   * Filtrage par propriété ownerUid.
   * Cas pratiques avec Postman.

8. **Validation et gestion des erreurs**

   * Validation d’entrée (Joi ou Zod).
   * Standardisation des réponses JSON.
   * Middleware global d’erreurs.
   * Exemples d’erreurs normalisées.

---

## Bloc 3 — Fonctionnalités avancées

9. **Pagination, tri et filtres**

   * Pagination avec limit et cursor.
   * Tri ascendant/descendant.
   * Filtres complexes (where + orderBy).
   * Index composés dans Firestore.

10. **Recherche et indexation**

* Recherche textuelle simple (title\_lower).
* Préfixes et correspondances partielles.
* Création et test d’index composites.
* Limites et alternatives (Algolia, Elasticsearch).

11. **Sécurité avancée et rôles**

* Custom claims dans Firebase Auth.
* Rôles user et admin.
* Règles RBAC dans Firestore.
* Endpoint admin (promotion d’un utilisateur).

12. **Idempotence et limitation de débit**

* Idempotency-Key pour POST.
* Détection des doublons.
* Rate limiting simple (par UID et route).
* Recommandations backoff côté client.

13. **Tests automatisés**

* Jest et Supertest pour tests HTTP.
* Emulator Suite pour tests d’intégration.
* Tests de règles avec @firebase/rules-unit-testing.
* Structuration des tests.

---

## Bloc 4 — Industrialisation

14. **Documentation avec OpenAPI (Swagger)**

* Création d’une spécification OpenAPI 3.0.
* Exposition de Swagger UI.
* Cohérence entre code et documentation.
* Tests contractuels.

15. **Observabilité et métriques**

* Corrélation via X-Request-Id.
* Logs structurés (JSON).
* Metrics minimalistes (latence, statut, route).
* Export potentiel vers Prometheus ou Datadog.

16. **CI/CD avec GitHub Actions**

* Installation et configuration Firebase Action.
* Secrets (service account, token).
* Pipeline : lint, build, test, déploiement.
* Stratégies multi-environnements (dev/staging/prod).

17. **Export et import de données**

* Endpoint d’export JSON (Content-Disposition).
* Upload vers Cloud Storage.
* Import de données depuis Storage.
* Bonnes pratiques pour gros volumes.

18. **Fichiers et Cloud Storage**

* Gestion d’uploads binaires.
* Génération d’URLs signées.
* Règles de sécurité Storage.
* Intégration avec Firestore.

19. **Optimisations et performance**

* Transactions et batch writes.
* Limites Firestore (taille doc, écriture/s).
* Caching côté client (ETag, If-None-Match).
* Compression HTTP et best practices.

---

## Bloc 5 — Finalisation et ouverture

20. **Projet final et soutenance**

* API REST complète : Auth, CRUD, règles RBAC, idempotence, pagination.
* Documentation Swagger.
* Tests automatisés.
* Pipeline CI/CD opérationnel.
* Présentation orale (architecture, sécurité, limites, perspectives).

### Durée

* Chaque cours = 2 à 3 heures (théorie + atelier).
* Le projet fil rouge progresse à chaque cours.
* À la fin, chaque étudiant dispose d’une API complète et sécurisée, prête à démontrer.
