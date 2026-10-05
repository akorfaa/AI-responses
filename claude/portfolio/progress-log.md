# Progress Log — Invoice & Payment Tracker API

Format for each entry: date, what got done, what's next, any decision needed from Kofi/akorfa before continuing.

---

## 2026-08-25 — Session: Planning

**Done:**
- Confirmed Project 1 domain: Invoice & Payment Tracker API (fintech/operations-flavored CRUD API with JWT auth).
- Created project folder, README.md, and BUILD_PLAN.md (12 sessions, Session 0 through Session 11).
- No code written yet.

**Next:**
- Session 0 — Project skeleton & tooling (FastAPI hello-world app, git repo, venv, requirements.txt).

**Decision needed:** None right now. When ready to start Session 0, just say so and we'll go step by step.

---

## 2026-08-25 — Session 0: Project skeleton & tooling (01-invoice-payment-tracker-api)

**Done:**
- Fixed PowerShell execution-policy block on venv activation (`Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`).
- Created venv, installed FastAPI + Uvicorn, froze `requirements.txt`.
- Built a hello-world FastAPI app (`app/main.py`), confirmed it runs and `/docs` (Swagger UI) loads.
- Added `.gitignore`, initialized git, first commit made.
- Created a public GitHub repo (`invoice-payment-tracker-api`) and pushed.

**Constraint discovered:** No admin/installer rights on this machine, so Docker Desktop is not installable.
**Decision made:** Adjusted the build plan — Session 1 uses a free cloud Postgres (Neon.tech) instead of local Docker Postgres; Session 9 still writes a Dockerfile, but it gets built/run for real by Render's cloud build (Session 10) instead of locally. Updated `BUILD_PLAN.md` and `README.md` in the project folder to reflect this.

**Next:**
- Session 1 — Postgres connection via Neon.tech (no local Docker).

**Decision needed:** None — Neon.tech was chosen as the default (free, no install, no credit card, doesn't expire on a timer like some free-tier competitors). Flag it if you'd rather use something else.

---

## 2026-08-25 — Session 1: Postgres connection via Neon (01-invoice-payment-tracker-api)

**Done:**
- Created Neon.tech project, got connection string, stored it in `.env` (confirmed not tracked by git).
- Installed `sqlalchemy`, `psycopg2-binary`, `python-dotenv`.
- Built `app/database.py` (engine, session factory, `get_db()` dependency) and a `/health` endpoint that runs `SELECT 1` against Neon to prove the connection.
- Committed and pushed (verified: `.env` correctly excluded from the commit).

**Housekeeping note:** `git diff` on this session's files shows line-ending noise (CRLF vs LF) — cosmetic only, doesn't affect functionality, but worth a quick fix so future diffs stay clean. One-time fix, next session: `git config core.autocrlf true`, then a single commit to normalize.

**Next:**
- Session 2 — User model, registration endpoint, password hashing (Alembic migrations start here).

**Decision needed:** None.

---

## 2026-08-26 — Session 2: User model, Alembic migration, registration endpoint (01-invoice-payment-tracker-api)

**Done:**
- `app/models.py` — `User` table (id, email unique, hashed_password, created_at).
- Alembic initialized and configured to read `DATABASE_URL` from `.env` and pick up models automatically; first migration generated and applied — `users` table confirmed in Neon.
- `app/security.py` — password hashing/verification via `bcrypt` directly (skipped `passlib` due to its known compatibility issue with recent bcrypt versions).
- `app/schemas.py` — `UserCreate`/`UserOut` split so hashed passwords can never leak into an API response.
- `POST /auth/register` — creates a user, rejects duplicate emails with 400. Manually verified in Swagger UI.
- First pytest tests (`tests/test_auth.py`) — registration success + duplicate-email rejection, both passing.
- Verified independently (not just by report): checked git history, confirmed `.env` still untracked, read through `main.py`, `models.py`, `schemas.py`, `security.py`, the Alembic migration file, and the test file — all correct and consistent with the plan.
- Committed and pushed.

**Known simplification (flagged, not forgotten):** tests currently write real rows to the live Neon dev database with no cleanup/isolation — acceptable for now, gets fixed properly in Session 8.

**Next:**
- Session 3 — Login endpoint + JWT auth (`get_current_user` dependency).

**Decision needed:** None.

---

## 2026-10-02 — Session 3: JWT login, get_current_user dependency (01-invoice-payment-tracker-api)

**Done:**
- `app/security.py` — `create_access_token`/`decode_access_token` using `python-jose`, reading `SECRET_KEY`/`ALGORITHM`/`ACCESS_TOKEN_EXPIRE_MINUTES` from `.env`.
- `POST /auth/login` — verifies credentials, returns a signed JWT.
- `get_current_user` dependency + `GET /users/me` — proves the protection pattern works (401 without a token, 200 with a valid one).
- `tests/test_login.py` — 4 new tests (login success, wrong password, protected route without/with token), all passing.

**Debugging note (longer than usual — worth remembering):** this session ran across several real environment issues rather than one clean build:
- `SECRET_KEY` was left blank in `.env` at first — generated one with `secrets.token_hex(32)` and filled it in.
- Spent time debugging Swagger UI's Authorize flow for `OAuth2PasswordBearer`, which doesn't take a manually-pasted token the way I'd implied — switched to testing directly via PowerShell's `Invoke-RestMethod` instead, which is more reliable and a better general-purpose habit than relying on Swagger's UI.
- The actual bug once we got there: a typo in `main.py` — `/auth/login` returned the key `"access_taken"` instead of `"access_token"`, so every client reading `response.access_token` silently got `null`. Found it by reading the committed code directly rather than guessing from symptoms. Fixed.
- `tests/test_auth.py::test_register_new_user` fails on repeat runs because it reuses an email already created in the live Neon dev database — this is the known test-isolation gap flagged in Session 2, not a new issue. Left as-is; Session 8 fixes it properly with fixtures.

**Next:**
- Session 4 — Client CRUD, scoped to the logged-in user (first use of the `get_current_user` pattern on a real resource).

**Decision needed:** None.

---

## 2026-10-02 — Session 4: Client CRUD, split into routers (01-invoice-payment-tracker-api)

**Done:**
- Refactored `main.py` into `app/routers/` (`auth.py`, `users.py`, `clients.py`) plus a shared `app/dependencies.py` for `get_current_user` — done ahead of adding Invoice/Payment in the next two sessions, to avoid `main.py` becoming unmanageable.
- `Client` model + migration (`owner_id` foreign key to `users`).
- Full Client CRUD (`POST`/`GET`/`PUT`/`DELETE`), every query scoped by `owner_id == current_user.id`. Cross-user access returns `404`, not `403` (deliberate — doesn't confirm a resource's existence to someone who doesn't own it).
- `tests/test_clients.py` — create, list-only-shows-own, and three cross-user-denial tests (view/update/delete another user's client), all passing.
- Cleaned up `UserOut`/`ClientOut` to use `ConfigDict` instead of the deprecated class-based `Config` (fixes a pytest warning from Session 3).
- 15 pytest tests total passing (the one pre-existing `test_register_new_user` failure from the live-DB test-pollution issue is unrelated, already logged in Session 2/3).

**Process note:** the optional git line-ending normalization step swept the whole session's work into a misleadingly-labeled "Normalize line endings" commit. Caught before push by checking git history directly, fixed with `git commit --amend` (safe pre-push), then pushed correctly as `74c019b`.

**Next:**
- Session 5 — Invoice CRUD + status logic (first derived/business-rule field, not just plain CRUD).

**Decision needed:** None.
