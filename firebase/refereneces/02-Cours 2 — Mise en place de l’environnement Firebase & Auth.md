# Cours 2 — Mise en place de l’environnement Firebase & Auth

## Objectifs

* Installer et configurer l’outillage (Node.js, Firebase CLI).
* Créer le projet Firebase, activer **Authentication** (email/mot de passe) et **Firestore**.
* Initialiser **Cloud Functions (TypeScript)** + **Express**.
* Déployer une première fonction HTTP et préparer la validation d’ID tokens.
* Poser des **règles Firestore** minimales et tester en local (Emulators) et en cloud.

## Prérequis

* Node.js ≥ 18, npm ≥ 9.
* Un compte Google.

---

## 1) Installation & création du projet

1. Installer le CLI Firebase :

   ```
   npm install -g firebase-tools
   firebase --version
   ```
2. Se connecter :

   ```
   firebase login
   ```
3. Créer un projet (console Firebase) puis noter l’ID projet : `PROJECT_ID`.

---

## 2) Activer Auth et Firestore

Dans la console Firebase :

* **Authentication** → **Sign-in method** → activer **Email/Password**.
* **Firestore** → **Create database** → mode **Production** (ou **Test** si vous travaillez 100% en émulateur).

---

## 3) Initialiser Cloud Functions (TypeScript)

Dans un dossier vide de travail :

```
firebase init functions
```

Choisir : **TypeScript**, **ESLint** (oui), **npm**. Une arborescence `functions/` est créée.

Scripts utiles (déjà présents ou à vérifier) dans `functions/package.json` :

```json
{
  "scripts": {
    "build": "tsc",
    "serve": "firebase emulators:start",
    "deploy": "firebase deploy --only functions",
    "lint": "eslint ."
  }
}
```

> Remarque : les **Cloud Functions payantes** nécessitent le plan **Blaze**. Vous pouvez travailler en **Emulator** gratuitement pour le développement local.

---

## 4) Installer Express et middleware utiles

Dans `functions/` :

```
npm i express cors
npm i -D @types/express @types/cors
```

---

## 5) Première fonction HTTP avec Express

Fichier `functions/src/index.ts` :

```ts
import * as functions from "firebase-functions";
import express from "express";
import cors from "cors";

const app = express();
app.use(cors());
app.use(express.json());

app.get("/health", (_req, res) => {
  res.status(200).json({ ok: true, service: "api", version: "v1" });
});

export const api = functions.https.onRequest(app);
```

Déploiement (cloud) :

```
npm run build
npm run deploy
```

Notez l’URL renvoyée (ex. `https://REGION-PROJECT_ID.cloudfunctions.net/api/health`).

Test rapide :

```
curl https://.../api/health
```

---

## 6) Préparer l’Admin SDK & l’Auth côté serveur

Nous validerons les **ID tokens Firebase** envoyés par les clients.

Installer l’Admin SDK :

```
cd functions
npm i firebase-admin
```

Créer `functions/src/firebase.ts` :

```ts
import * as admin from "firebase-admin";

// Initialisation par défaut (recommandée dans Functions)
// Utilise les credentials gérés par Google dans l'environnement Functions.
if (!admin.apps.length) {
  admin.initializeApp();
}

export const auth = admin.auth();
export const db = admin.firestore();
```

Middleware d’authentification `functions/src/middlewares/auth.ts` :

```ts
import { Request, Response, NextFunction } from "express";
import { auth } from "../firebase";

export async function requireAuth(req: Request, res: Response, next: NextFunction) {
  try {
    const header = req.headers.authorization || "";
    const [, token] = header.split(" "); // "Bearer <ID_TOKEN>"
    if (!token) return res.status(401).json({ error: "Missing Bearer token" });

    const decoded = await auth.verifyIdToken(token);
    // attacher l'UID au contexte requête
    (req as any).uid = decoded.uid;
    next();
  } catch (e) {
    return res.status(401).json({ error: "Invalid or expired token" });
  }
}
```

Brancher le middleware dans `index.ts` :

```ts
import { requireAuth } from "./middlewares/auth";

// routes publiques
app.get("/health", (_req, res) => res.status(200).json({ ok: true }));

// routes protégées (exemple)
app.get("/me", requireAuth, (req, res) => {
  const uid = (req as any).uid;
  res.status(200).json({ uid });
});
```

---

## 7) Règles de sécurité Firestore (minimales pour « notes »)

Fichier `firestore.rules` à la racine (même niveau que `functions/`) :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /notes/{noteId} {
      allow create: if request.auth != null
                    && request.resource.data.ownerUid == request.auth.uid;
      allow read, update, delete: if request.auth != null
                    && resource.data.ownerUid == request.auth.uid;
    }
  }
}
```

Publier les règles :

```
firebase deploy --only firestore:rules
```

---

## 8) Emulators pour un flux de dev rapide

Initialiser :

```
firebase init emulators
```

Choisir **Functions**, **Firestore**, **Auth**, ports par défaut.

Fichier `firebase.json` (exemple pertinent) :

```json
{
  "functions": {
    "source": "functions"
  },
  "emulators": {
    "functions": { "port": 5001 },
    "firestore": { "port": 8080 },
    "auth": { "port": 9099 },
    "ui": { "enabled": true }
  }
}
```

Lancer :

```
npm run serve
```

UI des émulateurs (souvent `http://localhost:4000`).

---

## 9) Test côté client : obtenir un ID token

Exemple Web minimal (client) :

```ts
import { initializeApp } from "firebase/app";
import { getAuth, signInWithEmailAndPassword } from "firebase/auth";

const app = initializeApp({
  apiKey: "…",
  authDomain: "…",
  projectId: "…",
});
const auth = getAuth(app);

async function callMe() {
  const cred = await signInWithEmailAndPassword(auth, "john@doe.com", "secret123");
  const idToken = await cred.user.getIdToken();

  const res = await fetch("https://…cloudfunctions.net/api/me", {
    headers: { Authorization: `Bearer ${idToken}` }
  });
  console.log(await res.json());
}
```

---

## 10) Structure de base pour la suite (CRUD « notes »)

Créez un contrôleur `functions/src/controllers/noteController.ts` (squelette) :

```ts
import { Request, Response } from "express";
import { db } from "../firebase";

export async function createNote(req: Request, res: Response) {
  const uid = (req as any).uid;
  const { title, content } = req.body || {};
  if (!title) return res.status(422).json({ error: "title is required" });

  const doc = db.collection("notes").doc();
  const note = {
    id: doc.id,
    title,
    content: content ?? "",
    ownerUid: uid,
    createdAt: Date.now(),
    updatedAt: Date.now()
  };
  await doc.set(note);
  return res.status(201).json({ data: note });
}
```

Routing (dans `index.ts`) :

```ts
import { createNote } from "./controllers/noteController";

app.post("/v1/notes", requireAuth, createNote);
```

Test rapide (après déploiement) :

```
curl -X POST https://.../api/v1/notes \
 -H "Content-Type: application/json" \
 -H "Authorization: Bearer <ID_TOKEN>" \
 -d '{"title":"Ma première note","content":"Bonjour"}'
```

---

## 11) Pièges fréquents

* **ID token vs token de service** : côté client = ID token Firebase (respecte vos règles). Le token de service (OAuth/IAM) contourne les règles Firestore et n’est pas destiné aux appels utilisateurs.
* **Horodatage** : privilégier `FieldValue.serverTimestamp()` lorsque vous écrirez depuis le SDK client ; côté Admin SDK, `Date.now()` est courant.
* **Emulators** : pensez à initialiser l’Auth Emulator dans votre client (ex. `connectAuthEmulator(auth, "http://localhost:9099")`) quand vous testez en local.

---

## Atelier (15–25 min)

* Déployer `GET /health`, `GET /me` (protégé), et `POST /v1/notes`.
* Créer un utilisateur de test (console Auth ou Emulator UI), se connecter côté client, appeler `/me` et `/v1/notes` avec le **Bearer token**.
* Vérifier dans Firestore que la note est créée avec `ownerUid`.

### Livrables

* Capture d’écran de Postman/cURL montrant `201 Created` pour `/v1/notes`.
* Extrait du code `index.ts` (routes) + `noteController.ts` (create).
* Règles Firestore appliquées (`firestore.rules`).



## Quiz rapide

1. Où placer l’ID token dans une requête HTTP ?
2. Quelle différence entre un ID token et un token de compte de service ?
3. Donnez une règle Firestore qui autorise seulement le propriétaire d’un document à le lire.
4. Quel code HTTP renvoyer lorsqu’une ressource est créée avec succès ?



## Préparation du Cours 3

* Implémenter **GET /v1/notes** (liste paginée, filtrage par `ownerUid`), **GET /v1/notes/{id}**, **PATCH /v1/notes/{id}**, **DELETE /v1/notes/{id}**.
* Mettre en place une pagination **cursor-based** (avec `startAfter` + `limit`).

