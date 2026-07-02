# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is Udacity's "Great Bookshelf" exercise repo for the API Development and Documentation course. It is a sequence of six progressively-built exercises (Requests → Errors → Testing → TDD → API Doc → Final), each implementing the same small book-rating app: a Flask + SQLAlchemy + PostgreSQL backend and a React (Vite) frontend.

Each exercise has two folders:
- `N_Name_Starter/` — contains `# TODO` / `@TODO` markers for students to fill in (routes, error handlers, tests). Backend logic is often intentionally incomplete or commented out.
- `N_Name_Review/` — the completed reference solution for that exercise.

Later exercises build on earlier ones (e.g. `4_TDD_*` adds `test_flaskr.py` before implementing endpoints; `5_API_Doc_*` documents the completed API in its README). There is no shared code between exercise folders — each is a fully standalone backend + frontend pair with its own `requirements.txt` / `package.json`.

`Examples_from_plants_database/` and `Bookshelf_database_files/` hold auxiliary reference material (a plants-DB CORS demo, and root-level `.psql`/`setup.sql` dumps) and are not part of the exercise flow.

When making changes, identify which specific exercise folder(s) are in scope — do not assume a fix in one `N_Name_Starter`/`N_Name_Review` pair should propagate to others; check whether the same TODO/logic is duplicated there first.

## Architecture (per exercise)

**Backend** (`<exercise>/backend/`):
- `flaskr/__init__.py` — app factory `create_app()`, all routes, CORS setup, and error handlers live here in a single file.
- `models.py` — SQLAlchemy setup (`setup_db`) and the `Book` model (`id`, `title`, `author`, `rating`) with `insert()`/`update()`/`delete()`/`format()` helpers. Hardcodes DB credentials (`student`/`student`) and DB name `bookshelf` (test DB: `bookshelf_test`).
- `test_flaskr.py` (from exercise 3 onward) — unittest-based API tests using Flask's test client.
- `books.psql` / `setup.sql` — schema/seed data and DB+user bootstrap script.
- Pagination is fixed at `BOOKS_PER_SHELF = 8`, controlled by a `page` query/body arg (1-indexed).
- Standard error envelope: `{"success": false, "error": <code>, "message": "..."}` for 400/404/422.

**Frontend** (`<exercise>/frontend/`):
- Vite + React 18 (migrated from CRA — see `vite.config.js`).
- Dev server runs on port 3000 and proxies `/books` requests to the Flask backend at `http://127.0.0.1:5000`.
- `src/App.jsx` is the main component; `src/components/Book.jsx` and `FormView.jsx` render the book list and the add-book form.

## Commands

Run all commands from inside the specific exercise's `backend/` or `frontend/` folder — there is no root-level build/test setup.

### Backend setup & run
```bash
python3 -m venv venv && source venv/bin/activate
pip3 install -r requirements.txt
export FLASK_APP=flaskr
export FLASK_ENV=development
flask run          # serves on http://127.0.0.1:5000
```

### Database setup (PostgreSQL, once per exercise folder)
```bash
psql postgres -f setup.sql          # creates db/user, per backend/setup.sql
psql -f books.psql -U student -d bookshelf
```

### Backend tests (exercises 3+, i.e. `3_Testing_*` onward)
```bash
psql postgres
dropdb bookshelf_test   # omit on first run
createdb bookshelf_test
\q
psql bookshelf_test < books.psql
python test_flaskr.py
```
Run a single test case: `python -m unittest test_flaskr.BookTestCase.test_get_paginated_books`

### Frontend setup & run
```bash
npm install
npm start           # vite dev server on http://localhost:3000
```
Other scripts: `npm run build`, `npm run preview`. `npm test` is a no-op placeholder (no frontend test suite is configured).

## Conventions

- Backend code follows PEP8.
- Verify the DB username in `books.psql`, `models.py`, and `test_flaskr.py` matches your local Postgres user (`student` for the Udacity classroom workspace, `postgres` for a typical local default) before running anything — these files hardcode credentials independently and can drift out of sync within an exercise.
