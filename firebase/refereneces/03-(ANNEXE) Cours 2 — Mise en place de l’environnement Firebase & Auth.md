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

