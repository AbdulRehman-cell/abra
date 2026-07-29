# Deploy Guide — Static Site on Render

This repository is a **plain static HTML/CSS/JS website**. There is no build step, no npm, no backend. It is served via **nginx inside a Docker container** on **Render**.

You can deploy this in under 5 minutes using either method below.

---

## Option A: Deploy via Render Dashboard (no CLI needed)

1. Push this repo to GitHub (if not already there).
2. Go to https://dashboard.render.com → **New** → **Blueprint**.
3. Connect your GitHub repo. Render will detect `render.yaml` automatically.
4. Click **Apply** — Render builds the Docker image and deploys it.
5. Once live, Render gives you a URL like `https://static-site.onrender.com`.

That's it — no environment variables are required.

---

## Option B: Deploy via Render CLI

```bash
# 1. Install the Render CLI
brew install render   # or: curl -fsSL https://render.com/download-cli.sh | sh

# 2. Log in
render login

# 3. Deploy using the blueprint in this repo
render blueprint launch
```

---

## Option C: Test locally first (recommended before deploying)

```bash
# 1. Build the Docker image
docker build -t static-site:local .

# 2. Run it locally
docker run --rm -p 8080:8080 static-site:local

# 3. Open in browser
open http://localhost:8080
```

Or with Docker Compose:

```bash
docker compose up --build
```

Visit `http://localhost:8080` — you should see `index.html`. Check `http://localhost:8080/healthz` returns `ok`.

---

## Setting up CI/CD (GitHub Actions → auto-deploy on push to `main`)

1. In Render Dashboard, go to your service → **Settings** → copy the **Service ID**.
2. Go to Render **Account Settings → API Keys** → create a new API key.
3. In your GitHub repo, go to **Settings → Secrets and variables → Actions** and add:
   - `RENDER_API_KEY` = your Render API key
   - `RENDER_SERVICE_ID` = your Render service ID
4. Push to `main` — GitHub Actions will build, smoke-test, then trigger a Render deploy automatically.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Build fails on Render | Check the Dockerfile builds locally first: `docker build -t test .` |
| Health check failing | Confirm `/healthz` returns `200 ok` locally on port 8080 |
| 404 on a page | Ensure the `.html` file exists at repo root and is copied in the Dockerfile |
| Port mismatch | Render requires the container to listen on the port set by `$PORT`, but this Dockerfile fixes nginx to `8080` and Render auto-detects it via `EXPOSE 8080` — no changes needed |

---

## Files Reference

- `Dockerfile` — multi-stage build; copies static files into nginx image
- `nginx.conf` — serves site, adds security headers, gzip, `/healthz` endpoint
- `render.yaml` — Render Blueprint (Docker runtime, health check path)
- `docker-compose.yml` — local dev/test convenience
- `.github/workflows/deploy.yml` — CI build/test + auto-deploy to Render
- `.env.example` — optional CI secrets reference (no runtime env vars needed)