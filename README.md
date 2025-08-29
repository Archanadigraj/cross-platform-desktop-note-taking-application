# NoteSync Desktop (Electron + React) — Hybrid Node.js + Python Backend

Production-leaning desktop note-taking app with secure auth, offline-first caching, pagination, real-time sync, Redis caching, and Windows packaging.

## Key Features
- **Authentication:** Signup/Login with hashed+salted passwords (bcrypt), JWT access + refresh tokens.
- **Authorization:** Per-user notes; optional admin role to view all notes.
- **Electron + React UI:** Login, Dashboard, Settings; **Compact/Expand** window toggle (400×300 ↔ 1200×800).
- **State Management:** Zustand store for auth/session/preferences.
- **Local/Offline:** Preferences in localStorage; notes cached in IndexedDB via `localforage` (offline view + later sync).
- **Lazy Loading:** Notes fetched in pages (20 per request) with infinite scroll.
- **Realtime:** WebSockets (Socket.IO) for bi-directional, multi-client sync.
- **Backend Hybrid:** Node.js (REST + Socket.IO + MongoDB + Redis) + Python (FastAPI microservice for note analysis & health).
- **Caching:** Redis cache for paginated `/notes` fetch (invalidated on new note).
- **Packaging:** Windows `.exe` via `electron-builder`.
- **Extras:** Error handling, load-test script, encrypted local storage (CryptoJS), role-based access (admin).

## Repo Layout
```text
.
├─ backend-node/        # Express + Mongoose + Redis + Socket.IO
├─ backend-python/      # FastAPI microservice
├─ frontend/            # Electron + React + Vite renderer
├─ docs/                # Assignment PDF + architecture diagram
├─ docker-compose.yml   # MongoDB, Redis, Python service, Node service (optional)
└─ README.md
```

## Prerequisites
- **Windows 10/11** (for `.exe` build), PowerShell, Git
- **Node.js 20+** and **npm 10+**
- **Python 3.10+**
- **Docker Desktop** (for MongoDB + Redis + optional containerized services)
- **Internet** for `npm install`/`pip install`

## Quick Start (Dev)
1) Clone or unzip this project.
2) Copy environment and install deps:
   ```powershell
   cd backend-node
   copy .env.example .env
   npm install
   cd ..ackend-python
   python -m venv .venv
   .\.venv\Scriptsctivate
   pip install -r requirements.txt
   cd ..
   ```
3) Start infra (MongoDB + Redis) with Docker:
   ```powershell
   docker compose up -d mongo redis
   ```
4) Start Python service (locally or via Docker):
   ```powershell
   cd backend-python
   .\.venv\Scriptsctivate
   uvicorn app:app --host 0.0.0.0 --port 8001 --reload
   # or: docker compose up -d python
   ```
5) Start Node backend:
   ```powershell
   cd ..ackend-node
   npm run dev
   ```
6) Start Electron + React frontend:
   ```powershell
   cd ..rontend
   npm install
   npm run dev         # dev renderer + Electron
   ```

Open the app window that launches. Login/signup and start adding notes.
If the backend is unreachable, the UI shows a friendly error and uses cached notes.

## Build Windows .exe
```powershell
cd frontend
npm run build          # bundles renderer
npm run electron:build # builds Windows installer via electron-builder
```
Output appears under `frontend/dist-electron` / `frontend/release`.

## Environment Variables
**backend-node/.env**
```ini
PORT=8000
MONGO_URI=mongodb://localhost:27017/notesync
JWT_ACCESS_SECRET=dev_access_secret_change_me
JWT_REFRESH_SECRET=dev_refresh_secret_change_me
ACCESS_TTL_MINUTES=15
REFRESH_TTL_DAYS=7
REDIS_URL=redis://localhost:6379
PYTHON_SERVICE_URL=http://localhost:8001
```

## API (Selected)
- `POST /api/auth/signup` {username, password}
- `POST /api/auth/login` {username, password} → {accessToken, refreshToken, user}
- `POST /api/auth/refresh` {refreshToken}
- `POST /api/notes` {text}  (auth)
- `GET  /api/notes?cursor=<id>&limit=20`  (auth, paginated)
- Admin: `GET /api/admin/notes`  (role=admin)

## Load Testing
- Quick local stress: `cd backend-node && npm run loadtest` (uses `autocannon`).

## Offline-First + Caching
- Notes cached in IndexedDB. If offline, dashboard shows cached notes and queues new notes for retry on reconnect.
- Redis caches paginated responses per user (`notes:<userId>:<cursor>:<limit>`). Invalidated on new note.

## Security Notes
- Passwords hashed (bcrypt) with salt.
- JWT access & refresh with rotation; refresh stored server-side in Redis (revocable).
- Renderer encrypts token-at-rest in localStorage using AES (CryptoJS).

## Docs
- See `docs/🖥️ Full Stack Software Developer Intern — First Round Assignment.pdf` for the original assignment.
- `docs/architecture.md` explains module boundaries and data flow.

---
© 2025 NoteSync Demo
