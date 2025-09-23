# Cours 2 — Mise en place de l’environnement Firebase & Auth

## Objectifs

* Installer et configurer l’outillage (Node.js, Firebase CLI).
* Créer le projet Firebase, activer **Authentication** (email/mot de passe) et **Firestore**.
* Initialiser **Cloud Functions (TypeScript)** + **Express**.
* Déployer une première fonction HTTP et préparer la validation d’ID tokens.
* Poser des **règles Firestore** minimales et tester en local (Emulators) et en cloud.

## Prérequis

* Node.js ≥ 18, npm ≥ 9
* Un compte Google

---

## 1) Installation & création du projet

1. Installer le CLI Firebase :

```bash
npm install -g firebase-tools
firebase --version
```

2. Se connecter :

```bash
firebase login
```

3. Créer un projet dans la **console Firebase** et noter l’ID projet, par exemple `PROJECT_ID`.

---

## 2) Activer Auth et Firestore

Dans la **console Firebase** :

* **Authentication** → **Sign-in method** → activer **Email/Password**.
* **Firestore** → **Create database** → mode **Production** (ou **Test** si vous travaillez 100% avec les émulateurs).

---

## 3) Initialiser Cloud Functions (TypeScript)

Dans un **dossier vide** de travail :

```bash
firebase init functions
```

Choisir : **TypeScript**, **ESLint (Yes)**, **npm**. Une arborescence `functions/` est créée.

Scripts (exemple initial) dans `functions/package.json` :

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

> Remarque : Les **Cloud Functions** facturables nécessitent le plan **Blaze**. Vous pouvez travailler gratuitement en **Emulator** pour le dev local.

---

## 4) Installer Express et middleware utiles

Dans le dossier `functions/` :

```bash
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
  res.status(200).json({ok: true, service: "api", version: "v1"});
});

export const api = functions.https.onRequest(app);
```

Déploiement (cloud) :

```bash
cd functions
npm run build
npm run deploy
firebase deploy --only functions (il faut passer au plan pay as you go)
npm run serve (en mode développement)
curl http://localhost:5001/backend-demo-1/us-central1/api/health
```

- Possibilité d'erreur classique (Voir annexe 1)


```bash
PS C:\03-projetsGA\autres\demoflutter1\functions> firebase deploy --only functions
(node:67428) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)

=== Deploying to 'backend-demo-1'...

i  deploying functions
Running command: npm --prefix "$RESOURCE_DIR" run lint

> lint
> eslint --ext .js,.ts .

=============

WARNING: You are currently running a version of TypeScript which is not officially supported by @typescript-eslint/typescript-estree.

You may find that it works just fine, or you may not.

SUPPORTED TYPESCRIPT VERSIONS: >=3.3.1 <5.2.0

YOUR TYPESCRIPT VERSION: 5.9.2

Please only submit bug reports when using the officially supported version.

=============
Running command: npm --prefix "$RESOURCE_DIR" run build

> build
> tsc

+  functions: Finished running predeploy script.
i  functions: preparing codebase default for deployment
i  functions: ensuring required API cloudfunctions.googleapis.com is enabled...
i  functions: ensuring required API cloudbuild.googleapis.com is enabled...
i  artifactregistry: ensuring required API artifactregistry.googleapis.com is enabled...
!  functions: missing required API cloudbuild.googleapis.com. Enabling now...
!  artifactregistry: missing required API artifactregistry.googleapis.com. Enabling now...

Error: Your project backend-demo-1 must be on the Blaze (pay-as-you-go) plan to complete this command. Required API artifactregistry.googleapis.com can't be enabled until the upgrade is complete. To upgrade, visit the following URL:

https://console.firebase.google.com/project/backend-demo-1/usage/details
```




Notez l’URL renvoyée (ex. `https://REGION-PROJECT_ID.cloudfunctions.net/api/health`).

Test rapide :

```bash
curl https://REGION-PROJECT_ID.cloudfunctions.net/api/health
```

Test rapide :

```bash
http://localhost:5001/backend-demo-1/us-central1/api/health
```

---

## 6) Préparer l’Admin SDK & l’Auth côté serveur

Nous validerons les **ID tokens Firebase** envoyés par les clients.

Installer l’Admin SDK :

```bash
cd functions
npm i firebase-admin
```

Créer `functions/src/firebase.ts` :

```ts
import * as admin from "firebase-admin";

if (!admin.apps.length) {
  admin.initializeApp();
}

export const auth = admin.auth();
export const db = admin.firestore();
```

Middleware d’authentification `functions/src/middlewares/auth.ts` :

```ts
import {Request, Response, NextFunction} from "express";
import {auth} from "../firebase";

export async function requireAuth(req: Request, res: Response, next: NextFunction) {
  try {
    const header = req.headers.authorization || "";
    const [, token] = header.split(" "); // "Bearer <ID_TOKEN>"
    if (!token) return res.status(401).json({error: "Missing Bearer token"});

    const decoded = await auth.verifyIdToken(token);
    (req as any).uid = decoded.uid; // attacher l'UID au contexte requête
    next();
  } catch {
    return res.status(401).json({error: "Invalid or expired token"});
  }
}
```

Branchement du middleware dans `index.ts` :

```ts
import * as functions from "firebase-functions";
import {requireAuth} from "./middlewares/auth";

import express from "express";
import cors from "cors";

const app = express();
app.use(cors());
app.use(express.json());


// routes publiques
app.get("/health", (_req, res) => res.status(200).json({ok: true}));

// routes protégées (exemple)
app.get("/me", requireAuth, (req, res) => {
  const uid = (req as any).uid;
  res.status(200).json({uid});
});

export const api = functions.https.onRequest(app);
```

---

## 7) Règles de sécurité Firestore (minimales, collection « notes »)

Fichier `firestore.rules` à la **racine** (au même niveau que `functions/`) :

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

Déployer les règles :

```bash
firebase deploy --only firestore:rules
```

### Attention erreur (voir Annexe 2 et Annexe 3)

```bash
PS C:\03-projetsGA\autres\demoflutter1\functions> firebase deploy --only firestore:rules
(node:30500) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)

Error: Cannot understand what targets to deploy/serve. No targets in firebase.json match '--only firestore:rules'. If you are using PowerShell make sure you place quotes around any comma-separated lists (ex: --only "functions,firestore").
PS C:\03-projetsGA\autres\demoflutter1\functions> cd ..
PS C:\03-projetsGA\autres\demoflutter1> firebase deploy --only firestore:rules
(node:48604) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)

Error: Cannot understand what targets to deploy/serve. No targets in firebase.json match '--only firestore:rules'. If you are using PowerShell make sure you place quotes around any comma-separated lists (ex: --only "functions,firestore").

Having trouble? Try firebase [command] --help
```




# Arborescence  (à la racine du projet)

```
demoflutter1/
├─ .firebaserc
├─ firebase.json              ← (déclare firestore.rules + indexes)
├─ firestore.rules            ← (règles Firestore)
├─ firestore.indexes.json     ← (index Firestore, même vide)
├─ package.json
└─ functions/
   ├─ package.json
   ├─ tsconfig.json
   ├─ .eslintrc.js
   ├─ src/
   │  ├─ index.ts
   │  ├─ firebase.ts
   │  ├─ middlewares/
   │  │  └─ auth.ts
   │  └─ controllers/
   │     └─ noteController.ts
   └─ lib/                    ← (généré par tsc après build)
```




---

## 8) Emulators pour un flux de dev rapide

Initialiser :

```bash
firebase init emulators
```

Choisir **Functions**, **Firestore**, **Auth**, ports par défaut.

Exemple de `firebase.json` :

```json
{
  "functions": {
    "source": "functions"
  },
  "emulators": {
    "functions": {"port": 5001},
    "firestore": {"port": 8080},
    "auth": {"port": 9099},
    "ui": {"enabled": true}
  }
}
```

> Voici le package.json final dans le dossier functions


```json
{
  "functions": {
    "source": "functions",
    "predeploy": [
      "npm --prefix \"%RESOURCE_DIR%\" run build"
    ]
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "emulators": {
    "functions": {
      "port": 5001
    },
    "firestore": {
      "port": 8080
    },
    "auth": {
      "port": 9099
    },
    "ui": {
      "enabled": true
    },
    "singleProjectMode": true
  }
}
```


Lancer :

```bash
npm run serve
```

UI des émulateurs (souvent) : `http://localhost:4000`



### Erreur :

```bash
PS C:\03-projetsGA\autres\demoflutter1> npm run serve npm error Missing script: "serve" npm error npm error To see a list of scripts, run: npm error npm run npm error A complete log of this run can be found in: C:\Users\rehou\AppData\Local\npm-cache\_logs\2025-09-23T00_22_10_921Z-debug-0.log
```

### Solution : voir les Annexe 5 et 6

ou exécuter

```powershell
firebase emulators:start
```

<br/>

---

## 9) Test côté client : obtenir un ID token

Exemple Web minimal (client) :

```ts
import {initializeApp} from "firebase/app";
import {getAuth, signInWithEmailAndPassword} from "firebase/auth";

const app = initializeApp({
  apiKey: "…",
  authDomain: "…",
  projectId: "…"
});
const auth = getAuth(app);

async function callMe() {
  const cred = await signInWithEmailAndPassword(auth, "john@doe.com", "secret123");
  const idToken = await cred.user.getIdToken();

  const res = await fetch("https://…cloudfunctions.net/api/me", {
    headers: {Authorization: `Bearer ${idToken}`}
  });
  console.log(await res.json());
}
```

---

## 10) Structure de base pour la suite (CRUD « notes »)

Créer `functions/src/controllers/noteController.ts` (squelette) :

```ts
import {Request, Response} from "express";
import {db} from "../firebase";

export async function createNote(req: Request, res: Response) {
  const uid = (req as any).uid;
  const {title, content} = req.body || {};
  if (!title) return res.status(422).json({error: "title is required"});

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
  return res.status(201).json({data: note});
}
```

Routing (dans `index.ts`) :

```ts
import {createNote} from "./controllers/noteController";

app.post("/v1/notes", requireAuth, createNote);
```

Test rapide (après déploiement) :

```bash
curl -X POST https://REGION-PROJECT_ID.cloudfunctions.net/api/v1/notes \
 -H "Content-Type: application/json" \
 -H "Authorization: Bearer <ID_TOKEN>" \
 -d "{\"title\":\"Ma première note\",\"content\":\"Bonjour\"}"
```

---

## 11) Pièges fréquents (rappels)

* **ID token vs token de service** : côté client = **ID token Firebase** (respecte vos règles). Le **token de service** (OAuth/IAM) contourne les règles Firestore et n’est pas pour des appels utilisateurs.
* **Horodatage** : en SDK client, préférer `FieldValue.serverTimestamp()`. En Admin SDK (dans Functions), `Date.now()` est courant.
* **Emulators** : n’oubliez pas de connecter l’Auth Emulator côté client (`connectAuthEmulator(auth, "http://localhost:9099")`) en test local.

---

## Atelier (15–25 min)

* Déployer `GET /health`, `GET /me` (protégé), `POST /v1/notes`.
* Créer un utilisateur test (console Auth ou Emulator UI), se connecter côté client, appeler `/me` et `/v1/notes` avec le **Bearer token**.
* Vérifier dans Firestore que la note est créée avec `ownerUid`.

### Livrables

* Capture d’écran Postman/cURL montrant `201 Created` pour `/v1/notes`.
* Extraits `index.ts` (routes) + `noteController.ts` (create).
* Règles Firestore appliquées (`firestore.rules`).

---

## Quiz rapide

1. Où placer l’ID token dans une requête HTTP ?
2. Différence entre un ID token et un token de compte de service ?
3. Une règle Firestore qui autorise **seulement** le propriétaire à lire un document ?
4. Quel code HTTP renvoyer quand une ressource est créée ?

---

## Préparation du Cours 3

* Implémenter **GET /v1/notes** (liste paginée, filtre par `ownerUid`), **GET /v1/notes/{id}**, **PATCH /v1/notes/{id}**, \*\*DELETE /v1/notes/{id}\`.
* Pagination **cursor-based** (`startAfter` + `limit`).

---

# Arborescence ASCII exhaustive (après initialisation et ajouts du cours)

```
demoflutter1/
├─ .firebaserc
├─ firebase.json
├─ firestore.rules
├─ package.json
├─ node_modules/               (si npm i à la racine)
├─ .gitignore                  (optionnel)
└─ functions/
   ├─ package.json
   ├─ tsconfig.json
   ├─ .eslintrc.js
   ├─ .gitignore
   ├─ node_modules/            (dépendances functions)
   ├─ lib/                     (sortie compilée TypeScript → JS)
   │  ├─ index.js
   │  ├─ firebase.js
   │  ├─ middlewares/
   │  │  └─ auth.js
   │  └─ controllers/
   │     └─ noteController.js
   └─ src/
      ├─ index.ts
      ├─ firebase.ts
      ├─ middlewares/
      │  └─ auth.ts
      └─ controllers/
         └─ noteController.ts
```

> Selon la config, `lib/` est généré après `npm run build` (tsc).

---

# Version « prête pour Windows » (scripts racine + predeploy sans lint)

Cette variante règle les soucis rencontrés sous PowerShell/Windows (scripts manquants à la racine, ENOENT sur `lint`, versions TS/ESLint).

## Fichiers à la racine

### `package.json` (racine)

```json
{
  "name": "backend-demo-1-root",
  "private": true,
  "scripts": {
    "build": "npm --prefix functions run build",
    "serve": "firebase emulators:start",
    "deploy:functions": "npm --prefix functions run build && firebase deploy --only functions",
    "deploy:rules": "firebase deploy --only firestore:rules",
    "deploy:all": "npm --prefix functions run build && firebase deploy --only functions,firestore:rules"
  }
}
```

### `.firebaserc` (adapter `backend-demo-1` si besoin)

```json
{
  "projects": {
    "default": "backend-demo-1"
  }
}
```

### `firebase.json` (predeploy sans lint)

```json
{
  "functions": {
    "source": "functions",
    "predeploy": [
      "npm --prefix \"%RESOURCE_DIR%\" run build"
    ]
  },
  "emulators": {
    "functions": {"port": 5001},
    "firestore": {"port": 8080},
    "auth": {"port": 9099},
    "ui": {"enabled": true}
  }
}
```

### `firestore.rules`

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

## Dossier `functions/`

### `functions/package.json`

```json
{
  "name": "backend-demo-1-functions",
  "main": "lib/index.js",
  "private": true,
  "engines": {
    "node": "18"
  },
  "scripts": {
    "build": "tsc",
    "serve": "firebase emulators:start",
    "deploy": "firebase deploy --only functions",
    "lint": "eslint --ext .js,.ts . || exit 0"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^4.19.2",
    "firebase-admin": "^12.6.0",
    "firebase-functions": "^5.1.1"
  },
  "devDependencies": {
    "@types/cors": "^2.8.18",
    "@types/express": "^4.17.21",
    "@typescript-eslint/eslint-plugin": "^8.6.0",
    "@typescript-eslint/parser": "^8.6.0",
    "eslint": "^9.8.0",
    "typescript": "5.1.6"
  }
}
```

### `functions/tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020"],
    "module": "commonjs",
    "outDir": "lib",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "moduleResolution": "node",
    "noImplicitReturns": true,
    "noUnusedLocals": true,
    "resolveJsonModule": true,
    "skipLibCheck": true
  },
  "compileOnSave": true,
  "include": ["src"]
}
```

### `functions/.eslintrc.js`

```js
/* eslint-env node */
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser',
  parserOptions: { ecmaVersion: 'latest', sourceType: 'module' },
  plugins: ['@typescript-eslint'],
  extends: ['eslint:recommended', 'plugin:@typescript-eslint/recommended'],
  rules: {
    'object-curly-spacing': ['error', 'never'],
    'eol-last': ['error', 'always'],
    'no-trailing-spaces': ['error'],
    'quotes': ['error', 'double', {'avoidEscape': true}]
  },
  ignorePatterns: ['lib/**']
};
```

### `functions/src/firebase.ts`

```ts
import * as admin from "firebase-admin";

if (!admin.apps.length) {
  admin.initializeApp();
}

export const auth = admin.auth();
export const db = admin.firestore();
```

### `functions/src/middlewares/auth.ts`

```ts
import {Request, Response, NextFunction} from "express";
import {auth} from "../firebase";

export async function requireAuth(req: Request, res: Response, next: NextFunction) {
  try {
    const header = req.headers.authorization || "";
    const [, token] = header.split(" ");
    if (!token) return res.status(401).json({error: "Missing Bearer token"});

    const decoded = await auth.verifyIdToken(token);
    (req as any).uid = decoded.uid;
    next();
  } catch {
    return res.status(401).json({error: "Invalid or expired token"});
  }
}
```

### `functions/src/controllers/noteController.ts`

```ts
import {Request, Response} from "express";
import {db} from "../firebase";

export async function createNote(req: Request, res: Response) {
  const uid = (req as any).uid as string;
  const {title, content} = req.body || {};
  if (!title || typeof title !== "string") {
    return res.status(422).json({error: "title is required (string)"});
  }

  const doc = db.collection("notes").doc();
  const now = Date.now();

  const note = {
    id: doc.id,
    title,
    content: typeof content === "string" ? content : "",
    ownerUid: uid,
    createdAt: now,
    updatedAt: now
  };

  await doc.set(note);
  return res.status(201).json({data: note});
}
```

### `functions/src/index.ts`

```ts
import * as functions from "firebase-functions";
import express from "express";
import cors from "cors";
import {requireAuth} from "./middlewares/auth";
import {createNote} from "./controllers/noteController";

const app = express();
app.use(cors());
app.use(express.json());

// Public route
app.get("/health", (_req, res) => {
  res.status(200).json({ok: true, service: "api", version: "v1"});
});

// Protected example
app.get("/me", requireAuth, (req, res) => {
  const uid = (req as any).uid as string;
  res.status(200).json({uid});
});

// Notes (create) — protected
app.post("/v1/notes", requireAuth, createNote);

export const api = functions.https.onRequest(app);
```

> Assure-toi qu’il y a **une ligne vide** en fin de fichier (`eol-last`).

---

# Commandes utiles (depuis la racine)

```bash
# 1) Installer les deps
npm install
cd functions && npm install && cd ..

# 2) Build
npm run build

# 3) Déployer uniquement les functions (cloud)
npm run deploy:functions

# 4) Déployer les règles Firestore
npm run deploy:rules

# 5) Lancer les émulateurs (local)
npm run serve
```

UI Emulators : `http://localhost:4000`

---

# Tests rapides

## Health (public)

```bash
curl https://REGION-PROJECT_ID.cloudfunctions.net/api/health
```

ou

```bash
http://localhost:5001/backend-demo-1/us-central1/api/health
```

## Auth (côté client) → `/me`

* Se connecter (`signInWithEmailAndPassword`) pour obtenir un **ID token**
* Appeler `/me` avec `Authorization: Bearer <ID_TOKEN>`

## Créer une note (protégé)

```bash
curl -X POST "https://REGION-PROJECT_ID.cloudfunctions.net/api/v1/notes" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <ID_TOKEN>" \
  -d "{\"title\":\"Ma première note\",\"content\":\"Bonjour\"}"
```

---

# Troubleshooting (Windows/PowerShell + Firebase)

## A. `npm run build` introuvable à la racine

* Les scripts `build/deploy` sont **dans `functions/`** par défaut.
* Solutions :

  1. Exécuter depuis `functions/` :

     ```powershell
     cd .\functions\
     npm run build
     firebase deploy --only functions
     ```
  2. Ajouter des **scripts à la racine** (déjà fait plus haut avec `--prefix functions`) et utiliser :

     ```powershell
     npm run build
     npm run deploy:functions
     ```

## B. ESLint bloque le déploiement (ex. `object-curly-spacing`, `eol-last`)

* Corriger le code (imports **sans espaces** : `import {requireAuth} from ...`) et **newline fin de fichier**.
* Auto-fix :

  ```powershell
  cd .\functions\
  npx eslint --ext .ts . --fix
  npm run build
  ```
* En dépannage : rendre le lint **non bloquant**

  * `functions/package.json` → `"lint": "eslint --ext .js,.ts . || exit 0"`

## C. Erreur `spawn npm --prefix "%RESOURCE_DIR%" run lint ENOENT`

* Provoquée par le **predeploy lint**.
* Fix simple : dans `firebase.json` (racine), remplacer le predeploy par **build seulement** :

  ```json
  {
    "functions": {
      "source": "functions",
      "predeploy": ["npm --prefix \"%RESOURCE_DIR%\" run build"]
    }
  }
  ```

## D. Avertissement de version TypeScript / @typescript-eslint

* Message du style : *SUPPORTED TS <5.2.0* mais vous avez TS 5.9.x.
* Deux options :

  * **Rapide** : ignorer si le build passe.
  * **Propre** :

    ```powershell
    cd .\functions\
    npm i -D typescript@5.1.6
    ```

## E. `firebase : Le terme «firebase» n'est pas reconnu`

* Vérifier l’installation globale et le PATH :

  * `npm list -g --depth=0` doit montrer `firebase-tools`.
  * Vérifier le **prefix** npm :

    ```powershell
    npm config get prefix
    ```

    Ajouter `<prefix>\` au **PATH utilisateur** (Windows) si nécessaire (ex. `C:\Users\<user>\AppData\Roaming\npm`).
  * Fermer/réouvrir le terminal.

## F. Versions Node/npm inadéquates

* **Node ≥ 18** recommandé pour Functions v2/v5.
* Vérifier : `node -v` et `npm -v`.
* Si Node 22 : OK, mais gardez **engines.node** `"18"` dans `functions/package.json` (exécution GCF).

## G. Emulators non atteignables

* Vérifier les ports dans `firebase.json` et qu’ils ne sont pas occupés.
* Ouvrir l’UI : `http://localhost:4000`.
* Côté client, en local :

  ```ts
  import {connectAuthEmulator} from "firebase/auth";
  connectAuthEmulator(auth, "http://localhost:9099");
  ```

## H. 401 Unauthorized sur `/me` et `/v1/notes`

* Vérifier l’en-tête `Authorization: Bearer <ID_TOKEN>`.
* L’ID token doit provenir de **Firebase Auth** (après `signInWithEmailAndPassword` ou autre méthode).
* Vérifier que l’utilisateur existe (Auth console / Emulator UI).

## I. 403/Permission denied côté Firestore

* Les **règles** exigent `ownerUid == request.auth.uid`.
* Vérifier que le document écrit contient `ownerUid` correct (celui du token).
* Vérifier que l’auth est bien en place côté client (ID token valide).












<br/>
<br/>

# Annexe 1


```bash
PS C:\03-projetsGA\autres\demoflutter1\functions> firebase deploy --only functions
(node:67428) [DEP0040] DeprecationWarning: The `punycode` module is deprecated. Please use a userland alternative instead.
(Use `node --trace-deprecation ...` to show where the warning was created)

=== Deploying to 'backend-demo-1'...

i  deploying functions
Running command: npm --prefix "$RESOURCE_DIR" run lint

> lint
> eslint --ext .js,.ts .

=============

WARNING: You are currently running a version of TypeScript which is not officially supported by @typescript-eslint/typescript-estree.

You may find that it works just fine, or you may not.

SUPPORTED TYPESCRIPT VERSIONS: >=3.3.1 <5.2.0

YOUR TYPESCRIPT VERSION: 5.9.2

Please only submit bug reports when using the officially supported version.

=============
Running command: npm --prefix "$RESOURCE_DIR" run build

> build
> tsc

+  functions: Finished running predeploy script.
i  functions: preparing codebase default for deployment
i  functions: ensuring required API cloudfunctions.googleapis.com is enabled...
i  functions: ensuring required API cloudbuild.googleapis.com is enabled...
i  artifactregistry: ensuring required API artifactregistry.googleapis.com is enabled...
!  functions: missing required API cloudbuild.googleapis.com. Enabling now...
!  artifactregistry: missing required API artifactregistry.googleapis.com. Enabling now...

Error: Your project backend-demo-1 must be on the Blaze (pay-as-you-go) plan to complete this command. Required API artifactregistry.googleapis.com can't be enabled until the upgrade is complete. To upgrade, visit the following URL:

https://console.firebase.google.com/project/backend-demo-1/usage/details
```



Il s'agit d'un blocage classique : **le déploiement des Cloud Functions (gen2)** nécessite d’activer **Cloud Build** et **Artifact Registry**, ce qui exige le **plan Blaze (pay-as-you-go)**. Sur **Spark**, tu peux utiliser les **Emulators** et déployer Firestore rules/hosting, mais **pas** les Functions en cloud.

Voici nos options — rapides et nettes.

---

# Option A — Continuer sans frais (local uniquement)

Tu peux tout faire tourner **en local** pour le cours :

1. Lancer les émulateurs

```bash
npm run serve
```

URL typiques (par défaut) :

* **Emulator UI** : [http://localhost:4000](http://localhost:4000)
* **Function HTTP** :

  ```
  http://localhost:5001/<PROJECT_ID>/us-central1/api/health
  http://localhost:5001/<PROJECT_ID>/us-central1/api/me
  ```

  (remplace `<PROJECT_ID>` par le tien)

2. Côté client (pendant que tu es en local), pense à connecter l’émulateur Auth :

```ts
import {connectAuthEmulator} from "firebase/auth";
connectAuthEmulator(auth, "http://localhost:9099");
```

3. Tu peux **déployer** quand même les **règles Firestore** (autorisé en Spark) :

```bash
npm run deploy:rules
```

> Pour la démo en classe, ça suffit souvent : tout le backend tourne localement, les étudiants testent avec Postman/cURL contre l’URL `localhost:5001/...`.

---

# Option B — Déployer en Cloud (exige Blaze)

Si tu veux une URL publique (`https://REGION-PROJECT.cloudfunctions.net/api/...`) :

## Étapes exactes

1. **Passer au plan Blaze**

* Ouvre le lien fourni par le message d’erreur (ou va dans **Firebase Console → Usage & billing → Upgrade**), rattache une carte.
* (Recommandé) Configure un **budget** et des **alertes** sur Google Cloud Billing pour éviter les surprises.

2. **Réexécuter le déploiement**
   Les APIs **cloudbuild.googleapis.com** et **artifactregistry.googleapis.com** seront (ré)activées automatiquement par `firebase deploy` si besoin.

```bash
npm run deploy:functions
```

3. **Contrôle des coûts (important)**

* Dans `functions/package.json`, garde `engines.node: "18"`.
* Ne configure **pas** de `minInstances` (par défaut = 0, pas de cold-start facturé).
* Garde un seul region (ex. `us-central1`) pour rester dans une zone bon marché.
* Évite les appels sortants inutiles (egress).
* Pas de tâches planifiées (Cloud Scheduler) tant que tu ne veux pas de coûts additionnels.

> Dès que Blaze est actif, le message “must be on the Blaze plan” disparaît et le déploiement passe.

---

## (Facultatif) Forcer gen1 ?

* Avec Functions SDK v5, tes handlers `functions.https.onRequest` ciblent **gen2** par défaut.
* Si tu importes depuis `firebase-functions/v1` (ex. `import * as functions from "firebase-functions/v1";`), tu forces **gen1**.
* **Mais** le déploiement de gen1 demande **aussi** Cloud Build côté Google. En pratique, **tu auras quand même besoin de Blaze**. Donc ce “downgrade” n’évite pas le paywall : je ne te le recommande pas.

---

# Pourquoi tu vois ces lignes d’erreur ?

```
! functions: missing required API cloudbuild.googleapis.com. Enabling now...
! artifactregistry: missing required API artifactregistry.googleapis.com. Enabling now...

Error: Your project backend-demo-1 must be on the Blaze (pay-as-you-go) plan ...
```

* **Cloud Build** : sert au packaging et à la livraison des fonctions côté Google.
* **Artifact Registry** : stocke l’image/container de la fonction (gen2: Cloud Run sous le capot).
* **Spark** bloque l’activation de ces APIs. D’où l’arrêt du déploiement.

---

# Checklist express selon ton choix

### Rester en local (Spark)

* `npm run serve`
* Tester sur `http://localhost:5001/<PROJECT_ID>/us-central1/api/...`
* `npm run deploy:rules` (règles Firestore seulement)

### Déployer en cloud (Blaze)

* Passer au plan Blaze (configurer budget/alertes)
* `npm run deploy:functions`
* Tester `https://REGION-PROJECT_ID.cloudfunctions.net/api/health`



<br/>
<br/>

# Annexe 2



Le message dit que Firebase **ne trouve pas la cible `firestore:rules`**… parce qu’elle **n’est pas déclarée dans ton `firebase.json`**. On corrige ça en 3 minutes.

---

## 1) Assure-toi d’être **à la racine** du projet

Vérifie que tu vois `.firebaserc`, `firebase.json`, `firestore.rules`, et le dossier `functions/`.

```powershell
cd C:\03-projetsGA\autres\demoflutter1
firebase use           # doit afficher backend-demo-1 comme "active"
```

Si besoin :

```powershell
firebase use backend-demo-1
```

---

## 2) Ajoute la section **firestore** dans `firebase.json`

Ouvre **`firebase.json`** (à la racine) et **ajoute** le bloc `firestore`. Exemple complet sûr pour Windows :

```json
{
  "functions": {
    "source": "functions",
    "predeploy": [
      "npm --prefix \"%RESOURCE_DIR%\" run build"
    ]
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "emulators": {
    "functions": { "port": 5001 },
    "firestore": { "port": 8080 },
    "auth": { "port": 9099 },
    "ui": { "enabled": true }
  }
}
```

> Le champ `"rules": "firestore.rules"` **déclare** la cible `firestore:rules`. Sans ça, `--only firestore:rules` n’existe pas pour Firebase CLI → d’où ton erreur.

---

## 3) Crée le fichier d’index (si absent)

À côté de `firestore.rules`, ajoute un fichier **`firestore.indexes.json`** minimal :

```json
{
  "indexes": [],
  "fieldOverrides": []
}
```

---

## 4) (Optionnel) Régénérer la config via l’assistant

Tu peux aussi laisser Firebase le faire :

```powershell
firebase init firestore
# → accepte de créer/écraser firestore.rules et firestore.indexes.json
```

---

## 5) Déploie **les règles** (Spark autorisé)

Sous PowerShell, la forme suivante marche très bien :

```powershell
firebase deploy --only "firestore:rules"
```

> Tu peux aussi déployer tout Firestore (règles + index) :

```powershell
firebase deploy --only "firestore"
```

---

## 6) À propos des warnings

* `DeprecationWarning: punycode` : **inoffensif**, tu peux l’ignorer.
* Le plan **Spark** bloque **les Functions cloud** (tu as déjà vu l’erreur Blaze).

  * Tu peux **déployer les règles Firestore** sans passer à Blaze.
  * Pour Functions : utilise **les émulateurs** en local, ou **upgrade Blaze** si tu veux une URL publique.

---

## 7) Vérification rapide

* Voir les règles actives (Console Firebase → Firestore → Rules)
* Test lecture/écriture côté client avec un **ID token** : tes règles exigent `ownerUid == request.auth.uid`.

---

### Récap express

1. Ajouter `"firestore": { "rules": "firestore.rules", "indexes": "firestore.indexes.json" }` dans `firebase.json`.
2. Créer `firestore.indexes.json` vide si manquant.
3. Depuis la **racine** :

   ```powershell
   firebase deploy --only "firestore:rules"
   ```















<br/>

<br/>


# Annexe 3 - Arborescence ASCII (à la racine du projet)

```
demoflutter1/
├─ .firebaserc
├─ firebase.json              ← (déclare firestore.rules + indexes)
├─ firestore.rules            ← (règles Firestore)
├─ firestore.indexes.json     ← (index Firestore, même vide)
├─ package.json
└─ functions/
   ├─ package.json
   ├─ tsconfig.json
   ├─ .eslintrc.js
   ├─ src/
   │  ├─ index.ts
   │  ├─ firebase.ts
   │  ├─ middlewares/
   │  │  └─ auth.ts
   │  └─ controllers/
   │     └─ noteController.ts
   └─ lib/                    ← (généré par tsc après build)
```

> **Important :** les fichiers **`firebase.json`**, **`firestore.rules`** et **`firestore.indexes.json`** sont **à la racine** (`demoflutter1/`), pas dans `functions/`.

---

# Contenu **complet** des fichiers Firestore/Config

## 1) `firebase.json` (à la racine)

```json
{
  "functions": {
    "source": "functions",
    "predeploy": [
      "npm --prefix \"%RESOURCE_DIR%\" run build"
    ]
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "emulators": {
    "functions": { "port": 5001 },
    "firestore": { "port": 8080 },
    "auth": { "port": 9099 },
    "ui": { "enabled": true }
  }
}
```

* La section `"firestore"` **déclare** où trouver les règles et les indexes.
* Le `predeploy` ne lance **que le build** (pas de lint) → évite les soucis `ENOENT` sous Windows.

---

## 2) `firestore.rules` (à la racine)

```
// Firestore Security Rules
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /notes/{noteId} {
      // Création autorisée seulement par un utilisateur connecté
      // et qui écrit ownerUid == request.auth.uid
      allow create: if request.auth != null
                    && request.resource.data.ownerUid == request.auth.uid;

      // Lecture / mise à jour / suppression autorisées
      // uniquement au propriétaire du document
      allow read, update, delete: if request.auth != null
                    && resource.data.ownerUid == request.auth.uid;
    }
  }
}
```

---

## 3) `firestore.indexes.json` (à la racine)

```json
{
  "indexes": [],
  "fieldOverrides": []
}
```

> Vide pour l’instant. Tu l’enrichiras quand tu ajouteras de la pagination/tri complexes.

---

# Commandes (PowerShell) pour déployer **uniquement les règles**

Depuis la **racine** (`demoflutter1\`) :

```powershell
firebase use backend-demo-1
firebase deploy --only "firestore:rules"
```

> Sous PowerShell, **garde les guillemets** autour de `firestore:rules`.
> Si tu veux déployer à la fois **règles + indexes** :

```powershell
firebase deploy --only "firestore"
```

---

# Rappels utiles

* Si tu vois encore l’erreur *“No targets in firebase.json match '--only firestore\:rules'”*, c’est presque toujours parce que :

  1. Tu n’es **pas à la racine** (tu es resté dans `functions/`), **ou**
  2. La section `"firestore"` **manque** dans `firebase.json`, **ou**
  3. Tu as tapé la commande sans guillemets sur PowerShell.

* Le plan **Spark** permet ce déploiement des **règles**.
  Pour **déployer les Functions en cloud**, il faut **Blaze** (tu peux rester sur les **émulateurs** en local sans frais).




<br/>
<br/>



# Annexe 4 - erreur 40 liée à firebase deploy --only "firestore"


Le 403 vient uniquement du **déploiement des index** : l’API **Cloud Firestore** côté **Google Cloud** n’est pas encore activée pour notre projet.

Voici la correction en deux voies (tu peux faire A tout de suite, puis B pour réactiver les index) (IL FAUT CHOISIR L'OPTION B)



## A) Déployer sans toucher aux index (immédiat)

Tant que l’API n’est pas activée, déploie **seulement les règles** :

```powershell
firebase deploy --only "firestore:rules"
```

ou enlève temporairement la clé `"indexes"` de `firebase.json` :

```json
{
  "firestore": {
    "rules": "firestore.rules"
  }
}
```

Puis:

```powershell
firebase deploy --only "firestore"
```

> Tes règles restent correctes; l’erreur 403 ne concernait que l’appel aux **indexes**.

---

## B) Activer l’API Cloud Firestore (solution propre)

1. Ouvre ce lien (déjà fourni par Firebase) et **active l’API** :
   [https://console.developers.google.com/apis/api/firestore.googleapis.com/overview?project=backend-demo-1](https://console.developers.google.com/apis/api/firestore.googleapis.com/overview?project=backend-demo-1)

   * Vérifie bien que le **projet actif** est `backend-demo-1`.
   * Clique sur **Enable** (Activer).
   * Patiente 1–2 minutes.

2. (Si ce n’est pas fait) Dans **Firebase Console → Firestore Database**, clique **Create database** et termine l’assistant (Production/Test). Ça “finalise” l’initialisation côté Firebase.

3. Relance le déploiement complet Firestore (règles + index) :

```powershell
firebase deploy --only "firestore"
```

Tu devrais voir :

* `rules file firestore.rules compiled successfully`
* `firestore: deploying indexes...`
* puis un `+` vert de réussite.

---

## Pourquoi ça arrive ?

* Les **règles** se déploient via un endpoint interne Firebase (OK même si l’API GCP n’était pas activée).
* Les **indexes** passent par **l’API Cloud Firestore** GCP → d’où le 403 tant qu’elle n’est pas “Enabled”.

---

## Récap express

* Tu peux **continuer sans index** (A) : `firebase deploy --only "firestore:rules"`.
* Pour réactiver le déploiement des **index** : **active l’API** (B) via le lien → reviens faire `firebase deploy --only "firestore"`.


<br/>
<br/>

# Annexe 5



# Option 1 — Lancer les émulateurs sans script

Depuis la **racine** du projet (`demoflutter1\`) :

```powershell
firebase emulators:start
```

> Ça suffit pour démarrer **Auth (9099)**, **Firestore (8080)**, **Functions (5001)** et l’**UI** (souvent sur 4000).

---

# Option 2 — Ajouter le script `serve` (recommandé pour tes étudiants)

## 1) Arborescence ASCII (où se trouvent les fichiers)

```
demoflutter1/
├─ .firebaserc
├─ firebase.json
├─ firestore.rules
├─ firestore.indexes.json
├─ package.json                 ← on ajoute "serve" ici
└─ functions/
   ├─ package.json             ← contient déjà "serve": "firebase emulators:start"
   ├─ tsconfig.json
   ├─ .eslintrc.js
   └─ src/
      ├─ index.ts
      ├─ firebase.ts
      ├─ middlewares/
      │  └─ auth.ts
      └─ controllers/
         └─ noteController.ts
```

## 2) Mets à jour **`package.json` (racine)**

Ajoute des scripts qui redirigent vers `functions/` et un `serve` global :

```json
{
  "name": "backend-demo-1-root",
  "private": true,
  "scripts": {
    "build": "npm --prefix functions run build",
    "serve": "firebase emulators:start",
    "deploy:functions": "npm --prefix functions run build && firebase deploy --only functions",
    "deploy:rules": "firebase deploy --only firestore:rules",
    "deploy:all": "npm --prefix functions run build && firebase deploy --only functions,firestore:rules"
  }
}
```


> Voici le package.json finale dans la racine 


```json
{
  "name": "backend-demo-1-root",
  "private": true,
  "scripts": {
    "build": "npm --prefix functions run build",
    "serve": "firebase emulators:start",
    "deploy:functions": "npm --prefix functions run build && firebase deploy --only functions",
    "deploy:rules": "firebase deploy --only firestore:rules",
    "deploy:all": "npm --prefix functions run build && firebase deploy --only functions,firestore:rules"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "express": "^5.1.0"
  },
  "devDependencies": {
    "@types/cors": "^2.8.19",
    "@types/express": "^5.0.3"
  }
}
```

> Tu peux créer ce fichier s’il n’existe pas, ou ajouter/compléter les `scripts` si le fichier existe.

## 3) Vérifie que **`functions/package.json`** a bien un `serve`

Il doit déjà contenir :

```json
{
  "scripts": {
    "build": "tsc",
    "serve": "firebase emulators:start",
    "deploy": "firebase deploy --only functions",
    "lint": "eslint --ext .js,.ts . || exit 0"
  }
}
```

## 4) Lance les émulateurs

Depuis la **racine** :

```powershell
npm run serve
```

---

## URLs utiles (par défaut)

* Emulator UI : `http://localhost:4000`
* Functions (local) :

  * `http://localhost:5001/<PROJECT_ID>/us-central1/api/health`
  * `http://localhost:5001/<PROJECT_ID>/us-central1/api/me`

> Remplace `<PROJECT_ID>` par `backend-demo-1`.

---

## Client web en **mode émulateurs**

Quand tu testes depuis une app web locale, connecte l’émulateur Auth :

```ts
import {initializeApp} from "firebase/app";
import {getAuth, signInWithEmailAndPassword, connectAuthEmulator} from "firebase/auth";

const app = initializeApp({
  apiKey: "dev-key",
  authDomain: "localhost",
  projectId: "backend-demo-1"
});

const auth = getAuth(app);
connectAuthEmulator(auth, "http://localhost:9099");

async function callMe() {
  const cred = await signInWithEmailAndPassword(auth, "john@doe.com", "secret123");
  const idToken = await cred.user.getIdToken();

  const res = await fetch("http://localhost:5001/backend-demo-1/us-central1/api/me", {
    headers: {Authorization: `Bearer ${idToken}`}
  });
  console.log(await res.json());
}
```

---

## cURL (local)

```bash
# Health (public)
curl http://localhost:5001/backend-demo-1/us-central1/api/health

# Créer une note (protégé)
curl -X POST "http://localhost:5001/backend-demo-1/us-central1/api/v1/notes" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <ID_TOKEN>" \
  -d "{\"title\":\"Ma première note\",\"content\":\"Bonjour\"}"
```

---

## Si tu vois encore “Missing script: serve”

* Vérifie que tu es **à la racine** (`demoflutter1\`) quand tu fais `npm run serve`.
* Vérifie que **`package.json` (racine)** contient bien `"serve": "firebase emulators:start"`.
* Alternative immédiate : exécute directement `firebase emulators:start`.


<br/>
<br/>

# Annexe 6 - Problème de port 8080

- Si **le port 8080 est déjà utilisé**, donc l’émulateur Firestore ne peut pas démarrer. Tu as deux chemins simples :



# Option A — Libérer le port 8080 (recommandé)

## 1) Trouver le processus qui occupe 8080

PowerShell (admin pas obligatoire, mais utile pour tuer un process) :

```powershell
netstat -ano | findstr :8080
```

Tu verras une ligne avec un **PID** (dernier champ). Par exemple `LISTENING ... PID`.

## 2) Identifier l’appli

```powershell
tasklist /FI "PID eq <PID_TRouvé>"
```

ou

```powershell
Get-Process -Id <PID_TRouvé>
```

## 3) Fermer ou tuer le processus

Fermer proprement l’appli si tu la connais (Docker Desktop, un autre serveur web, un ancien émulateur, etc.).
Sinon, force (en tant qu'administrateur):

```powershell
taskkill /PID <PID_TRouvé> /F
```

## 4) Relancer les émulateurs

```powershell
firebase emulators:start
```

ou avec sélection explicite (PowerShell → guillemets !) :

```powershell
firebase emulators:start --only "auth,functions,firestore"
```

---

# Option B — Changer le port de Firestore (très rapide)

Modifie **`firebase.json`** à la racine pour éviter 8080 (ex. 8081) :

```
demoflutter1/
├─ .firebaserc
├─ firebase.json          ← on modifie ici
├─ firestore.rules
└─ functions/
   └─ ...
```

## `firebase.json` (exemple corrigé)

```json
{
  "functions": {
    "source": "functions",
    "predeploy": [
      "npm --prefix \"%RESOURCE_DIR%\" run build"
    ]
  },
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "emulators": {
    "functions": {"port": 5001},
    "firestore": {"host": "127.0.0.1", "port": 8081},
    "auth": {"port": 9099},
    "ui": {"enabled": true}
  }
}
```

Ensuite :

```powershell
firebase emulators:start
```

> Si tu changes le port Firestore, **pense à connecter l’émulateur Firestore** dans ton client web (si tu en utilises un) :

```ts
import {getFirestore, connectFirestoreEmulator} from "firebase/firestore";
const db = getFirestore(app);
connectFirestoreEmulator(db, "127.0.0.1", 8081);
```

Pour **Auth**, on garde :

```ts
import {connectAuthEmulator} from "firebase/auth";
connectAuthEmulator(auth, "http://localhost:9099");
```

---

## Notes utiles

* Souvent, **Docker**, un **autre serveur web**, ou un **ancien émulateur** bloquent 8080.
* Si tu as lancé des émulateurs dans une autre fenêtre, ferme-la ou tue le PID.
* Pour Functions Admin SDK, quand tu démarres via `firebase emulators:start`, les variables d’environnement sont injectées automatiquement → Admin parle à l’émulateur. Si tu lances ton serveur Express autrement, force l’ENV `FIRESTORE_EMULATOR_HOST=127.0.0.1:8081`.

---

## Vérification rapide

1. Démarrage :

```powershell
firebase emulators:start --only "auth,functions,firestore"
```

2. UI : `http://localhost:4000`
3. Endpoints locaux :

```
http://localhost:5001/backend-demo-1/us-central1/api/health
http://localhost:5001/backend-demo-1/us-central1/api/me
```





