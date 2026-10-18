# THavalon Release Strategy

## Overview

THavalon is deployed on **Render.com (free tier)** at `thavalon.onrender.com`.

- **PR Previews (Staging):** Every pull request automatically gets an isolated, ephemeral preview server (e.g., `https://thavalon-pr-X.onrender.com`) to test changes before merging.
- **Production Releases:** Deployments to production are **manually triggered** via GitHub Actions — never automatic.

---

## Architecture

```
GitHub (hw55/THavalon)

  Feature Branch / PR ──► Render PR Preview (automatic)
                           (thavalon-pr-X.onrender.com)
                           • Isolated test environment
                           • Automatically destroyed on PR merge / close

  Merge to main ──► CI tests run
    │
    └──► "Deploy to Production" button (Actions tab)
           └──► POST to Render production deploy hook
                (requires typing "deploy" to confirm)
                         │
                         ▼
              ┌──────────────────────┐
              │ Render: Production   │
              │ auto-deploy: OFF     │
              │ thavalon.onrender    │
              │ .com                 │
              └──────────────────────┘
```

---

## Release Flow

1. **Develop** — create a feature branch and open a Pull Request into `main`.
2. **Preview & Test** — Render automatically builds and deploys an ephemeral preview environment at `https://thavalon-pr-<PR_NUMBER>.onrender.com`. Test your changes live on this URL.
3. **Merge** — when tests pass, merge the PR into `main`. Render automatically destroys the preview environment.
4. **Release** — go to the **Actions** tab on GitHub, select **"Deploy to Production"**, and type `deploy` to confirm. Render deploys the new code to `thavalon.onrender.com`.

---

## Design Principles

| Principle | How |
|---|---|
| Isolated preview per PR | Render spins up a dedicated preview URL per PR |
| Automatic cleanup | Preview servers auto-delete when PR is merged or closed |
| No accidental prod releases | Auto-deploy is OFF on production; deploys require manual confirmation |
| Owner-only releases | GitHub Actions `workflow_dispatch` is restricted to repo collaborators |

---

## Render Free Tier Constraints

| Constraint | Impact |
|---|---|
| 750 instance hours/month (shared) | Previews sleep after 15 min of inactivity and auto-delete on PR close |
| 15-min inactivity spin-down | Cold starts ~30–60s on first hit |
| 512 MB RAM, ~0.1 vCPU | App uses SerialGC + 70% RAM cap; fits comfortably |
| No persistent disk on free tier | Game data in `/data` does not survive redeploys |
| 5 GB outbound bandwidth/month | Sufficient for a game lobby app |

---

## GitHub Secrets Required

| Secret | Purpose |
|---|---|
| `RENDER_PROD_DEPLOY_HOOK_URL` | Deploy hook URL for the production Render service |
