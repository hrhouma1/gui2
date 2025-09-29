<img width="336" height="647" alt="image" src="https://github.com/user-attachments/assets/a36f6873-d266-4f1a-98ba-6c418852febe" />


# functions\src\controllers\noteController.ts

```bash
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


# functions\src\middlewares\auth.ts

```bash
import {Request, Response, NextFunction} from "express";
import {auth} from "../firebase";

export async function requireAuth(
  req: Request,
  res: Response,
  next: NextFunction
): Promise<void> {
  const header = req.headers.authorization ?? "";
  const token = header.startsWith("Bearer ") ? header.slice(7) : "";

  if (!token) {
    res.status(401).json({error: "Missing Bearer token"});
    return;
  }

  try {
    const decoded = await auth.verifyIdToken(token);
    (req as any).uid = decoded.uid;
    next();
    return;
  } catch {
    res.status(401).json({error: "Invalid or expired token"});
    return;
  }
}

```


# functions\src\firebase.ts

```bash
import * as admin from "firebase-admin";

if (!admin.apps.length) {
  admin.initializeApp();
}

export const auth = admin.auth();
export const db = admin.firestore();
```



# functions\src\index.ts

```bash
import * as functions from "firebase-functions";
import express from "express";
import cors from "cors";
import {requireAuth} from "./middlewares/auth";
import {createNote} from "./controllers/noteController";

const app = express();
app.use(cors());
app.use(express.json());

// Public
app.get("/health", (_req, res) => {
  res.status(200).json({ok: true, service: "api", version: "v1"});
});

// Protégé
app.get("/me", requireAuth, (req, res) => {
  const uid = (req as any).uid as string;
  res.status(200).json({uid});
});

// >>>>>>>>>>>> LA ROUTE QUI NOUS INTÉRESSE <<<<<<<<<<<<
app.post("/v1/notes", requireAuth, createNote);

export const api = functions.https.onRequest(app);

```


# functions\.eslintrc.js

```bash
module.exports = {
  root: true,
  env: {
    es6: true,
    node: true,
  },
  extends: [
    "eslint:recommended",
    "plugin:import/errors",
    "plugin:import/warnings",
    "plugin:import/typescript",
    "google",
    "plugin:@typescript-eslint/recommended",
  ],
  parser: "@typescript-eslint/parser",
  parserOptions: {
    project: ["tsconfig.json", "tsconfig.dev.json"],
    sourceType: "module",
  },
  ignorePatterns: [
    "/lib/**/*", // Ignore built files.
    "/generated/**/*", // Ignore generated files.
  ],
  plugins: [
    "@typescript-eslint",
    "import",
  ],
  rules: {
    "quotes": ["error", "double"],
    "import/no-unresolved": 0,
    "indent": ["error", 2],
  },
};

```


# functions\package.json

```bash
{
  "name": "functions",
  "scripts": {
    "lint": "eslint --ext .js,.ts .",
    "build": "tsc",
    "build:watch": "tsc --watch",
    "serve": "npm run build && firebase emulators:start --only functions",
    "shell": "npm run build && firebase functions:shell",
    "start": "npm run shell",
    "deploy": "firebase deploy --only functions",
    "logs": "firebase functions:log"
  },
  "engines": {
    "node": "22"
  },
  "main": "lib/index.js",
  "dependencies": {
    "firebase-admin": "^12.7.0",
    "firebase-functions": "^6.0.1"
  },
  "devDependencies": {
    "@typescript-eslint/eslint-plugin": "^5.12.0",
    "@typescript-eslint/parser": "^5.12.0",
    "eslint": "^8.9.0",
    "eslint-config-google": "^0.14.0",
    "eslint-plugin-import": "^2.25.4",
    "firebase-functions-test": "^3.1.0",
    "typescript": "^5.7.3"
  },
  "private": true
}

```


# .firebaserc

```bash
{
  "projects": {
    "default": "backend-demo-1"
  }
}

```



# firebase.json

```bash
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



# firestore.indexes.json

```bash
{
  "indexes": [],
  "fieldOverrides": []
}

```



# firestore.rules

```bash
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


# package.json

```bash
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





# Commande 1 - Création du token

```powershell
$login = Invoke-RestMethod -Method Post `
  -Uri "http://localhost:9099/identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=anything" `
  -ContentType "application/json" `
  -Body '{"email":"john@doe.com","password":"secret123","returnSecureToken":true}'

$token = $login.idToken
$token
```

# Commande 2 - Appeler ta route protégée `/me` (Functions Emulator)

```powershell
Invoke-RestMethod `
  -Uri "http://localhost:5001/backend-demo-1/us-central1/api/me" `
  -Headers @{ Authorization = "Bearer $token" }
```

> Ça doit répondre `{ "uid": "..." }`.

> Variante cURL “pure” (CMD/PowerShell) :

# Commande 3 - Créer l’utilisateur dans **l’émulateur**

```powershell
# crée john@doe.com dans l'Auth Emulator (clé = anything)
Invoke-RestMethod -Method Post `
  -Uri "http://localhost:9099/identitytoolkit.googleapis.com/v1/accounts:signUp?key=anything" `
  -ContentType "application/json" `
  -Body '{"email":"john@doe.com","password":"secret123","returnSecureToken":true}'
```

# Commande 4




##  Méthode 1 — Tout en ligne de commande (PowerShell friendly)

### 1) Créer l’utilisateur dans **l’émulateur**

```powershell
# crée john@doe.com dans l'Auth Emulator (clé = anything)
Invoke-RestMethod -Method Post `
  -Uri "http://localhost:9099/identitytoolkit.googleapis.com/v1/accounts:signUp?key=anything" `
  -ContentType "application/json" `
  -Body '{"email":"john@doe.com","password":"secret123","returnSecureToken":true}'
```

### 2) Se connecter → récupérer l’ID token

```powershell
$login = Invoke-RestMethod -Method Post `
  -Uri "http://localhost:9099/identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=anything" `
  -ContentType "application/json" `
  -Body '{"email":"john@doe.com","password":"secret123","returnSecureToken":true}'

$token = $login.idToken
$token
```

### 3) Appeler ta route protégée `/me` (Functions Emulator)

```powershell
Invoke-RestMethod `
  -Uri "http://localhost:5001/backend-demo-1/us-central1/api/me" `
  -Headers @{ Authorization = "Bearer $token" }
```

> Ça doit répondre `{ "uid": "..." }`.

> Variante cURL “pure” (CMD/PowerShell) :

```bat
curl -X POST "http://localhost:9099/identitytoolkit.googleapis.com/v1/accounts:signUp?key=anything" ^
  -H "Content-Type: application/json" ^
  -d "{\"email\":\"john@doe.com\",\"password\":\"secret123\",\"returnSecureToken\":true}"

curl -X POST "http://localhost:9099/identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=anything" ^
  -H "Content-Type: application/json" ^
  -d "{\"email\":\"john@doe.com\",\"password\":\"secret123\",\"returnSecureToken\":true}"
```

(prends `idToken` de la 2ᵉ réponse et utilise-le dans `Authorization: Bearer <ID_TOKEN>`)



# Méthode 2 — Par l’UI des Émulateurs (clic)

1. Ouvre `http://localhost:4000` → **Authentication** → **Add user** → saisis `john@doe.com` / `secret123`.
2. Reprends **l’étape 2** ci-dessus (signIn via REST) pour obtenir l’`idToken`.
3. Appelle `/me` avec le header `Authorization: Bearer <ID_TOKEN>`.



# Méthode 3 — Dans la console Dev du navigateur (JS)

```js
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-app.js";
import { getAuth, signInWithEmailAndPassword, connectAuthEmulator } from "https://www.gstatic.com/firebasejs/10.12.2/firebase-auth.js";

const app = initializeApp({ apiKey: "dev", authDomain: "localhost", projectId: "backend-demo-1" });
const auth = getAuth(app);
connectAuthEmulator(auth, "http://localhost:9099");

// si l'utilisateur n'existe pas encore, crée-le une fois via l'UI ou la route signUp
const cred = await signInWithEmailAndPassword(auth, "john@doe.com", "secret123");
const idToken = await cred.user.getIdToken();
console.log("ID_TOKEN =", idToken);

const me = await fetch("http://localhost:5001/backend-demo-1/us-central1/api/me", {
  headers: { Authorization: `Bearer ${idToken}` }
});
console.log(await me.json());
```

---

## ⚠️ Petits pièges courants

* **Toujours l’émulateur !** Tes URLs doivent pointer sur `http://localhost:9099/...identitytoolkit...` quand tu es en local.
* **Compte inexistant → `EMAIL_NOT_FOUND`** : crée d’abord l’utilisateur (signUp ou UI).
* **`Missing Bearer token`** : le navigateur n’envoie aucun header. Utilise Postman/cURL/JS avec
  `Authorization: Bearer <ID_TOKEN>`.
* **Ports** : Functions (`5001`), Auth (`9099`), Firestore (`8080` ou celui que tu as défini), UI (`4000`).
* Si tu as changé le port Firestore (ex. `8081`), pense à `connectFirestoreEmulator(db, "127.0.0.1", 8081)` côté client.




# Commande 5

```js
node --version
npm install -g firebase-tools
firebase --version
firebase login
firebase emulators:start 
```


# Commande 6

```js
firebase deploy --only firestore:rules
firebase init emulators
npm run serve
firebase emulators:start  
```

# Commande 7



# Commande 8
