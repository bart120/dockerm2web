# TP Docker Avancé – Développement Node.js sans Node en local (Docker-only)
Master 2 – Développement Web

## Objectifs pédagogiques

À l’issue de ce TP, l’étudiant sera capable de :

- Utiliser Docker comme environnement unique de développement
- Initialiser un projet Node.js uniquement via Docker
- Construire une image Docker orientée développement
- Optimiser le cache Docker
- Utiliser correctement bind mounts, volumes et variables d’environnement
- Lancer plusieurs conteneurs sans Docker Compose
- Déboguer une application Node.js dans Docker

## Contexte

Dans cette équipe, aucune dépendance de runtime (Node.js, npm, etc.) n’est installée localement.
Docker est l’unique environnement d’exécution autorisé.

Vous devez être capable de créer un projet, installer les dépendances, lancer l’API, modifier le code et déboguer, uniquement avec Docker.

## Pré-requis

- Docker installé et fonctionnel
- Accès à un terminal
- Aucun Node.js installé ou utilisé localement

## Partie 0 – Création de l’arborescence

Arborescence attendue :

```txt
docker-advanced-dev/
├── src/
│   ├── index.js
│   └── routes.js
├── Dockerfile
└── .dockerignore
```

## Partie 1 – Initialiser Node.js uniquement via Docker

```bash
docker run -it --rm   -v "$(pwd)":/app   -w /app   node:20-alpine sh
```

Dans le conteneur :

```bash
npm init -y
npm install express
npm install -D nodemon
```

Modifier le package.json :
- ajouter "type": "module"
- ajouter le script "dev": "nodemon src/index.js"

Quitter le conteneur avec `exit`.

## Partie 2 – Création de l’API Express

### src/routes.js
```js
import { Router } from "express";

const router = Router();

router.get("/health", (req, res) => {
  res.json({ status: "ok" });
});

export default router;
```

### src/index.js
```js
import express from "express";
import routes from "./routes.js";

const app = express();
const PORT = process.env.PORT || 3000;

app.use("/api", routes);

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Test via Docker :

```bash
docker run -it --rm   -v "$(pwd)":/app   -w /app   -p 3000:3000   node:20-alpine sh
```

Puis :

```bash
npm run dev
```

## Partie 3 – Dockerfile orienté développement

Contraintes :
- Base node:20-alpine
- npm ci
- Cache optimisé
- Code monté en bind mount
- node_modules isolé

Livrable : Dockerfile

## Partie 4 – Lancement en mode développement

```bash
docker run --rm   -p 3000:3000   -v "$(pwd)":/app   -v /app/node_modules   <nom_image>
```

## Partie 5 – Lancer deux instances

```bash
docker run -d --name api1 -p 3001:3000 ...
docker run -d --name api2 -p 3002:3000 ...
```

## Partie 6 – Variables d’environnement

```bash
docker run --rm   -e PORT=4000   -p 4000:4000   -v "$(pwd)":/app   -v /app/node_modules   <nom_image>
```

## Partie 7 – Debug Node.js

- Lancer Node avec --inspect=0.0.0.0:9229
- Exposer le port 9229
- Debug fonctionnel depuis l’IDE

