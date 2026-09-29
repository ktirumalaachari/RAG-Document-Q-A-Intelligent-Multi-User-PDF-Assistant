# 🧠 RAG Document Q&A — Intelligent Multi-User PDF Assistant

<div align="center">

![Project Banner](https://img.shields.io/badge/AI%20Architecture-RAG%20System-6366f1?style=for-the-badge&logo=openai&logoColor=white)
![Google Gemini](https://img.shields.io/badge/LLM-Google%20Gemini%20Flash-4285F4?style=for-the-badge&logo=google&logoColor=white)
![MongoDB Atlas](https://img.shields.io/badge/Vector%20Store-MongoDB%20Atlas-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![React 19](https://img.shields.io/badge/Frontend-React%2019%20%2B%20Vite-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

<p align="center">
  <b>A production-ready, full-stack Retrieval-Augmented Generation (RAG) platform.</b><br/>
  Upload PDF documents, extract and chunk text with exact page tracking, generate 3072-dimensional vector embeddings, and query documents with Google Gemini providing grounded, cited answers with match scores.
</p>

[🚀 Live Demo](https://rag-document-qa-theta.vercel.app/) • [⚡ Backend API](https://rag-document-qa-backend.vercel.app/) • [📑 API Docs](#-api-reference) • [🛠️ Setup Guide](#-complete-environment--local-setup-guide)

</div>

---

## 📑 Table of Contents

- [🧠 RAG Document Q\&A — Intelligent Multi-User PDF Assistant](#-rag-document-qa--intelligent-multi-user-pdf-assistant)
  - [📑 Table of Contents](#-table-of-contents)
  - [🌟 Overview](#-overview)
  - [⚡ Key Features](#-key-features)
    - [🔐 1. Authentication \& Multi-Tenant Isolation](#-1-authentication--multi-tenant-isolation)
    - [📄 2. Smart PDF Ingestion \& Processing](#-2-smart-pdf-ingestion--processing)
    - [🤖 3. Vector Retrieval \& Grounded Q\&A (RAG)](#-3-vector-retrieval--grounded-qa-rag)
  - [🛠️ Tech Stack \& Icons](#️-tech-stack--icons)
    - [Frontend](#frontend)
    - [Backend \& AI Pipeline](#backend--ai-pipeline)
  - [🏗️ System Architecture \& Data Flow](#️-system-architecture--data-flow)
  - [📂 Project Directory Structure](#-project-directory-structure)
  - [📸 Screenshots](#-screenshots)
    - [1. Account Authentication](#1-account-authentication)
    - [2. PDF Ingestion \& Duplicate Protection](#2-pdf-ingestion--duplicate-protection)
    - [3. Grounded Q\&A with Citation Scores](#3-grounded-qa-with-citation-scores)
  - [🔑 Required Environment Variables](#-required-environment-variables)
    - [1. Backend (`backend/.env`)](#1-backend-backendenv)
    - [2. Frontend (`frontend/.env`)](#2-frontend-frontendenv)
  - [🚀 Complete Environment \& Local Setup Guide](#-complete-environment--local-setup-guide)
    - [1. Prerequisite Accounts \& Links](#1-prerequisite-accounts--links)
    - [2. How to Obtain Every Credential](#2-how-to-obtain-every-credential)
      - [A. Google Gemini API Key (`GEMINI_API_KEY`)](#a-google-gemini-api-key-gemini_api_key)
      - [B. MongoDB Atlas Connection String (`MONGODB_URI`)](#b-mongodb-atlas-connection-string-mongodb_uri)
      - [C. JWT Secret (`JWT_SECRET`)](#c-jwt-secret-jwt_secret)
    - [3. Local Backend Configuration](#3-local-backend-configuration)
    - [4. Local Frontend Configuration](#4-local-frontend-configuration)
  - [🔍 MongoDB Atlas Vector Search Setup](#-mongodb-atlas-vector-search-setup)
    - [Step-by-Step Index Creation:](#step-by-step-index-creation)
  - [📡 API Reference](#-api-reference)
  - [☁️ Deployment on Vercel](#️-deployment-on-vercel)
    - [Project 1: Backend API](#project-1-backend-api)
    - [Project 2: Frontend Client](#project-2-frontend-client)
  - [🔄 Cloning \& Personalization Guide](#-cloning--personalization-guide)
  - [📜 License](#-license)
  - [👨‍💻 Author](#-author)

---

## 🌟 Overview

**RAG Document Q&A** allows users to securely register, upload PDF documents, and converse with their specific files. Unlike generic chatbots, every answer is synthesized **strictly from the retrieved chunks of that document**, eliminating hallucinations and providing transparent source citations with page numbers and cosine similarity scores.

Documents and vector embeddings are partitioned strictly per user account (`userId`), ensuring complete data privacy and isolation in a multi-tenant setup.

---

## ⚡ Key Features

### 🔐 1. Authentication & Multi-Tenant Isolation

- **Secure Auth Pipeline:** Signup, Login, and Session recovery with JWT (`Authorization: Bearer <token>`).
- **PBKDF2 Password Hashing:** Cryptographically secure salt & iterations for password storage.
- **Strict Data Scoping:** Every document and embedding chunk is scoped to `userId`. Cross-user access is impossible.
- **Session Restoration:** Instant verification on browser refresh using `GET /api/auth/me`.
- **Automatic Logout:** Auto-terminates session in the UI on `401 Unauthorized` responses.

### 📄 2. Smart PDF Ingestion & Processing

- **PDF Extraction:** Built with `pdfjs-dist` for precise page-by-page text parsing.
- **Sliding-Window Chunking:** Overlapping text chunks (size: 1,000 characters, overlap: 200 characters) preserving contextual integrity across chunk boundaries.
- **High-Dimensional Embeddings:** Embedded via Google Gemini (`gemini-embedding-2`) generating **3072-dimensional vectors** in batches of 5.
- **Deduplication Engine:** SHA-256 content hashing per user prevents redundant uploads (renaming the file does not bypass detection).
- **Graceful Error Handling:** Rejects scanned/empty PDFs without text and maps MongoDB duplicate keys cleanly to HTTP `409 Conflict`.
- **Safety Limits:** 20 documents max per user account.

### 🤖 3. Vector Retrieval & Grounded Q&A (RAG)

- **Conversational Intent Classification:** Lightweight intent classifier (`gemini-3.1-flash-lite`) intercepts greetings and compliments to save vector compute.
- **Vector Search:** MongoDB Atlas `$vectorSearch` leveraging cosine similarity over the dedicated `autoembed_index`.
- **Isolated Document Filtering:** Retrieves top-5 most relevant chunks filtered strictly by `documentId`.
- **Grounded Answer Synthesis:** Answers generated with `gemini-3.6-flash` (with automated fallback to `gemini-3.5` or `gemini-3.1-flash-lite`).
- **Verifiable Citations:** Each answer displays the source file name, page number, chunk text, and similarity percentage.
- **Rate Safeguards:** 50 search queries per 24 hours per account.

---

## 🛠️ Tech Stack & Icons

### Frontend

| Technology         | Badge                                                                                                                    | Purpose                                       |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------- |
| **React 19**       | ![React](https://img.shields.io/badge/React%2019-20232A?style=flat-square&logo=react&logoColor=61DAFB)                   | Dynamic Single Page Application               |
| **Vite 8**         | ![Vite](https://img.shields.io/badge/Vite%208-646CFF?style=flat-square&logo=vite&logoColor=white)                        | Next-generation bundler and local dev server  |
| **Tailwind CSS 4** | ![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS%204-38B2AC?style=flat-square&logo=tailwind-css&logoColor=white) | Modern glassmorphic styling and responsive UI |
| **Oxlint**         | ![Oxlint](https://img.shields.io/badge/Oxlint-EC5990?style=flat-square&logo=javascript&logoColor=white)                  | High-speed JavaScript linting                 |

### Backend & AI Pipeline

| Technology            | Badge                                                                                                                      | Purpose                                                             |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Node.js**           | ![NodeJS](https://img.shields.io/badge/Node.js%2020+-43853D?style=flat-square&logo=node.js&logoColor=white)                | Runtime environment                                                 |
| **Express 5**         | ![Express](https://img.shields.io/badge/Express%205-000000?style=flat-square&logo=express&logoColor=white)                 | REST API framework & serverless handlers                            |
| **Google Gemini API** | ![Google Gemini](https://img.shields.io/badge/Google%20Gemini-8E75C2?style=flat-square&logo=google-gemini&logoColor=white) | Embeddings (`gemini-embedding-2`) & Generation (`gemini-3.6-flash`) |
| **MongoDB Atlas**     | ![MongoDB](https://img.shields.io/badge/MongoDB%20Atlas-4EA94B?style=flat-square&logo=mongodb&logoColor=white)             | NoSQL vector database with `$vectorSearch`                          |
| **Mongoose 9**        | ![Mongoose](https://img.shields.io/badge/Mongoose%209-880000?style=flat-square&logo=mongoose&logoColor=white)              | Object Data Modeling (ODM)                                          |
| **Multer & PDF.js**   | ![Multer](https://img.shields.io/badge/pdfjs--dist-FF6C37?style=flat-square&logo=adobeacrobatreader&logoColor=white)       | Multi-part upload handling & page-level text extraction             |
| **JWT & PBKDF2**      | ![JWT](https://img.shields.io/badge/JWT%20Auth-000000?style=flat-square&logo=jsonwebtokens&logoColor=white)                | Token-based stateless authentication                                |
| **Vercel**            | ![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white)                        | Frontend hosting & Serverless backend deployment                    |

---

## 🏗️ System Architecture & Data Flow

```mermaid
flowchart TD
    subgraph Client["💻 Frontend (React + Vite)"]
        UI["User Interface"]
        AuthM["Auth Gate & Token Store"]
    end

    subgraph API["⚡ Backend (Express on Node.js / Vercel)"]
        Auth["Auth Controller (JWT + PBKDF2)"]
        Upload["Upload Route (Multer)"]
        Parser["PDF Engine (pdfjs-dist)"]
        Chunker["Sliding Chunking (1000/200)"]
        SearchRoute["Search Router (/api/search)"]
        Intent["Intent Classifier (gemini-3.1-flash-lite)"]
    end

    subgraph AI["🤖 Google Gemini API (@google/genai)"]
        Embed["Vector Embedder (gemini-embedding-2: 3072 dims)"]
        LLM["Answer Generator (gemini-3.6-flash)"]
    end

    subgraph DB["🍃 MongoDB Atlas Vector Store"]
        Users[("Users Collection")]
        Docs[("Documents Collection")]
        Chunks[("DocumentChunks ($vectorSearch)")]
    end

    %% Auth Flow
    UI -->|1. Sign up / Login| Auth
    Auth --> Users
    Auth -->|Returns JWT| AuthM

    %% Upload Flow
    UI -->|2. Upload PDF| Upload
    Upload --> Parser --> Chunker
    Chunker -->|Text chunks in batches of 5| Embed
    Embed -->|Vectors [3072]| Chunks
    Upload --> Docs

    %% Search Flow
    UI -->|3. Ask Question| SearchRoute
    SearchRoute --> Intent
    Intent -->|Greeting| UI
    Intent -->|Query| Embed
    Embed -->|Query Vector| Chunks
    Chunks -->|Top-5 Cosine Chunks| LLM
    LLM -->|Grounded Answer + Citations| UI
```

---

## 📂 Project Directory Structure

```
rag-document-qa/
├── backend/
│   ├── index.js                     # Express server & Vercel entrypoint
│   ├── config/
│   │   ├── auth.js                  # JWT token management & PBKDF2 hashing
│   │   ├── db.js                    # MongoDB Atlas connection & index manager
│   │   ├── gemini.js                # Google GenAI client & model wrappers
│   │   ├── extractPdf.js            # PDF text extraction engine
│   │   └── pdfPolyfill.js           # Polyfills for pdfjs-dist in Node environment
│   ├── middleware/
│   │   └── authMiddleware.js        # Bearer token verification
│   ├── models/
│   │   ├── User.js                  # User schema & credentials
│   │   ├── Document.js              # Uploaded document metadata { userId, fileHash }
│   │   └── DocumentChunk.js         # Text chunk with 3072-dim embedding vector
│   ├── routes/
│   │   ├── authRoutes.js            # /api/auth/signup, /login, /me, /logout
│   │   ├── uploadRoutes.js          # /api/upload, /api/documents, delete document
│   │   └── searchRoutes.js          # /api/search (RAG retrieval & answering)
│   ├── .env.example                 # Backend environment variable template
│   ├── package.json
│   └── vercel.json                  # Serverless function configuration
│
└── frontend/
    ├── index.html                   # HTML entry point with meta tags & fonts
    ├── vite.config.js               # Vite + React + Tailwind v4 build settings
    ├── .env                         # Local/production API endpoints
    ├── .env.example
    ├── package.json
    └── src/
        ├── main.jsx                 # React root mount
        ├── App.jsx                  # State manager, layout & auth gate
        ├── App.css / index.css      # Custom styling, glow gradients, scrollbars
        ├── api/
        │   ├── client.js            # Unified fetch wrapper with auth header injector
        │   ├── auth.js              # Authentication API calls
        │   └── documents.js         # Document upload, query & delete API calls
        └── components/
            ├── AppHeader.jsx        # Navigation bar & user profile status
            ├── AuthScreen.jsx       # Auth switcher container
            ├── LoginForm.jsx        # Login card
            ├── SignupForm.jsx       # Signup card
            ├── DocumentWorkspace.jsx# Document manager & active selection
            ├── DocumentUpload.jsx   # Drag-and-drop file uploader
            ├── DocumentQA.jsx       # Question input & query execution
            ├── AnswerCard.jsx       # Grounded LLM response renderer
            ├── SourceList.jsx       # Retrieved chunk citations with match score
            ├── StatusBadge.jsx      # Operation feedback pills
            └── Icons.jsx            # SVG icon kit
```

---

## 📸 Screenshots

### 1. Account Authentication

![Create account](signup.png)
_Private user registration and session restoration._

### 2. PDF Ingestion & Duplicate Protection

![Upload document](upload-duplicate.png)
_Smart file parsing with SHA-256 duplicate blocking per user account._

### 3. Grounded Q&A with Citation Scores

![Ask a question with retrieved sources](ask-question.png)
_Document-grounded answers citing chunk text, page numbers, and similarity metrics._

---

## 🔑 Required Environment Variables

To operate this project locally or in production, configure the following variables:

### 1. Backend (`backend/.env`)

| Variable         | Description                                              | Example / Recommended Value                                                      |
| ---------------- | -------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `PORT`           | Local Express listening port                             | `8000`                                                                           |
| `MONGODB_URI`    | MongoDB Atlas cluster connection string with credentials | `mongodb+srv://user:pass@cluster.mongodb.net/ragdoc?retryWrites=true&w=majority` |
| `GEMINI_API_KEY` | Google AI Studio API key                                 | `AIzaSy...`                                                                      |
| `JWT_SECRET`     | Cryptographic secret for signing auth tokens             | Any secure string (e.g. `openssl rand -hex 32`)                                  |

### 2. Frontend (`frontend/.env`)

| Variable       | Description                                                                 | Example / Recommended Value       |
| -------------- | --------------------------------------------------------------------------- | --------------------------------- |
| `VITE_API_URL` | Deployed backend URL (Localhost uses `http://localhost:8000` automatically) | `https://your-backend.vercel.app` |

---

## 🚀 Complete Environment & Local Setup Guide

Follow this definitive, step-by-step walkthrough to get every service and credential set up from scratch.

### 1. Prerequisite Accounts & Links

| Service              | Official Portal URL                                               | Free Tier Availability   |
| -------------------- | ----------------------------------------------------------------- | ------------------------ |
| **Node.js (v20+)**   | [nodejs.org](https://nodejs.org/)                                 | Open Source / Free       |
| **Google AI Studio** | [aistudio.google.com](https://aistudio.google.com/)               | Generous Free Tier       |
| **MongoDB Atlas**    | [mongodb.com/atlas](https://www.mongodb.com/cloud/atlas/register) | Free Shared Cluster (M0) |
| **Vercel**           | [vercel.com](https://vercel.com/signup)                           | Free Hobby Plan          |
| **Git**              | [git-scm.com](https://git-scm.com/)                               | Open Source / Free       |

---

### 2. How to Obtain Every Credential

#### A. Google Gemini API Key (`GEMINI_API_KEY`)

1. Go to [Google AI Studio](https://aistudio.google.com/).
2. Sign in with your Google account.
3. Click on the blue **"Get API key"** button on the left sidebar.
4. Click **"Create API key"** (choose an existing Google Cloud project or create a new one instantly with 1 click).
5. Copy the generated key starting with `AIzaSy...`.
6. Store this in your `backend/.env` file as `GEMINI_API_KEY`.

#### B. MongoDB Atlas Connection String (`MONGODB_URI`)

1. Register/Sign in at [MongoDB Atlas](https://www.mongodb.com/cloud/atlas/register).
2. Click **Create Deployment** and select the **M0 Free Cluster**. Choose your preferred cloud provider and region, then click **Create**.
3. **Create Database User:**
   - Under **Database Access** (left menu), click **Add New Database User**.
   - Select **Password Authentication**.
   - Enter a username (e.g. `raguser`) and a secure password. _(Avoid `@` or `:` inside your password to prevent URL parsing errors)_.
   - Set privileges to **Read and write to any database**. Click **Add User**.
4. **Configure Network Access (Whitelisting):**
   - Under **Network Access** (left menu), click **Add IP Address**.
   - Click **Allow Access from Anywhere** (`0.0.0.0/0`) — _This is mandatory for local development and Vercel serverless functions_.
   - Click **Confirm**.
5. **Get Connection String:**
   - Under **Clusters**, click **Connect** on your cluster.
   - Choose **Drivers** (Node.js).
   - Copy the connection URI:
     ```
     mongodb+srv://<username>:<password>@cluster0.xxxx.mongodb.net/?retryWrites=true&w=majority
     ```
   - Add your database name (e.g. `ragdoc`) right before the query parameters:
     ```
     mongodb+srv://raguser:yourpassword@cluster0.xxxx.mongodb.net/ragdoc?retryWrites=true&w=majority
     ```
   - Store this in `backend/.env` as `MONGODB_URI`.

#### C. JWT Secret (`JWT_SECRET`)

Generate a 64-character random string using your terminal:

```bash
# On Windows PowerShell:
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Maximum 256 }))

# Or on Mac/Linux/Git Bash:
openssl rand -hex 32
```

Copy the string and save it in `backend/.env` as `JWT_SECRET`.

---

### 3. Local Backend Configuration

1. Open your terminal and navigate to the `backend` folder:
   ```bash
   cd backend
   ```
2. Install dependencies:
   ```bash
   npm install
   ```
3. Create your `.env` file from `.env.example`:

   ```bash
   # Windows PowerShell:
   Copy-Item .env.example .env

   # Mac / Linux:
   cp .env.example .env
   ```

4. Fill in `backend/.env`:
   ```ini
   PORT=8000
   MONGODB_URI=mongodb+srv://raguser:yourpassword@cluster0.xxxx.mongodb.net/ragdoc?retryWrites=true&w=majority
   GEMINI_API_KEY=AIzaSy...your_gemini_api_key
   JWT_SECRET=your_generated_jwt_secret_string
   ```
5. Start the backend development server:
   ```bash
   npm run dev
   ```
   _The server will start at `http://localhost:8000` with hot-reload enabled via Nodemon._

---

### 4. Local Frontend Configuration

1. Open a **second terminal** and navigate to the `frontend` folder:
   ```bash
   cd frontend
   ```
2. Install dependencies:
   ```bash
   npm install
   ```
3. Create your `.env` file:

   ```bash
   # Windows PowerShell:
   Copy-Item .env.example .env

   # Mac / Linux:
   cp .env.example .env
   ```

4. Start the frontend development server:
   ```bash
   npm run dev
   ```
5. Open your browser and navigate to **`http://localhost:5173`**.

> **Note on Local Connection:** When running locally on `localhost` or `127.0.0.1`, the frontend automatically sends API calls to `http://localhost:8000`. You do not need to alter `VITE_API_URL` for local development!

---

## 🔍 MongoDB Atlas Vector Search Setup

This project uses MongoDB Atlas Vector Search to perform high-speed cosine similarity searches over your document chunks. You must create the index in Atlas before searching will work.

### Step-by-Step Index Creation:

1. In the [MongoDB Atlas Dashboard](https://cloud.mongodb.com/), go to **Atlas Search** on the left menu (or click into your Cluster > **Search** tab).
2. Click **Create Search Index**.
3. Under **Atlas Vector Search**, select the **JSON Editor** and click **Next**.
4. In the database and collection selector:
   - **Database:** `ragdoc` (or your chosen database name in `MONGODB_URI`)
   - **Collection:** `documentchunks` _(If the collection does not exist yet, upload 1 PDF document through the app first, or create the collection manually)_.
5. In the **Index Name** field, enter exactly:
   ```
   autoembed_index
   ```
6. Paste the following JSON configuration:
   ```json
   {
     "fields": [
       {
         "type": "vector",
         "path": "embedding",
         "numDimensions": 3072,
         "similarity": "cosine"
       },
       {
         "type": "filter",
         "path": "documentId"
       }
     ]
   }
   ```
7. Click **Next**, then **Create Search Index**. Atlas will build the index within 1–2 minutes.

> ⚠️ **Important:** Do not add `userId` to the vector filter without indexing it as a filter path in Atlas. Cross-user isolation is enforced deterministically at the application layer via `Document.findOne({ documentId, userId })`.

---

## 📡 API Reference

All protected endpoints require the header: `Authorization: Bearer <JWT_TOKEN>`.

| Method   | Endpoint                     | Auth   | Content-Type          | Description                                   |
| -------- | ---------------------------- | ------ | --------------------- | --------------------------------------------- |
| `GET`    | `/api/health`                | ❌ No  | -                     | Server health status                          |
| `POST`   | `/api/auth/signup`           | ❌ No  | `application/json`    | Register account `{ email, password, name? }` |
| `POST`   | `/api/auth/login`            | ❌ No  | `application/json`    | Authenticate user `{ email, password }`       |
| `GET`    | `/api/auth/me`               | ✅ Yes | -                     | Get currently authenticated user profile      |
| `POST`   | `/api/auth/logout`           | ✅ Yes | -                     | Client drops stateless JWT                    |
| `POST`   | `/api/upload`                | ✅ Yes | `multipart/form-data` | Upload PDF file (form-data field: `file`)     |
| `GET`    | `/api/documents`             | ✅ Yes | -                     | Fetch user's uploaded documents               |
| `DELETE` | `/api/documents/:documentId` | ✅ Yes | -                     | Delete document & cascade delete all chunks   |
| `POST`   | `/api/search`                | ✅ Yes | `application/json`    | Vector Q&A query `{ question, documentId }`   |

---

## ☁️ Deployment on Vercel

The application is architected to deploy seamlessly as two decoupled services on Vercel.

### Project 1: Backend API

1. Push your repository to your GitHub account.
2. In [Vercel](https://vercel.com/), click **Add New Project** and import your repository.
3. In **Root Directory**, click edit and select **`backend`**.
4. In **Environment Variables**, add:
   - `MONGODB_URI` = _(Your MongoDB Atlas connection string)_
   - `GEMINI_API_KEY` = _(Your Google AI Studio API key)_
   - `JWT_SECRET` = _(Your secure random token string)_
     _(Do **not** set `PORT`, Vercel assigns ports automatically)_.
5. Click **Deploy**. Note your backend production domain (e.g. `https://my-rag-backend.vercel.app`).

### Project 2: Frontend Client

1. In Vercel, click **Add New Project** and import the same repository again.
2. In **Root Directory**, click edit and select **`frontend`**.
3. In **Environment Variables**, add:
   - `VITE_API_URL` = `https://my-rag-backend.vercel.app` _(Your deployed backend URL from step 1)_
4. Click **Deploy**.

> 💡 **Remember:** Vite compiles `VITE_*` environment variables during the build process. If you update `VITE_API_URL` in Vercel settings, you must trigger a **Redeploy** for the change to take effect.

---

## 🔄 Cloning & Personalization Guide

If you cloned this repository from a peer or collaborator, here is the exact checklist of items and file locations you must personalize:

| File Location                                                                        | Purpose                        | Required Action                                                 |
| ------------------------------------------------------------------------------------ | ------------------------------ | --------------------------------------------------------------- |
| [`frontend/src/api/client.js`](frontend/src/api/client.js#L2)                        | Hardcoded Fallback Backend URL | Replace `DEPLOYED_API_URL` with your own backend Vercel URL.    |
| [`frontend/.env`](frontend/.env#L3)                                                  | Frontend Environment Variable  | Set `VITE_API_URL` to your own deployed backend URL.            |
| [`frontend/.env.example`](frontend/.env.example#L3)                                  | Frontend Template              | Update fallback reference to your own backend domain.           |
| [`frontend/index.html`](frontend/index.html#L7)                                      | Page Title & Metadata          | Personalize the `<title>` and `<meta name="description">` tags. |
| [`frontend/src/components/AppHeader.jsx`](frontend/src/components/AppHeader.jsx#L11) | UI Branding & Header           | Update app title, descriptions, or pipeline badges.             |
| [`backend/package.json`](backend/package.json#L2)                                    | Package Metadata               | Update `"name"`, `"author"`, and `"repository"`.                |
| [`frontend/package.json`](frontend/package.json#L2)                                  | Package Metadata               | Update `"name"`, `"author"`, and `"repository"`.                |
| [`README.md`](README.md)                                                             | Project Documentation          | Update live links, demo URLs, and personal social handles.      |

---

## 📜 License

This project is open-source under the [MIT License](LICENSE). Feel free to use, modify, and distribute with attribution.

---

## 👨‍💻 Author

**K Tirumala Achari**  
Full Stack Developer | Aspiring Software Engineer

<a href="mailto:ktirumalachari@gmail.com">
  <img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white" alt="Gmail"/>
</a>
<a href="https://www.linkedin.com/in/k-tirumala-achari-921106307/">
  <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn"/>
</a>
<a href="https://github.com/ktirumalaachari">
  <img src="https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white" alt="GitHub"/>
</a>
<a href="https://www.ktirumalaachari.me">
  <img src="https://img.shields.io/badge/Portfolio-FF6B35?style=for-the-badge&logo=firefox&logoColor=white" alt="Portfolio"/>
</a>
<br/><br/>

> _"Passionate about building impactful, user-centric solutions through technology,_
> _committed to continuous learning and innovation."_

**⭐ If you found this project helpful or inspiring, please give it a star! ⭐**
Made with ❤️ by **K Tirumala Achari**

[![GitHub](https://img.shields.io/badge/GitHub-ktirumalaachari-blue?style=flat&logo=github)](https://github.com/ktirumalaachari)

</div>
