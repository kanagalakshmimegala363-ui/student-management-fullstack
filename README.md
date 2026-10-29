# Student Management System (Fullstack)

Django REST API backend + plain HTML/CSS/JS frontend. Deployable to Vercel as a single project.

## Project structure

```text
.
├── api/index.py               # Vercel serverless entry (exposes Django as `app`)
├── backend/                   # Django project (config/) and students app
├── public/                    # Frontend (index.html, style.css, app.js) served as static files
├── requirements.txt           # Production dependencies (used by Vercel)
├── requirements-local-mysql.txt  # Optional: adds mysqlclient for local MySQL
├── vercel.json
└── .env.example
```

## Run locally

```bash
python -m venv venv
venv\Scripts\activate          # Windows   (macOS/Linux: source venv/bin/activate)
pip install -r requirements.txt
cp .env.example backend/.env   # Windows: copy .env.example backend\.env   then edit it

cd backend
python manage.py migrate
python manage.py runserver
```

- With no database variables set, Django uses a local SQLite file, so it works out of the box.
- For local MySQL: `pip install -r requirements-local-mysql.txt` and set `DB_NAME`, `DB_USER`, `DB_PASSWORD` in `backend/.env`.
- Open `public/index.html` with VS Code Live Server (port 5500). It calls the API at `http://127.0.0.1:8000`.

## Deploy to Vercel

Vercel's filesystem is read-only/temporary, and MySQL (`mysqlclient`) can't be built there, so use a hosted **Postgres** database (Neon, Supabase, or Vercel Postgres).

1. Create a Postgres database and copy its connection string.
2. Create the tables once, from your computer, pointing at that database:
   ```bash
   cd backend
   # Windows PowerShell:  $env:DATABASE_URL="postgresql://..." ; $env:DEBUG="True"
   # macOS/Linux:         export DATABASE_URL="postgresql://..." DEBUG=True
   python manage.py migrate
   python manage.py createsuperuser   # optional, for /admin
   ```
3. Push this repo to GitHub, then **Import** it in Vercel (Framework Preset: *Other*, root directory: the repo root).
4. In Vercel → Project → Settings → Environment Variables add:

   | Name | Value |
   |------|-------|
   | `SECRET_KEY` | a long random string |
   | `DEBUG` | `False` |
   | `DATABASE_URL` | your Postgres connection string |
   | `ALLOWED_HOSTS` | `.vercel.app` (plus your custom domain, if any) |
   | `CSRF_TRUSTED_ORIGINS` | `https://your-project.vercel.app` (only needed for `/admin`) |

5. Deploy. The site is served at `/`, the API at `/api/students/`.

Generate a secret key with:
`python -c "import secrets; print(secrets.token_urlsafe(50))"`

> Note: the Django admin (`/admin`) works but has no CSS on Vercel because static files are not collected. The student app itself does not use the admin.
