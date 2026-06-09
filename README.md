# n8n GitOps Demo — Buzzwords 2026

This repository manages n8n workflows as code using [n8n-gitops](https://github.com/n8n-gitops/n8n-gitops).

Once set up, the git hooks take care of everything automatically:
- **`git pull`** → deploys the latest workflows from Git into your local n8n instance
- **`git commit`** (even empty) → exports workflows from your n8n instance into Git

---

## Setup

### 1. Clone the repository

```bash
git clone git@github.com:n8n-gitops/n8n-gitops-demo-buzzwords-2026.git
cd n8n-gitops-demo-buzzwords-2026
```

### 2. Start n8n

If you don't have an n8n instance yet, run it with Docker:

```bash
docker run -it --rm \
  --name n8n \
  -p 15678:5678 \
  -v n8n_data:/home/node/.n8n \
  docker.n8n.io/n8nio/n8n
```

n8n will be available at `http://localhost:15678`. Complete the initial setup, then proceed.

### 3. Install n8n-gitops

Using [uv](https://github.com/astral-sh/uv) (recommended):

```bash
uv tool install n8n-gitops
```

Or with pipx:

```bash
pipx install n8n-gitops
```

### 4. Get your n8n API credentials

1. Open your n8n instance
2. Go to **Settings → N8N API → Create API Key**
3. Copy the API key and your instance URL (e.g. `http://localhost:5678`)

### 5. Install git hooks

Run the following and follow the prompts for your config profile name, API URL, and API key:

```bash
n8n-gitops install-hooks
```

This creates a `.n8n-gitops.yaml` in the repo root with your credentials and installs two git hooks:

| Hook         | Trigger                  | What it does                     |
|--------------|--------------------------|----------------------------------|
| `pre-commit` | `git commit`             | Exports workflows from n8n → Git |
| `post-merge` | `git pull` / `git merge` | Deploys workflows from Git → n8n |

> Your `.n8n-gitops.yaml` is ignored by Git — credentials stay local.

---

## Daily workflow

### Sync workflows from n8n into Git

```bash
git add n8n/
git commit -m "your message"   # or: git commit --allow-empty -m "sync"
git push
```

The `pre-commit` hook exports all workflows from your n8n instance before the commit is created.
If workflows changed, they are staged automatically — just commit again to include them.

### Pull the latest workflows from Git into n8n

```bash
git pull
```

The `post-merge` hook deploys the updated workflows into your n8n instance automatically.

---

## CI/CD

| Event           | Workflow              | Target             |
|-----------------|-----------------------|--------------------|
| Push to `main`  | `deploy-staging.yaml` | Staging            |
| Tag `v*.*.*`    | `deploy-prod.yaml`    | Production         |
| Manual dispatch | `deploy-prod.yaml`    | Production         |
| PR → `main`     | `validate.yaml`       | Offline validation |

Required GitHub secrets:

| Secret                | Used by           |
|-----------------------|-------------------|
| `N8N_STAGING_URL`     | Staging deploy    |
| `N8N_STAGING_API_KEY` | Staging deploy    |
| `N8N_PROD_URL`        | Production deploy |
| `N8N_PROD_API_KEY`    | Production deploy |