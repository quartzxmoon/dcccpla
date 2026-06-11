# Repository Guidelines

## Project Structure & Module Organization
`frontend/` contains the Next.js App Router UI in `src/app`, shared UI in `src/components`, hooks in `src/hooks`, and API helpers in `src/lib`. `backend/` contains the FastAPI service: routes in `app/api`, ORM models in `app/models`, business logic in `app/services`, and Alembic migrations in `app/migrations`. `word-addin/` is a separate Office add-in with source under `src/` and Jest tests under `tests/`. Treat `archived_stale_addins/`, `old/`, and status/report Markdown files as historical artifacts, not active implementation targets.

## Build, Test, and Development Commands
- `cd backend && uvicorn main:app --reload --host 0.0.0.0 --port 8000` runs the API locally on `:8000`.
- `cd frontend && npm run dev` starts the web app on `:3000`.
- `cd frontend && npm run build` creates the production Next.js build.
- `cd frontend && npx playwright test` runs browser tests from `frontend/e2e`.
- `cd word-addin && npm start` runs the add-in dev server and proxy.
- `cd word-addin && npm run build` builds the add-in bundle.
- `cd word-addin && npm test` or `npm run test:all` runs Jest tests.
- `cd backend && pytest tests -v` is the standard backend test entry point.

## Coding Style & Naming Conventions
Use TypeScript and React for frontend changes, Python for backend changes, and keep indentation to 2 spaces in frontend files and 4 spaces in Python. Follow the existing naming patterns: `PascalCase` for React components, `useX` for hooks, `snake_case` for Python modules, and feature-oriented API files such as `matter_context.py` or `filing_validation.py`. Lint frontend code with `cd frontend && npm run lint`. Keep styling aligned with the existing navy/gold design system rather than introducing a new palette.

## Testing Guidelines
Frontend browser coverage uses Playwright in `frontend/e2e/*.spec.ts`; component tests live beside components in `__tests__`. Word add-in tests use Jest with `tests/**/*.test.js`. Backend tests are pytest-based and currently live in `backend/tests/` plus a few root-level `test_*.py` scripts in `backend/`. Prefer naming tests `test_<feature>.py` and keep new automated tests close to the feature they validate.

## Commit & Pull Request Guidelines
Recent history follows short, imperative Conventional Commit prefixes such as `fix:`, `chore:`, and `archive:`. Keep subjects specific, for example `fix: tolerate schema drift in router startup`. PRs should summarize scope, note affected areas (`frontend`, `backend`, `word-addin`), link issues when available, and include screenshots for UI or add-in changes. Call out new env vars, migrations, or manual verification steps explicitly.
