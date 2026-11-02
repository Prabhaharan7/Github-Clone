# ⚡ VRel

A self-hosted GitHub-like platform built with Node.js + React for internal company use. The system functions on a serverless, database-free architecture utilizing a Git-like content-addressed file object store, custom file-based mutexes, real-time SSE log streaming, automated Maven/Python test pipelines, and exclusive lock checkout systems.

## 🚀 Features

| Feature | Description |
|---------|-------------|
| Repository Management | Init, stage, commit, log, status |
| Branching | Create, switch, fast-forward merge, conflict detection |
| Java Build Pipeline | Maven/Gradle triggered on push, live log streaming |
| Python Build Pipeline | virtualenv + pip + pytest, sequential steps |
| Exclusive Checkout | File locking with tokens, expiry, force release |
| JWT Authentication | Secure login, protected routes, auto token refresh |
| Race Condition Safety | File-based mutex for concurrent operations |

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Node.js 20+, Express.js |
| Frontend | React 18, Vite 5 |
| Storage | File-system (Git-like loose object store) |
| Auth | JWT (jsonwebtoken) |
| Real-time | Server-Sent Events (SSE) |
| Hashing | SHA-1 (Node.js crypto) |
| Build Tools | Maven/Gradle (Java), virtualenv/pytest (Python) |

## 📋 Prerequisites

- Node.js 20+
- npm 9+
- Java 11+ and Maven (for Java build pipeline)
- Python 3.8+ (for Python build pipeline)

## ⚙️ Installation & Setup

### 1. Clone the repository
```bash
git clone <your-repo-url>
cd VRel
```

### 2. Backend setup
```bash
cd backend
npm install
```

Create a `.env` file under `backend/`:
```env
PORT=3000
NODE_ENV=development
JWT_SECRET=replace-with-a-long-random-secret
CORS_ORIGIN=http://localhost:5173
```

### 3. Frontend setup
```bash
cd ../frontend
npm install
```

### 4. Run the application

Terminal 1 — Backend:
```bash
cd backend
npm run dev
```

Terminal 2 — Frontend:
```bash
cd frontend
npm run dev
```

Open: http://localhost:5174

### 5. Default login credentials

| Username | Password | Role |
|----------|----------|------|
| prabhaharan | admin123 | Admin |
| friend | dev123 | Developer |
| admin | admin123 | Admin |

---

## 🗂️ Project Structure

```
VRel/
├── backend/
│   ├── src/
│   │   ├── routes/        # repo, branch, build, checkout
│   │   ├── controllers/   # HTTP handlers for each domain
│   │   ├── models/        # blob, tree, commit, lock descriptors
│   │   ├── services/      # objectStore, merge, javaBuild, pythonBuild, lock
│   │   ├── middleware/    # JWT auth, validation, centralized error handling
│   │   ├── utils/         # hash, SSE, and file mutex utilities
│   │   └── app.js         # Express entry point
│   ├── storage/           # Local repositories data storage
│   ├── package.json
│   └── .env               # Environment configuration
├── frontend/
│   ├── src/
│   │   ├── pages/         # RepoPage, BranchPage, BuildPage, CheckoutPage
│   │   ├── components/    # CommitHistory, BuildLog, FileLockBadge
│   │   ├── api/           # Axios client with JWT interceptors
│   │   └── main.jsx       # React + Router entry point
│   ├── vite.config.js
│   └── package.json
├── tests/                 # Node HTTP test runners (standalone/integrated)
├── docker-compose.yml
├── .gitignore
└── README.md
```

---

## 🧪 Running Tests

Ensure backend is running, then run tests:
```bash
cd backend
npm test
```
Expected output: All 56/56 tests passing.

---
## 🚢 Docker Deployment

Build and run the entire stack with one command:

```bash
docker-compose up --build
```


|
 Service 
|
 URL 
|
|
---------
|
-----
|
|
 Backend API 
|
 http://localhost:3000 
|
|
 Frontend UI 
|
 http://localhost:5174 
|

To stop:
```bash
docker-compose down
```

## 📡 API Overview

| Area | Method | Path | Description |
|------|--------|------|-------------|
| **Auth** | `POST` | `/api/auth/login` | Log in user, return signed JWT |
| | `POST` | `/api/auth/verify` | Verify authentication status |
| **Repo** | `GET` | `/api/repos` | List all repositories |
| | `POST` | `/api/repos/:repoName/init` | Initialize a new repository |
| | `POST` | `/api/repos/:repoName/add` | Stage a file to index |
| | `POST` | `/api/repos/:repoName/commit` | Create a commit of staged files |
| | `GET` | `/api/repos/:repoName/log` | View commit history log |
| | `GET` | `/api/repos/:repoName/status` | Get staging area and branch info |
| | `GET` | `/api/repos/:repoName/objects/:hash` | Retrieve content-addressed object |
| **Branch** | `GET` | `/api/repos/:repoName/branches` | List branches and current tip |
| | `POST` | `/api/repos/:repoName/branches` | Create a new branch |
| | `POST` | `/api/repos/:repoName/branches/switch` | Switch active branch |
| | `POST` | `/api/repos/:repoName/branches/merge` | Merge source branch into active |
| **Build** | `GET` | `/api/repos/:repoName/builds/trigger` | Trigger pipeline build (SSE stream) |
| | `GET` | `/api/repos/:repoName/builds` | List repository builds history |
| | `GET` | `/api/repos/:repoName/builds/:buildId` | View build record details |
| | `POST` | `/api/repos/:repoName/builds/recover` | Recover stuck repository builds |
| **Locks** | `POST` | `/api/repos/:repoName/locks` | Checkout and acquire file lock |
| | `DELETE` | `/api/repos/:repoName/locks` | Release file lock |
| | `GET` | `/api/repos/:repoName/locks` | List active repository locks |
| | `GET` | `/api/repos/:repoName/locks/file` | Check file path lock status |

| **Build** | `POST` | `/api/repos/:repoName/builds/seed` | Seed Maven Java workspace |
| | `POST` | `/api/repos/:repoName/builds/seed-python` | Seed Python pytest workspace |
| **Locks** | `DELETE` | `/api/repos/:repoName/locks/force` | Force release lock (admin) |
| | `POST` | `/api/repos/:repoName/locks/clean` | Clean all expired locks |
---

## 🔄 How It Works

### Git-like Object Model
Files are saved as content-addressed Git-like objects. Raw file contents are hashed and written to disk as `blobs`. Directories are stored as `trees` mapping file names to blob hashes, and `commits` record metadata (author, message, parent commits) alongside a pointer to the root tree hash.

### Build Pipeline Flow
When changes are pushed, the system checks the workspace directory to detect the build files (e.g., Maven, Gradle, or Python). The backend spawns a child subprocess running the test suite commands. Standard output and error streams are captured line-by-line and piped directly to the frontend in real-time using Server-Sent Events (SSE).

### Exclusive Checkout
The application handles binary and critical files with an exclusive file-locking checkout mechanism. When a user locks a file path, a JSON lock record containing a unique `lockId`, timestamp, and time-to-live (TTL) expiration is written. Concurrent lock requests and releases are serialized using file-system based directory mutex locks to avoid race conditions.


