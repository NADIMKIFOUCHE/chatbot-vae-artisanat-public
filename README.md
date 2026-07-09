<div align="center">

# 🚲⚡ VAE Assistant — Autodiagnostic & mise en relation

**Un assistant IA conversationnel qui diagnostique les pannes de vélos à assistance électrique, estime le coût de réparation, et transmet la demande à un atelier — le tout en quelques messages.**

[![Next.js](https://img.shields.io/badge/Next.js-16-000000?logo=nextdotjs&logoColor=white)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![LangChain](https://img.shields.io/badge/LangChain-LangGraph-1C3C3C?logo=langchain&logoColor=white)](https://www.langchain.com/)
[![Gemini](https://img.shields.io/badge/Google-Vertex_AI_·_Gemini-4285F4?logo=googlecloud&logoColor=white)](https://cloud.google.com/vertex-ai)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)

</div>

---

## 🎬 Démo


https://github.com/user-attachments/assets/42b57afa-d324-4591-ab65-745bba33cabb
https://github.com/user-attachments/assets/964f41d0-d465-4b05-b05d-531d6e9cb7dd
https://github.com/user-attachments/assets/647b019d-c263-44aa-9e63-b083618ae3fe

<div align="center">

<!-- Remplacez par votre capture / GIF -->
<img src="docs/demo.gif" alt="Démo VAE Assistant" width="780"/>

| Diagnostic en streaming | Formulaire & photos | Email artisan |
|:---:|:---:|:---:|
| <img src="docs/screen-chat.png" width="250"/> | <img src="docs/screen-form.png" width="250"/> | <img src="docs/screen-email.png" width="250"/> |

</div>

---

## 📖 Sommaire

- [Le problème](#-le-problème)
- [Fonctionnalités](#-fonctionnalités)
- [Comment ça marche — le pipeline IA](#-comment-ça-marche--le-pipeline-ia)
- [Parcours utilisateur](#-parcours-utilisateur)
- [Stack technique](#-stack-technique)
- [Architecture](#-architecture)
- [Sécurité](#-sécurité)
- [Lancer en local](#-lancer-en-local)
- [Configuration](#-configuration)
- [Roadmap](#-roadmap)

---

## 🎯 Le problème

Quand un VAE tombe en panne, l'utilisateur ne sait souvent **ni ce qu'il a**, ni **combien ça va coûter**, ni **à qui s'adresser**. Côté atelier, on reçoit des demandes floues (« mon vélo marche plus ») qui obligent à un long échange avant même de savoir si l'intervention vaut le déplacement.

**VAE Assistant** comble ce trou : un dialogue guidé par IA établit un pré-diagnostic structuré, une estimation de coût, et envoie à l'atelier un **dossier complet et exploitable** — pendant que le client repart avec une réponse claire.

---

## ✨ Fonctionnalités

### 💬 Conversation & IA
- **Chat en streaming** token par token (Server-Sent Events) — réponse fluide, en temps réel.
- **Pipeline en 2 agents** : un agent qui *pose les bonnes questions*, puis un agent qui *diagnostique et chiffre*.
- **Titre de conversation auto-généré** dès le premier échange.
- **Rendu Markdown** complet (tableaux, listes, code avec coloration syntaxique).

### 📋 Diagnostic & demande
- **Estimation des coûts** structurée (prestation / fourchette / total).
- **Formulaire de contact** déclenché automatiquement quand le diagnostic est complet.
- **Photos de la panne** par **glisser-déposer** ou **collage (Ctrl+V)** — compression côté client, optionnel.
- **Génération d'emails HTML** stylés : dossier complet à l'atelier (diagnostic + coûts + photos en pièces jointes) et **accusé de réception au client** (sans l'estimation tarifaire).
- **Verrouillage de la demande** : une fois transmise, la conversation devient **statique** (état persisté en base).

### 👤 Comptes & historique
- **Authentification** (inscription / connexion) par **JWT**, mots de passe hachés **bcrypt**.
- **Profil** pré-rempli (nom, prénom, téléphone, adresse…) réinjecté dans le formulaire.
- **Historique des conversations** persistant, sidebar avec sélection / suppression.
- **Mode anonyme** : utilisable sans compte.

### 🎨 Expérience
- **Thème clair / sombre** avec anti-flash (le thème s'applique avant l'hydratation).
- **UI animée** (framer-motion), responsive, design soigné.

---

## 🧠 Comment ça marche — le pipeline IA

Le cœur du produit est un **pipeline séquentiel à 2 phases** orchestré avec **LangChain / LangGraph** sur **Google Vertex AI (Gemini)**.

```
Utilisateur
    │  « Mon VAE n'avance plus quand j'appuie sur l'assistance »
    ▼
┌─────────────────────────────────────────────┐
│  PHASE 1 — Agent « Renseignements »          │
│  • Pose des questions jusqu'à avoir assez    │
│    d'infos (modèle, symptômes, contexte…)    │
│  • Émet <titre>…</titre> (titre de la conv)  │
│  • Quand prêt → produit un <RAPPORT_PRET>     │
│    structuré (résumé interne pour l'agent 2) │
└─────────────────────────────────────────────┘
    │  rapport structuré
    ▼
┌─────────────────────────────────────────────┐
│  PHASE 2 — Agent « Diagnostic »               │
│  • Diffuse en temps réel (astream)            │
│  • Diagnostic probable + causes               │
│  • Estimation des coûts (tableau)             │
│  • Déclenche l'affichage du formulaire        │
└─────────────────────────────────────────────┘
    │
    ▼
Formulaire de contact → Emails (atelier + client) → Demande verrouillée
```

- **Tant que l'agent 1 manque d'infos**, il renvoie sa question au visiteur (pas de formulaire).
- **Le `<RAPPORT_PRET>` n'est jamais montré** au client : c'est le hand-off interne entre les deux agents.
- **Le streaming SSE** transporte trois types d'événements : `delta` (texte), `title` (titre), `done` (fin + flag formulaire).

---

## 🧭 Parcours utilisateur

```
1. Décrire la panne ──► 2. Répondre aux questions de l'IA ──► 3. Recevoir le diagnostic + l'estimation
                                                                          │
6. Demande verrouillée ◄── 5. Emails envoyés (atelier + client) ◄── 4. Remplir le formulaire (+ photos)
```

---

## 🛠 Stack technique

| Domaine | Technologies |
|---|---|
| **Frontend** | Next.js 16 (App Router, Turbopack) · React 19 · TypeScript · Tailwind CSS v4 · framer-motion · lucide-react · react-markdown + remark-gfm · highlight.js |
| **Backend** | FastAPI · SQLAlchemy (async) · Pydantic v2 · SSE streaming · python-jose (JWT) · bcrypt |
| **IA** | LangChain · LangGraph · Google Vertex AI — Gemini (`gemini-2.5-flash` par défaut) |
| **Base de données** | PostgreSQL 16 (`pgvector/pgvector:pg16`) |
| **Emails** | SMTP (Gmail / OVH / Outlook…) · templates HTML · pièces jointes images |
| **Infra** | Docker & Docker Compose (profils *dev* hot-reload et *prod*) |

---

## 🏗 Architecture

```

├── frontend/                      # Application Next.js
│   └── src/
│       ├── app/                   # Routes, layout, error boundary, thème
│       ├── components/
│       │   ├── chat/              # ChatWindow, ChatInput, MessageBubble, ContactForm
│       │   ├── sidebar/           # Historique des conversations, profil
│       │   └── auth/              # Modale de connexion / inscription
│       └── lib/                   # api.ts (client HTTP/SSE), auth, theme, utils
│
├── backend/                       # API FastAPI
│   └── app/
│       ├── main.py                # App, CORS, lifespan, garde-fous démarrage
│       ├── db.py                  # Modèles SQLAlchemy + migrations idempotentes
│       ├── core/security.py       # JWT, hachage bcrypt
│       ├── schemas/               # Schémas Pydantic
│       └── api/
│           ├── auth.py            # register / login / profil
│           ├── chat.py            # /chat/stream (SSE)
│           ├── conversations.py   # historique, messages, suppression
│           └── contact.py         # envoi emails, rate-limit, validation
│
├── ai/                            # Pipeline IA
│   ├── pipeline/router.py         # Orchestration 2 phases
│   ├── agents/                    # renseignements.py · diagnostic.py
│   ├── llm/                       # Connexion Vertex AI / Gemini
│   └── prompts/                   # Prompts système
│
├── docker-compose.yaml            # Stack prod
└── docker-compose.dev.yaml        # Override dev (hot-reload frontend)
```

**Modèle de données (simplifié)**

```
User ──1─────n── Conversation ──1─────n── Message
                     │
                     ├─ show_form         (formulaire à afficher)
                     └─ demande_transmise  (demande verrouillée)
```

---

## 🔒 Sécurité

Le projet intègre plusieurs **filets de sécurité** côté API :

- 🔑 **JWT** signés, mots de passe **bcrypt** ; refus de démarrer en production avec un secret par défaut.
- 🛡️ **Anti-IDOR** : les messages d'une conversation possédée ne sont lisibles que par son propriétaire.
- 🚦 **Rate-limiting** sur l'envoi de formulaire (anti-spam d'emails).
- 🧼 **Échappement HTML** des champs utilisateur dans les emails (anti-injection).
- ✅ **Validation Pydantic** : format d'email, bornes de taille (messages, photos, payload base64).
- 🌐 **CORS** restreint par variable d'environnement.

---

## 🚀 Lancer en local

> Prérequis : **Docker** + **Docker Compose**, et un accès **Google Vertex AI** (fichier `credentials.json`).

```bash
# 1. Cloner
git clone <repo-url> && cd

# 2. Configurer l'environnement
cp .env.example .env        # puis éditer (voir section Configuration)

# 3. Lancer en mode développement (hot-reload frontend)
docker compose -f docker-compose.yaml -f docker-compose.dev.yaml up -d --build
```

| Service | URL |
|---|---|
| Frontend | http://localhost:3000 |
| API (docs Swagger) | http://localhost:8000/docs |
| PostgreSQL | `localhost:5432` |

> En **production** : `docker compose up -d --build` (build figé, sans hot-reload).

---

## ⚙️ Configuration

Variables principales du fichier `.env` :

```env
# Base de données
POSTGRES_USER=vae
POSTGRES_PASSWORD=********
POSTGRES_DB=vae

# Backend
JWT_SECRET=<chaîne-aléatoire-longue>          # OBLIGATOIRE en prod
ALLOWED_ORIGINS=http://localhost:3000

# IA — Google Vertex AI
GEMINI_MODEL=gemini-2.5-flash
GOOGLE_CLOUD_PROJECT=<votre-projet-gcp>
GOOGLE_CLOUD_LOCATION=us-central1
GCP_CREDENTIALS_PATH=./credentials.json

# Email (SMTP) — ex. Gmail avec « mot de passe d'application »
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=<votre-email>
EMAIL_PASSWORD=<mot-de-passe-application-16-car>
FROM_EMAIL=<votre-email>
ARTISAN_EMAIL=<email-qui-reçoit-les-demandes>

# Frontend
NEXT_PUBLIC_API_URL=http://localhost:8000
```

> 💡 **Gmail** : activez la validation en 2 étapes puis générez un **mot de passe d'application** (le mot de passe du compte ne fonctionne pas en SMTP).

---

## 🗺 Roadmap

- [ ] Suite de tests automatisés (pipeline IA, endpoints, parcours formulaire)
- [ ] Tableau de bord atelier (suivi des demandes reçues)
- [ ] Base de connaissances pannes/coûts (RAG via `pgvector`)
- [ ] Prise de rendez-vous intégrée (créneaux atelier)
- [ ] Notifications (email/SMS) de suivi de demande
- [ ] Internationalisation

---

<div align="center">

**VAE Assistant** — pré-diagnostic sans engagement, du message à l'atelier.

_Projet de démonstration. Les estimations sont indicatives ; un examen en atelier reste nécessaire pour un devis définitif._

</div>
