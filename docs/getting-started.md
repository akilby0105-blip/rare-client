cd# Getting Started — Rare

Rare is a full-stack blogging platform. The backend (Django + PostgreSQL) lives in `rare-api/` and the frontend (React) lives in `rare-client/`. You need both running at the same time to use the app.

---

## Prerequisites

| Tool | Purpose | Notes |
|---|---|---|
| Docker Desktop | Runs the PostgreSQL database | Must be running before any API work |
| Python | Django API | Any version — Pipfile has no version pin |
| pipenv | Python dependency manager | `pip install pipenv` if missing |
| Node.js + npm | React client | Node 16+ recommended |

---

## 1. Start the database

The database runs in Docker. From the `rare-api/` directory:

```bash
docker compose up -d
```

This starts a PostgreSQL 16 container with:
- **Host:** `localhost:5432`
- **Database:** `rare`
- **User:** `rare_user`
- **Password:** `rare_password`

Data is persisted in a Docker volume (`rare_db_data`) so it survives container restarts. To wipe it completely and start fresh:

```bash
docker compose down -v
docker compose up -d
```

---

## 2. Set up the API

All of the following commands run from the `rare-api/` directory.

**Install Python dependencies:**

```bash
pip install pipenv
```

```bash
python -m pipenv install
```

```bash
python -m pipenv shell
```

**Apply database migrations:**

```bash
pipenv run python manage.py migrate
```

This creates all tables in the `rare` database. Run this again whenever you pull changes that include new migration files.


## 3. Seed the database

The fixture at `rare-api/rareapi/fixtures/initial_data.json` loads a full set of sample data: 12 users (2 admins, 10 regular), 12 categories, 20+ tags, 36 approved posts, 3 unapproved posts, comments, reactions, and subscriptions. Make sure to be in the right directory before running.

```bash
pipenv run python manage.py loaddata rareapi/fixtures/initial_data.json
```

Run this once after migrating. If you run it again on a populated database, it will overwrite the records with matching PKs (safe to re-run, no duplicates).

---
**Start the API server:**

```bash
pipenv run python manage.py runserver 8088
```

The API listens on `http://localhost:8088`. If you're using the VS Code launch config (`.vscode/launch.json`), the **Python: Django** configuration runs this same command — press F5 to start with the debugger attached.

---



## 4. Set up and start the client in a new terminal

All of the following commands run from the `rare-client/` directory.

**Install JavaScript dependencies:**

```bash
npm install
```

**Start the dev server:**

```bash
npm start
```

The client runs at `http://localhost:3000` and hot-reloads on file changes. It talks to the API at `http://localhost:8088` — this URL is hardcoded in `src/managers/api.js`.

---

## 5. Verify everything is working

With both servers running, open `http://localhost:3000`. You should see the login page. Log in with any seed account (see below) and confirm that the post list loads.

If the post list is empty after login, the fixture probably hasn't been loaded — run step 3.

---

## Seed accounts

All fixture accounts share the same password. The password hash in the fixture file is identical across all users. If you don't know the plaintext password, ask whoever set up the fixture, or reset it for any account with:

```bash
pipenv run python manage.py changepassword <username>
```

### Admin accounts (`is_staff = True`)

Admins can approve/unapprove posts, manage tags and categories, view the demotion queue, and access all admin-only routes in the UI.

| Username | Name |
|---|---|
| `admin_sarah` | Sarah Chen |
| `admin_marcus` | Marcus Johnson |

### Regular user accounts (`is_staff = False`)

| Username | Name | Writes about |
|---|---|---|
| `dev_diana` | Diana Reeves | Technology |
| `wanderlust_joe` | Joe Nakamura | Travel |
| `chef_maya` | Maya Patel | Food & Cooking |
| `bookworm_alex` | Alex Torres | Books |
| `fit_jordan` | Jordan Blake | Health & Fitness |
| `gamer_priya` | Priya Sharma | Gaming |
| `eco_oliver` | Oliver Green | Environment |
| `music_luna` | Luna Martinez | Music |
| `startup_raj` | Raj Gupta | Business |
| `photo_emma` | Emma Larsson | Photography |

### Useful test scenarios in the seed data

- **Moderation queue:** There are 3 unapproved posts (by `dev_diana`, `gamer_priya`, and `chef_maya`). Log in as `admin_sarah` or `admin_marcus` and go to `/unapprovedposts` to see them.
- **Admin-only routes:** Log in as a regular user and try navigating to `/unapprovedposts` — you'll be redirected to `/`.
- **Comments and reactions:** Most posts from January–March 2026 have existing comments and emoji reactions seeded.

---

## Orientation: running order checklist

```
[ ] Docker Desktop is running
[ ] docker compose up -d          (from rare-api/)
[ ] pipenv install                 (from rare-api/, first time only)
[ ] pipenv run python manage.py migrate
[ ] pipenv run python manage.py loaddata rareapi/fixtures/initial_data.json
[ ] pipenv run python manage.py runserver 8088
[ ] npm install                    (from rare-client/, first time only)
[ ] npm start
[ ] Open http://localhost:3000
```

---

## Common problems

**`django.db.utils.OperationalError: could not connect to server`**
Docker isn't running, or the container hasn't started yet. Check `docker ps` — you should see a `postgres:16` container.

**Port 8088 already in use**
Something else is on that port. Find and kill it (`lsof -i :8088` on Mac/Linux), or change the port in both `manage.py runserver <port>` and `rare-client/src/managers/api.js`.

**`Module not found` on `npm start`**
Run `npm install` first. This happens when someone added a new dependency and you haven't pulled their lockfile changes yet.

**Login succeeds but the post list is empty**
The fixture hasn't been loaded. Run the `loaddata` command in step 3.

**Posts I create don't appear on the main feed**
Regular users' posts start as `approved=False` and are hidden from all public lists until an admin approves them. Log in as an admin and go to `/unapprovedposts`.
