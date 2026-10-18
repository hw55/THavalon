# THavalon Release Strategy

## Overview

THavalon is deployed on **Render.com (free tier)** at `thavalon.onrender.com`.
Deployments are **manually triggered** via GitHub Actions — never automatic.

## Architecture

```
GitHub (hw55/THavalon)

  main ──► CI tests (on every push/PR)
    │
    ├──► "Deploy to Staging" button (Actions tab)
    │      └──► POST to Render staging deploy hook
    │
    └──► "Deploy to Production" button (Actions tab)
           └──► POST to Render production deploy hook
                (requires typing "deploy" to confirm)

         │                          │
         ▼                          ▼
┌──────────────────┐   ┌──────────────────────┐
│ Render: Staging  │   │ Render: Production   │
│ auto-deploy: OFF │   │ auto-deploy: OFF     │
│ thavalon-staging │   │ thavalon.onrender    │
│ .onrender.com    │   │ .com                 │
└──────────────────┘   └──────────────────────┘
```

Both Render services track `main`. Neither auto-deploys.
Staging is deployed first to test; production is deployed after verification.

## Release Flow

1. **Develop** — merge PRs into `main`. CI runs tests automatically.
2. **Stage** — click "Deploy to Staging" in the GitHub Actions tab.
   Test at `thavalon-staging.onrender.com`.
3. **Release** — click "Deploy to Production" in the GitHub Actions tab.
   Type `deploy` to confirm. Live at `thavalon.onrender.com`.

## Design Principles

| Principle | How |
|---|---|
| No accidental deploys | Auto-deploy is OFF on Render; deploys require a manual button click |
| Owner-only releases | `workflow_dispatch` is only visible to repo collaborators |
| Test before promote | Staging service exists at a separate URL for pre-release testing |
| Confirmation gate | Production deploy requires typing "deploy" to proceed |

## Render Free Tier Constraints

| Constraint | Impact |
|---|---|
| 750 instance hours/month (shared) | Fine — staging spins down when not in use |
| 15-min inactivity spin-down | Cold starts ~30–60s on first hit |
| 512 MB RAM, ~0.1 vCPU | App uses SerialGC + 70% RAM cap; fits comfortably |
| No persistent disk on free tier | Game data in `/data` does not survive redeploys |
| 5 GB outbound bandwidth/month | Sufficient for a game lobby app |

## GitHub Secrets Required

| Secret | Purpose |
|---|---|
| `RENDER_PROD_DEPLOY_HOOK_URL` | Deploy hook URL for the production Render service |
| `RENDER_STAGING_DEPLOY_HOOK_URL` | Deploy hook URL for the staging Render service |

## Setup Checklist

- [ ] Disable auto-deploy on the production Render service
- [ ] Create Deploy Hook for production (Render Dashboard → Settings)
- [ ] Create a second free Render web service `thavalon-staging`, auto-deploy OFF
- [ ] Create Deploy Hook for staging
- [ ] Add `RENDER_PROD_DEPLOY_HOOK_URL` as a GitHub Actions secret
- [ ] Add `RENDER_STAGING_DEPLOY_HOOK_URL` as a GitHub Actions secret
