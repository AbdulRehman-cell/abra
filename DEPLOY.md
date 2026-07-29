# Deploy Guide — Static Site on Render

This project is a **plain static HTML/CSS/JS site** with no build step. It is served in production by **nginx inside a Docker container**, deployed on **Render** using `render.yaml`.

You can go live in under 5 minutes.

---

## Prerequisites

- A [Render](https://render.com) account (free to sign up)
- This repository pushed to GitHub, GitLab, or Bitbucket
- (Optional, for local testing) [Docker](https://www.docker.com/) installed

---

## Option A: Deploy to Render in 3 commands (via CLI)

```bash
# 1. Install the Render CLI (if not already installed)
brew install render

# 2. Authenticate with Render
render login

# 3. Deploy using the render.yaml blueprint in this repo
render blueprint launch
```

Render will detect `render.yaml`, build the Docker image, and deploy the `static-site` web service automatically. Your site will be live at the URL Render assigns (e.g. `https://static-site.onrender.com`).

---

## Option B: Deploy via Render Dashboard (no CLI, beginner-friendly)

1. Push this repo to GitHub.
2. Go to https://dashboard.render.com → **New** → **Blueprint**.
3. Connect your repository. Render auto-detects `render.yaml`.
4. Click **Apply** — Render builds the Dockerfile and deploys.
5. Once the health check at `/healthz` passes, your site is live.

That's it — no environment variables are required to get started.

---

## Local Testing Before Deploy

Test the exact production image locally:

```bash
docker build -t static-site:1.0.0 .
docker run -p 8080:8080 static-site:1.0.0
```

Visit http://localhost:8080 — you should see `index.html`.

Or with Docker Compose:

```bash
docker compose up --build
```

---

## Enabling Automated CI/CD (GitHub Actions)

The included `.github/workflows/deploy.yml` will:
1. Lint HTML
2. Build the Docker image
3. Run a smoke/health test
4. Trigger a Render deploy hook on push to `main`

To enable step 4:

1. In Render Dashboard → your service → **Settings** → **Deploy Hook**, copy the URL.
2. In GitHub repo → **Settings** → **Secrets and variables** → **Actions**, add:
   - Name: `RENDER_DEPLOY_HOOK_URL`
   - Value: *(paste the hook URL)*
3. Push to `main` — deployment now happens automatically after tests pass.

---

## Environment Variables

None are required. See `.env.example` for optional/informational variables (`SITE_ENV`, `PORT`).

---

## Troubleshooting

| Issue | Fix |
|---|---|
| Health check fails on Render | Confirm `healthCheckPath: /healthz` matches `default.conf` and port `8080` |
| 404 on a page | Ensure the `.html` file exists at repo root and is referenced correctly |
| Styles/JS not loading | Check `assets/` cache headers aren't stale; hard-refresh browser |
| Local Docker build fails | Ensure Docker Desktop is running and you're on the repo root directory |