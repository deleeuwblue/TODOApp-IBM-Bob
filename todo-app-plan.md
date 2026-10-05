# TODO App — Implementation Plan

## Overview

Build a minimal TODO application with a Python FastAPI backend and a vanilla JavaScript
single-page frontend. All files live at the project root (flat structure). Data is
persisted in a local SQLite file via SQLAlchemy ORM. No build step is required.

**Constraints & decisions:**
- Python 3.13.11
- FastAPI + Uvicorn + SQLAlchemy (no extra DB infrastructure)
- SQLite file: `todo.db` (created automatically on startup)
- Frontend: single `index.html` served statically by FastAPI at `GET /`
- Vanilla JS only — no frameworks, no bundler
- Flat project structure: `main.py`, `index.html`, `requirements.txt` at root

---

## Sub-Task 1 — Backend Foundation

**Status:** [ ] pending

### Intent
Establish the FastAPI application skeleton with a SQLAlchemy model and automatic
database initialisation. This is the foundation every other sub-task depends on.

### Expected Outcomes
- `main.py` exists and imports cleanly
- A `Todo` SQLAlchemy model with `id`, `title`, and `done` columns is defined
- `todo.db` is created (or reused) when the app starts
- Running `uvicorn main:app --reload` starts without errors

### Todo List
1. Create `main.py` with a FastAPI `app` instance
2. Configure SQLAlchemy with a SQLite `DATABASE_URL` pointing to `./todo.db`
3. Define `SessionLocal` and `Base` using `declarative_base()`
4. Define the `Todo` ORM model (`id: int PK autoincrement`, `title: str`, `done: bool default False`)
5. Call `Base.metadata.create_all(bind=engine)` at module load to initialise the schema
6. Add a `get_db` dependency that yields a session and closes it on teardown

### Relevant Context
- SQLAlchemy `create_engine` with `connect_args={"check_same_thread": False}` is required for SQLite + FastAPI
- Use `sessionmaker(autocommit=False, autoflush=False, bind=engine)`

---

## Sub-Task 2 — CRUD API Routes

**Status:** [ ] pending

### Intent
Expose four REST endpoints that cover all TODO operations. Pydantic schemas enforce
request/response shapes at the API boundary.

### Expected Outcomes
- `GET /todos` returns a JSON array of all todos
- `POST /todos` creates a new todo and returns it (201)
- `PUT /todos/{id}` toggles the `done` field and returns the updated todo
- `DELETE /todos/{id}` removes the todo and returns 204
- Invalid `{id}` values return 404

### Todo List
1. Define Pydantic `TodoBase` (title: str), `TodoCreate(TodoBase)`, and `TodoRead` (id, title, done) schemas
2. Implement `GET /todos` — query all rows, return `list[TodoRead]`
3. Implement `POST /todos` — accept `TodoCreate`, insert row, return `TodoRead` with status 201
4. Implement `PUT /todos/{id}` — fetch by id (404 if missing), flip `done`, commit, return `TodoRead`
5. Implement `DELETE /todos/{id}` — fetch by id (404 if missing), delete, commit, return 204

### Relevant Context
- Inject `db: Session = Depends(get_db)` on every route
- Use `response_model=TodoRead` (or `list[TodoRead]`) on route decorators
- FastAPI auto-generates `/docs` (Swagger UI) — useful for manual testing

---

## Sub-Task 3 — Vanilla JS Frontend

**Status:** [ ] pending

### Intent
Deliver a self-contained `index.html` that lets users view, add, toggle, and delete
todos through the FastAPI API without any build tooling.

### Expected Outcomes
- Page loads and immediately fetches + renders the current todo list
- Submitting the form adds a new todo and refreshes the list
- Clicking a checkbox toggles the `done` state via `PUT /todos/{id}`
- Clicking a delete button removes the todo via `DELETE /todos/{id}`
- No page reloads — all interactions use `fetch()`

### Todo List
1. Create `index.html` with semantic HTML structure: heading, `<form>` with text input + submit button, `<ul id="todo-list">`
2. Write `loadTodos()` — `GET /todos`, clear the list, render each item as `<li>` with a checkbox, label, and delete button
3. Write form `submit` handler — `POST /todos` with `{title}`, then call `loadTodos()`
4. Write checkbox `change` handler — `PUT /todos/{id}` on the specific item, then call `loadTodos()`
5. Write delete button `click` handler — `DELETE /todos/{id}`, then call `loadTodos()`
6. Call `loadTodos()` on `DOMContentLoaded`
7. Add minimal inline CSS for readability (no external dependencies)

### Relevant Context
- All fetch calls target relative paths (`/todos`, `/todos/{id}`) so no hardcoded host is needed
- The `done` field should visually strike through completed todo titles

---

## Sub-Task 4 — Static Serving, CORS, and requirements.txt

**Status:** [ ] pending

### Intent
Wire the frontend into FastAPI so `index.html` is served at `GET /` and the app is
runnable from a fresh checkout with a single `pip install`.

### Expected Outcomes
- `GET /` returns `index.html` as an HTML response
- A browser hitting `http://127.0.0.1:8000/` sees the working UI
- `pip install -r requirements.txt` installs all dependencies
- CORS is configured to allow localhost origins (for local development)

### Todo List
1. Add `GET /` route in `main.py` that returns `FileResponse("index.html")`
2. Add `CORSMiddleware` to the app allowing origins `http://127.0.0.1:8000` and `http://localhost:8000`
3. Create `requirements.txt` pinned to compatible versions:
   - `fastapi>=0.115.0`
   - `uvicorn[standard]>=0.32.0`
   - `sqlalchemy>=2.0.36`
4. Verify the app starts with `uvicorn main:app --reload` and the UI is reachable at `http://127.0.0.1:8000/`

### Relevant Context
- `FileResponse` is imported from `fastapi.responses`
- `CORSMiddleware` is imported from `starlette.middleware.cors`
- Starlette is a transitive dependency of FastAPI — no extra install needed
- `uvicorn[standard]` includes `watchfiles` for `--reload` support on Python 3.13

---

## Implementation Notes

- Sub-tasks must be implemented in order: 1 → 2 → 3 → 4
- After each sub-task, update its **Status** to `[x] done` before starting the next
- Python version: **3.13.11**
- Do not add authentication, pagination, or any feature beyond what is described
