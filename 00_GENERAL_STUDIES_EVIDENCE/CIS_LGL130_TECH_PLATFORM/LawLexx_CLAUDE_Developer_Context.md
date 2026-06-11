# LawLexx — CLAUDE.md

Developer context for AI assistants working on this codebase.

---

## ABSOLUTE RULE — Production Only

**NEVER use mocks, stubs, placeholders, fake data, test fixtures, or demo values in any code, config, or environment variable.**
Every value, API call, database query, key, and URL must be real and production-ready.
If a value is unknown, ask — never fill with a placeholder.
This rule applies at all times unless the user explicitly says otherwise.

---

## Project Overview

LawLexx is a full-stack legal AI SaaS platform for law firms. It handles matter management, AI-assisted document drafting, legal research, billing, and more.

**Stack:**
- Frontend: Next.js 14 (App Router), TypeScript, Tailwind CSS, Zustand, Framer Motion
- Backend: FastAPI (Python 3.13+), SQLAlchemy 2 async, SQLite (dev) / PostgreSQL (prod)
- AI: Ollama (llama3:8b) locally, Anthropic Claude API as fallback/production
- Auth: JWT access + refresh tokens stored in localStorage/cookies

---

## Dev Commands

```bash
# Backend (from C:\LawLexx\backend\)
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Frontend (from C:\LawLexx\frontend\)
npm run dev
```

Backend API docs at `http://localhost:8000/docs`.
Frontend at `http://localhost:3000`.

---

## Project Structure

```
C:\LawLexx\
├── frontend/src/
│   ├── app/
│   │   ├── (auth)/           # /login, /register
│   │   ├── (dashboard)/      # Protected pages — layout.tsx handles auth check
│   │   │   └── dashboard/    # All dashboard routes (chat, matters, documents, etc.)
│   │   └── page.tsx          # Landing page
│   ├── components/layout/    # Sidebar.tsx, TopBar.tsx
│   ├── lib/
│   │   ├── api.ts            # All axios API calls — AI_TIMEOUT = 200_000ms
│   │   └── auth.ts           # getAccessToken(), isAuthenticated()
│   └── types/                # TypeScript types (Matter, Document, User...)
│
└── backend/
    ├── main.py               # FastAPI app entry, CORS, router registration
    └── app/
        ├── api/              # Route handlers (auth, matters, documents, ai_agent,
        │                     #   billing, calendar, upload, research, analytics)
        ├── models/           # SQLAlchemy ORM models (User, Matter, Document,
        │                     #   TimeEntry, CalendarEvent, Firm)
        ├── services/         # Business logic
        │   ├── ai_service.py         # Ollama + Anthropic AI calls
        │   ├── pleading_service.py   # PDF generation (reportlab)
        │   ├── courtlistener_service.py  # Case law search
        │   └── govinfo_service.py    # Federal statutes
        ├── database.py       # Async SQLAlchemy engine + get_db session
        └── config.py         # Pydantic settings (reads from .env)
```

---

## Critical: UUID Storage Bug Pattern

**All IDs are stored as `String(36)` with hyphens** (`"7db318c7-2522-4cc5-99c7-de5e455cf07e"`), created via `default=lambda: str(uuid.uuid4())`.

**Problem:** FastAPI path params typed as `uuid.UUID` are parsed into Python UUID objects. SQLAlchemy then converts them to no-hyphen hex (`7db318c725224cc599c7de5e455cf07e`) when binding to String(36) columns → 404 mismatch.

**Fix — always use `str()` in WHERE clauses:**
```python
# WRONG — causes 404
result = await db.execute(select(Document).where(Document.id == doc_id))

# CORRECT
result = await db.execute(select(Document).where(Document.id == str(doc_id)))
```

This applies to all path/query params typed as `uuid.UUID` in: `documents.py`, `matters.py`, `billing.py`, `upload.py`, `calendar.py`.

---

## AI Service Configuration

**File:** `backend/app/services/ai_service.py`

- **Primary:** Ollama (llama3:8b) at `OLLAMA_BASE_URL` (default `http://localhost:11434`)
- **Fallback:** Anthropic Claude API when Ollama unavailable
- **Token budgets:** `CHAT_NUM_PREDICT = 1024`, `DOC_NUM_PREDICT = 3000`
- **Timeouts:** `OLLAMA_TIMEOUT = 300.0` seconds
- llama3:8b takes ~30–120s for full legal responses with system prompt

**Streaming:** `POST /api/ai/chat/stream` returns SSE (`text/event-stream`).
Format: `data: {"token": "word"}\n\n` ... `data: [DONE]\n\n`

Frontend streams with native `fetch()` + `ReadableStream` (not axios — streaming needs the raw fetch API).

---

## Frontend API Calls

**File:** `frontend/src/lib/api.ts`

- All AI endpoints use `{ timeout: AI_TIMEOUT }` (200 seconds)
- `axiosInstance` base URL defaults to `http://localhost:8000`
- Auth header is auto-injected via request interceptor from `getAccessToken()`

For the streaming chat endpoint, the frontend uses `fetch()` directly with `Authorization: Bearer <token>` header (axios doesn't support SSE streaming well).

---

## Design System / Theme

**Landing page:** `#0A1628` navy, `#C9A84C` gold accent, `rounded-2xl` cards, `backdrop-blur`
**Dashboard (dark mode):** same `#0A1628` navy background, `#C9A84C` gold in sidebar/topbar, `12px` border radius on `.glass-card`

**CSS variables (dark mode in globals.css):**
- `--bg: #0A1628`, `--bg-2: #0d1b32`
- Orange `#e8632a` is used for: CTAs (`btn-primary`), focus rings, status indicators
- Gold `#C9A84C` is used for: sidebar active items, logo, topbar avatar, brand elements

**Sidebar:** `background: #060e1a`, gold active state, muted text `#8a94a6`
**TopBar:** `background: #060e1a` (dark mode)

---

## Auth Flow

1. `POST /api/auth/login` → returns `{ access_token, refresh_token }`
2. Tokens stored in `localStorage` as `ll_access` and `ll_refresh`
3. `getAccessToken()` from `@/lib/auth` reads `ll_access`
4. Dashboard layout checks `isAuthenticated()` on mount, redirects to `/login` if false
5. Axios interceptor auto-adds `Authorization: Bearer <token>` to all requests

---

## Backend Session / Commit Pattern

The `get_db` dependency:
```python
async with AsyncSession(engine) as session:
    yield session
    await session.commit()
```
Commit happens AFTER the endpoint returns. Use `await db.flush()` inside endpoints to get auto-generated IDs before the response is built.

---

## Common Patterns

**Returning model data:** Use `doc_to_dict()` / `matter_to_dict()` serializers (always call `str()` on UUID fields).

**Enum validation:** Wrap `DocumentType(value)` / `LawArea(value)` in try/except and raise HTTP 422 with the valid values listed.

**AI endpoint timeout:** All AI calls via axios must pass `{ timeout: 200_000 }` — the default 30s will always timeout on llama3:8b.

**New dashboard page:** Add to `PATH_TITLES` in `TopBar.tsx` to show proper page name in the header.
