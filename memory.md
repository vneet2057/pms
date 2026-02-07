# Project Memory: pms (Monorepo)

This memory file captures a directory-level schema and short descriptions for the main top-level folders in this workspace. It's intended as a quick-reference for contributors exploring the repo.

---

## Root layout

- `backend/` : Django backend service (APIs, models, business logic, migrations, Docker setup).
- `saas-pms-dashboard/` : Frontend single-page app (Vite + React + TypeScript + Tailwind).

---

## backend/

High-level: A Django project with multiple apps for features (customers, projects, users, work items, sprints, etc.). Contains Docker and orchestration helper files.

- `manage.py` : Django CLI entry.
- `requirements.txt` : Python dependencies for the backend.
- `Dockerfile`, `docker-compose.yml`, `entrypoint.sh`, `asgi-entrypoint.sh` : Containerization and process startup.

Subfolders (Django apps and utilities):

- `pms/` : Django project configuration.
  - `settings.py`, `urls.py`, `asgi.py`, `wsgi.py`, `middleware.py`, `jwt_auth.py`, `routing.py` — project-level settings, middleware, ASGI/WGI and auth helpers.

- `customer/` : App managing customers.
  - Typical files: `models.py`, `views.py`, `urls.py`, `admin.py`, `tests.py`, `adapters/`, `migrations/`.

- `dashboard/` : App for server-side pieces related to dashboard features and APIs.
  - Contains API endpoints, docs (`MEMBER_API_DOCS.md`), `models.py`, `views.py`, `adapters/`, `migrations/`.

- `extension/` : Feature-surface or plugin-like functionality (models, views, adapters).

- `lead/` : Lead management app; includes `permission.py` and app-specific APIs and models.

- `project/` : Project entity management (models, permissions, adapters, views).

- `sprint/` : Sprint entities and related logic (models, views, migrations).

- `user/` : User accounts, authentication, profiles, admin integration.

- `profile_pics/` : Static/media handling related to profile pictures (likely storage/backing models).

- `settings_app/` : App for application settings and configuration persisted in the DB.

- `work_items/` : Core work item entities (models, views, consumers for websockets, filters, signals, adapters). Notable files:
  - `consumers.py` : WebSocket consumers.
  - `filters.py` : DRF filter classes or search helpers.
  - `signals.py` : Django signal handlers.

- `utils/` : Shared utilities used across apps.
  - `custom_paginator.py`, `email_service.py`, `slack_notification.py` — helpers for pagination and notifications.

Notes about app layout: Each app typically follows Django conventions: `models.py`, `views.py`, `urls.py`, `admin.py`, `tests.py`, an `adapters/` folder for integration with external services, and `migrations/` for DB schema history.

---

## saas-pms-dashboard/

High-level: Frontend SPA built with Vite + React + TypeScript and Tailwind CSS. Implements the web UI for the SaaS product.

- `package.json` : Node dependencies and scripts.
- `vite.config.ts`, `tsconfig.*.json` : Build and TypeScript configuration.
- `tailwind.config.ts` : Tailwind CSS configuration.
- `index.html`, `public/` : App shell and static assets.
- `components.json`, `README.md`, `docs/` : Documentation and component lists.

`src/` structure (typical):

- `main.tsx` : App bootstrap.
- `App.tsx`, `App.css` : Root component + global styles.
- `assets/` : Images, icons and static assets used by the app.
- `components/` : Reusable UI components.
- `core/` : Core providers, routing, and global utilities.
- `hooks/` : Custom React hooks.
- `lib/` : Reusable libraries and API clients.
- `pages/` : Page-level components and route targets.
- `types/` : TypeScript type definitions used across the app.

Notes: The frontend appears tailored for an enterprise-like dashboard and includes docs for infinite scroll behaviour (`docs/WORKITEMS_INFINITE_SCROLL.md`) and other domain-specific guidelines.

---

## Helpful pointers

- To run backend locally: use `manage.py` or Docker files; check `Dockerfile` and `docker-compose.yml` for exact commands and environment variable expectations.
- To run frontend locally: `npm install` then `npm run dev` (check `package.json` for script names).

---

## Quick reference: where to look for common tasks

- Add a database migration: `backend/<app>/migrations/` and `manage.py makemigrations`.
- Add a REST API route: backend `<app>/views.py` and wire up in `<app>/urls.py` and project `pms/urls.py`.
- Frontend component: `saas-pms-dashboard/src/components/` or `src/pages/`.

---

If you want, I can expand this memory file with a tree-style listing of files for each app, or add run commands and environment variable references extracted from Docker-related files.
