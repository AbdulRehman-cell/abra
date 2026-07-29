# Deploy Guide — ForgeAI Static Site on Render

This is a **plain static HTML/CSS/JS site** served through a minimal, pinned
`nginx` Docker image. No build tools, no Node, no npm — just files served
over HTTP with security headers, gzip, caching, and a health check.

You can be live in under 5 minutes.

## Prerequisites

- A [Render](https://render.com) account (free to sign up)
- This repo pushed to GitHub/GitLab
- Docker installed locally (only needed for local testing, optional)

---

## Option A: Deploy to Render via Blueprint (fastest)

1. Push this repo to GitHub (if not already):
   ```bash
   git add .
   git commit -m "Add production deployment config"
   git push origin main
   ```

2. Go to the Render Dashboard → **New** → **Blueprint**, and connect this
   repository. Render will auto-detect `render.yaml` and configure the
   service for you.

3. Click **Apply** — Render will build the Docker image and deploy it.
   Your site will be live at `https://<your-service-name>.onrender.com`.

That's it — 3 steps, no manual service configuration needed.

---

## Option B: Manual Render Web Service (Docker runtime)

1. Render Dashboard → **New** → **Web Service** → connect your repo.
2. Runtime: **Docker**. Dockerfile path: `./Dockerfile`.
3. Health Check Path: `/healthz`.
4. Click **Create Web Service**.

---

## Enable Auto-Deploy from GitHub Actions (optional but recommended)

1. In Render, go to your service → **Settings** → **Deploy Hook** → copy the URL.
2. In GitHub, go to your repo → **Settings** → **Secrets and variables** →
   **Actions** → **New repository secret**:
   - Name: `RENDER_DEPLOY_HOOK_URL`
   - Value: (paste the deploy hook URL)
3. Every push to `main` will now: validate HTML → build & smoke-test the
   Docker image → trigger a Render deploy automatically.

---

## Test Locally Before Deploying

```bash
docker compose up --build
curl http://localhost:8080/healthz
```

Visit `http://localhost:8080` in your browser. Stop with `docker compose down`.

---

## The 3 Commands to Deploy Right Now

```bash
git push origin main
# Then in Render dashboard: New -> Blueprint -> select this repo -> Apply
curl https://<your-service-name>.onrender.com/healthz
```

---

## Troubleshooting

- **Build fails on Render**: confirm `Dockerfile` and `nginx.conf` are at the
  repo root and committed.
- **Health check failing**: ensure port `8080` is exposed and `/healthz`
  returns `200 ok` (test locally with the curl command above).
- **404 on routes like `/about`**: the nginx config uses
  `try_files $uri $uri.html $uri/ =404;` so `about.html` is served at `/about`.
  Ensure filenames match exactly (case-sensitive).