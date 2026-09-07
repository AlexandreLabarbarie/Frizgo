# Frizgo


**FrizGo** est une application mobile anti-gaspillage alimentaire qui aide les utilisateurs à suivre le contenu de leur frigo, à être alertés avant la péremption de leurs produits, et à valoriser ce qu'il leur reste grâce à des suggestions de recettes générées par IA.

Projet de fin de première année - **ETNA**, réalisé en équipe de 4.

---

## 📱 Fonctionnalités

### Authentification & sécurité

- Inscription / connexion par email + mot de passe (Bearer Token)
- Vérification d'email
- Gestion de session par access token
- Déconnexion

### Frigo & liste de courses

- Ajout et suppression d'aliments du frigo
- Liste de courses avec suivi des articles cochés/non cochés

### Dashboard & suivi

- Statistiques utilisateur en temps réel, connectées à l'API
- Historique alimentaire avec graphique hebdomadaire du gaspillage

### Scan & ajout de produits

- **Scanner de codes-barres** avec récupération des informations produit via **Open Food Facts** (nom, calories, catégorie)
- **Recherche manuelle par nom** avec double fallback API (recherche principale + API de secours en cas d'échec)
- **Scan de ticket de caisse** : photo ou import depuis la galerie → OCR + extraction automatique des aliments via Mistral AI → écran de révision avant ajout en masse au frigo

### Intelligence artificielle (Mistral AI)

- **Suggestions de recettes personnalisées** à partir des aliments disponibles dans le frigo
- **Estimation automatique des dates de péremption** pour chaque produit ajouté, avec un prompt calibré sur des durées de conservation réalistes (plafonné à 730 jours pour éviter les valeurs aberrantes)
- **OCR de tickets de caisse** (`mistral-ocr-latest`) suivi d'une extraction structurée des produits (`mistral-small-latest`)

---

## 🛠️ Stack technique

| Composant                 | Technologie                                          |
| ------------------------- | ---------------------------------------------------- |
| Framework backend         | AdonisJS v6 (kit API)                                |
| Langage                   | TypeScript                                           |
| Base de données           | PostgreSQL 18                                        |
| ORM                       | Lucid (`@adonisjs/lucid`)                            |
| Authentification          | Access Tokens via `@adonisjs/auth`                   |
| Validation                | VineJS (`@vinejs/vine`)                              |
| Interface DB (dev)        | Adminer                                              |
| Environnement DB          | Docker (PostgreSQL + Adminer)                        |
| Frontend mobile           | Expo / React Native, TypeScript                      |
| Styling frontend          | NativeWind (Tailwind CSS pour React Native)          |
| Intelligence artificielle | Mistral AI (recettes, estimation de péremption, OCR) |
| Données produits          | Open Food Facts API                                  |
| CI/CD                     | GitLab CI (`.gitlab-ci.yml`)                         |

---

## 👥 Équipe & répartition

| Membre     | Périmètre                                                                                                                                                 |
| ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Alex**   | Authentification & sécurité                                                                                                                               |
| **David**  | Frontend (avec Maxime) — ajout/suppression d'aliments, liste de courses, maquettes Figma                                                                  |
| **Maxime** | Frontend (avec David) — cœur de l'application                                                                                                             |
| **Lélia**  | Data, intelligence artificielle & scan — dashboard, intégration Mistral AI (recettes, péremption, OCR), scanner de codes-barres, scan de ticket de caisse |

---

## 📁 Structure du projet

```
PLI_projet_fin_1ère_année/
├── docker-compose.yml          # PostgreSQL + Adminer
├── .gitlab-ci.yml
├── README.md
│
├── backend/
│   └── FrizGo/                 # Projet AdonisJS v6
│       ├── app/
│       │   ├── controllers/    # Logique des endpoints
│       │   ├── exceptions/
│       │   ├── middleware/     # Auth middleware
│       │   ├── models/         # Modèles Lucid
│       │   ├── services/
│       │   ├── transformers/   # Formatage des réponses
│       │   └── validators/     # Validation des inputs (VineJS)
│       ├── config/
│       │   └── database.ts     # Configuration PostgreSQL
│       ├── database/
│       │   ├── migrations/
│       │   ├── factories/
│       │   └── seeders/
│       ├── start/
│       │   ├── routes.ts       # Définition des routes
│       │   ├── kernel.ts       # Middleware global
│       │   └── env.ts          # Validation des variables d'environnement
│       ├── tests/
│       ├── .env / .env.example / .env.test
│       ├── ace.js
│       ├── adonisrc.ts
│       └── package.json
│
└── frontend/                   # Application Expo / React Native
    ├── assets/
    ├── scripts/
    └── src/
        └── app/
            ├── (tabs)/
            ├── _layout.tsx
            ├── index.tsx
            ├── login.tsx
            ├── signup.tsx
            └── verify-email.tsx
        ├── components/
        ├── constants/
        ├── services/
        └── global.css
    ├── app.json
    ├── babel.config.js
    ├── tailwind.config.js
    └── package.json
```

---

## 🗄️ Base de données

### Schéma des tables

#### `users`

| Colonne                 | Type         | Description                        |
| ----------------------- | ------------ | ---------------------------------- |
| id                      | integer (PK) | Identifiant unique                 |
| full_name               | string       | Nom complet (nullable)             |
| email                   | string       | Email unique                       |
| password                | string       | Mot de passe hashé                 |
| email_verificated_at    | timestamp    | date de la verification de l'email |
| created_at / updated_at | timestamp    | Dates                              |

#### `foods` (aliments)

| Colonne                 | Type                 | Description                 |
| ----------------------- | -------------------- | --------------------------- |
| id                      | integer (PK)         | Identifiant unique          |
| user_id                 | integer (FK → users) | Propriétaire                |
| name                    | string               | Nom de l'aliment            |
| quantity                | decimal              | Quantité                    |
| unit                    | string               | Unité (unité, g, kg, ml, L) |
| category                | string               | Catégorie                   |
| expiration_date         | date                 | Date de péremption          |
| created_at / updated_at | timestamp            | Dates                       |

#### `recipes` (recettes)

| Colonne                 | Type                 | Description                |
| ----------------------- | -------------------- | -------------------------- |
| id                      | integer (PK)         | Identifiant unique         |
| user_id                 | integer (FK → users) | Propriétaire               |
| title                   | string               | Titre de la recette        |
| ingredients             | json                 | Liste des ingrédients      |
| instructions            | text                 | Étapes de préparation      |
| duration                | string               | La duree de la recette     |
| dificulty               | string               | La dificulte de la recette |
| generated_at            | timestamp            | Date de génération IA      |
| created_at / updated_at | timestamp            | Dates                      |

#### `shopping_list_items` (liste de courses)

| Colonne                 | Type                 | Description            |
| ----------------------- | -------------------- | ---------------------- |
| id                      | integer (PK)         | Identifiant unique     |
| user_id                 | integer (FK → users) | Propriétaire           |
| name                    | string               | Nom de l'item          |
| quantity                | string               | Quantité (ex : "2 kg") |
| is_checked              | boolean              | Coché ou non           |
| created_at / updated_at | timestamp            | Dates                  |

#### `food_histories` (Historique des aliments)

| Colonne                 | Type                 | Description                                       |
| ----------------------- | -------------------- | ------------------------------------------------- |
| id                      | integer (PK)         | Identifiant unique                                |
| user_id                 | integer (FK → users) | Propriétaire                                      |
| food_name               | string               | Nom de l'aliment                                  |
| category                | string               | Catégorie                                         |
| reason                  | boolean              | La raison de la suppresion (prerimer ou utiliser) |
| deleted_at              | timestamp            | Date de la suppresion du frigo                    |
| created_at / updated_at | timestamp            | Dates                                             |

#### `opt_codes` (Code de verifications)

| Colonne    | Type                 | Description                     |
| ---------- | -------------------- | ------------------------------- |
| id         | integer (PK)         | Identifiant unique              |
| user_id    | integer (FK → users) | Propriétaire                    |
| code       | varchar(6)           | Code de verification            |
| purpose    | text                 | Raison de la generation du code |
| expires_at | timestamp            | Date de l'expiration du code    |
| created_at | timestamp            | Dates                           |

### Relations

- Un `user` a plusieurs `foods`, plusieurs `recipes`, et une `shopping_list` avec plusieurs items
- Toutes les clés étrangères sont en `ON DELETE CASCADE` (suppression de l'user = suppression de toutes ses données)

---

## 🔌 API REST

### Base URL

```
http://localhost:3333/api/v1
```

### Authentification

L'API utilise des **Bearer Tokens**. Pour les routes protégées, ajouter le header :

```
Authorization: Bearer <token>
```

Le token est retourné lors de l'inscription ou de la connexion.

### Endpoints

#### Auth (public)

| Méthode | Endpoint                       | Description           |
| ------- | ------------------------------ | --------------------- |
| POST    | `/api/v1/auth/signup`          | Inscription           |
| POST    | `/api/v1/auth/login`           | Connexion             |
| POST    | `/api/v1/auth/forgot-password` | Mot de passe oublie   |
| POST    | `/api/v1/auth/reset-password`  | Reset du mot de passe |

**Body `signup` :**

```json
{
  "fullName": "Alexandre Labarbarie",
  "email": "alex@frizgo.com",
  "password": "password123",
  "passwordConfirmation": "password123"
}
```

**Réponse :**

```json
{
  "user": {
    "id": 1,
    "fullName": "Alexandre Labarbarie",
    "email": "alex@frizgo.com",
    "initials": "AL",
    "createdAt": "2026-07-06T...",
    "updatedAt": "2026-07-06T..."
  },
  "token": "oat_MQ.xxxxxxxxxxxxx"
}
```

`login` prend `email` + `password` et renvoie la même structure.

#### Account (protégé 🔒)

| Méthode | Endpoint                          | Description                      |
| ------- | --------------------------------- | -------------------------------- |
| GET     | `/api/v1/account/profile`         | Profil de l'utilisateur connecté |
| PATCH   | `/api/v1/account/profile`         | Modifie le profile               |
| PATCH   | `/api/v1/account/change-password` | Modifie le mot de passe          |
| POST    | `/api/v1/account/logout`          | Déconnexion                      |

#### Email verification (protégé 🔒)

| Méthode | Endpoint                            | Description          |
| ------- | ----------------------------------- | -------------------- |
| POST    | `/api/v1/email-verification/verify` | Vérifie le code reçu |
| POST    | `/api/v1/email-verification/resend` | Renvoie un code      |

#### Foods (protégé 🔒)

| Méthode | Endpoint            | Description                                  |
| ------- | ------------------- | -------------------------------------------- |
| GET     | `/api/v1/foods`     | Liste des aliments de l'utilisateur connecté |
| GET     | `/api/v1/foods/:id` | Détail d'un aliment                          |
| POST    | `/api/v1/foods`     | Ajout d'un aliment                           |
| PATCH   | `/api/v1/foods/:id` | Modification d'un aliment                    |
| DELETE  | `/api/v1/foods/:id` | Suppression d'un aliment                     |

#### Recipes (protégé 🔒)

| Méthode | Endpoint                              | Description                                       |
| ------- | ------------------------------------- | ------------------------------------------------- |
| GET     | `/api/v1/recipes/suggest`             | Suggestions de recettes générées par Mistral AI   |
| GET     | `/api/v1/recipes/estimate-expiration` | Estimation de la date d'expiration par Mistral AI |
| GET     | `/api/v1/recipes`                     | Liste des recettes sauvegardées                   |
| GET     | `/api/v1/recipes/:id`                 | Détail d'une recette                              |
| POST    | `/api/v1/recipes`                     | Sauvegarde d'une recette                          |
| DELETE  | `/api/v1/recipes/:id`                 | Suppression d'une recette                         |

#### Shopping list (protégé 🔒)

| Méthode | Endpoint                    | Description                        |
| ------- | --------------------------- | ---------------------------------- |
| GET     | `/api/v1/shopping-list`     | Liste des articles                 |
| POST    | `/api/v1/shopping-list`     | Ajout d'un article                 |
| PATCH   | `/api/v1/shopping-list/:id` | Modification / toggle `is_checked` |
| DELETE  | `/api/v1/shopping-list/:id` | Suppression d'un article           |

#### Receipts (protégé 🔒)

| Méthode | Endpoint                | Description                                                      |
| ------- | ----------------------- | ---------------------------------------------------------------- |
| POST    | `/api/v1/receipts/scan` | OCR + extraction des aliments d'un ticket de caisse (Mistral AI) |

#### Stats (protégé 🔒)

| Méthode | Endpoint        | Description                    |
| ------- | --------------- | ------------------------------ |
| GET     | `/api/v1/stats` | Statistiques de l'utilisateurs |

#### History (protégé 🔒)

| Méthode | Endpoint                  | Description                           |
| ------- | ------------------------- | ------------------------------------- |
| GET     | `/api/v1/history`         | Historique de l'utilisateur           |
| POST    | `/api/v1/recehistoryipts` | Ajout d'un aliments dans l'historique |
| GET     | `/api/v1/history/weekly`  | Semaine de gaspillage                 |

> La documentation interactive Swagger est disponible sur `/docs` une fois le serveur lancé.

---

## 🚀 Installation

### Prérequis

- Node.js >= 24
- Docker Desktop
- npm
- Une clé API [Mistral AI](https://mistral.ai)

### 1. Configurer les variables d'environnement

#### BACK :

```
cd backend/FrizGo
cp .env.example .env
```

```
TZ=UTC
PORT=3333
HOST=localhost
NODE_ENV=development
LOG_LEVEL=info
APP_KEY=<généré automatiquement>
APP_URL=http://localhost:3333
SESSION_DRIVER=cookie
DB_HOST=
DB_PORT=
DB_USER=
DB_PASSWORD=
DB_DATABASE=
DRIVE_DISK=
QUEUE_DRIVER=
MISTRAL_API_KEY=<ta clé Mistral>
```

#### FRONT:

```
cd frontend
cp .env.example .env
```

```
EXPO_PUBLIC_API_URL=http://localhost:3333/api/v1
```

Remplacer localhost par l'ip de la machine sur laquel est host le backend

### 2. Lancer la base de données

Depuis la racine du repo (dossier contenant `docker-compose.yml`) :

```bash
docker compose up --build
```

Cela démarre :

- PostgreSQL sur `localhost:5433`
- Adminer (interface web DB) sur `http://localhost:8081`

### 3. Backend

```bash
cd backend/FrizGo
npm install
node ace migration:run
node ace db:seed        # optionnel — données de test
npm run dev
```

Le serveur démarre sur `http://localhost:3333`, la doc Swagger sur `http://localhost:3333/docs`.

### 4. Frontend

```bash
cd frontend
npm install
npx expo start --go
```

> ⚠️ Utiliser bien `npx expo start --go` (la commande de base `npx expo start` seule renvoie une erreur "aucune donnée trouvée" sur ce projet).

> ⚠️ Sur iOS, le scan de code-barres via `expo-camera` (`CameraView`) a posé des soucis de compatibilité dans Expo Go ; le fonctionnement a été validé sur Android et finalement corrigé sur iOS après plusieurs itérations.

---

## 🖥️ Accès Adminer (interface web DB)

- URL : `http://localhost:8081`
- Serveur : `db`
- Utilisateur : `frizgo`
- Mot de passe : `frizgo`
- Base : `frizgo`

---

## 🔧 Commandes utiles (backend)

```bash
npm run dev                      # Serveur en mode développement (HMR)
node ace migration:run           # Lancer les migrations
node ace migration:rollback      # Rollback des migrations
node ace db:seed                 # Seeder la base de données
node ace make:migration <nom>    # Créer une migration
node ace make:model <nom>        # Créer un modèle
node ace make:controller <nom>   # Créer un controller
node ace make:seeder <nom>       # Créer un seeder
node ace make:factory <nom>      # Créer une factory
node ace tinker                  # REPL interactif
```

---

## 🧠 Défis techniques résolus

### Lélia — Data, IA & scan

- Migration de Gemini vers Mistral AI suite à des restrictions de quota sur le compte scolaire
- Scan de code-barres non fonctionnel sur iOS au démarrage du projet, résolu après plusieurs itérations (passage à `CameraView`)
- Correction du mapping des catégories Open Food Facts (faux positifs, ex. un soda classé comme un fruit)
- Résolution d'une race condition provoquant des erreurs `429` lors du scan (verrouillage synchrone des requêtes)
- Fiabilisation de l'estimation des dates de péremption par Mistral AI (prompt enrichi + plafond de sécurité à 730 jours)

### Alex — Authentification & sécurité

- Bugs de fuseau horaire faisant apparaître des aliments comme périmés alors qu'ils ne l'étaient pas (et inversement)
- **Solution :** harmonisation de tout le projet sur un même fuseau horaire, et calcul de la péremption des produits au jour près plutôt qu'à l'heure/minute près

### David — Frontend (avec Maxime)

- Recherche longue du bon logo lors du travail sur la maquette, avant de trouver la version qui convenait
- Bugs sur les boutons d'ajout et de suppression de produits (le produit ne s'ajoutait pas, ou ne se supprimait pas), rendant l'ajout au frigo compliqué à fiabiliser

### Maxime — Frontend (avec David)

- Difficultés avec le rendu des SVG, qui ne s'affichaient pas comme prévu
- Travail conséquent sur la liste de courses
- Gros travail de peaufinage sur le design : beaucoup d'attention portée aux détails pour que tout soit soigné

---

## 🗺️ Statut du projet

Application fonctionnelle de bout en bout : authentification, frigo, liste de courses, dashboard, recettes IA, scan code-barres et scan de ticket de caisse.

Projet réalisé dans le cadre de la première année Bachelor Informatique à l'ETNA - soutenance le **24 juillet 2026**.
