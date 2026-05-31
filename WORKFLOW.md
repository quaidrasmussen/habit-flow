# Workflow — Deploying Habit Flow

Two environments, both on Vercel, both auto-deploy on push to GitHub (`quaidrasmussen/habit-flow`). No build step — it serves `index.html` directly.

| Env | Branch | URL |
|-----|--------|-----|
| **Production** (live, on both phones) | `main` | https://habit-flow-eight-delta.vercel.app |
| **Staging** (Quaid only, for testing) | `staging` | https://habit-flow-git-staging-quaid-rasmussen-s-projects.vercel.app |

All work happens on `staging` first. Never edit on `main` directly.

## 1. Make changes on staging
```bash
git checkout staging
# ...edit index.html...
git add -A
git commit -m "Describe the change"
git push origin staging
```
Pushing to `staging` auto-deploys to the **staging URL** in a few seconds. Open it on your phone or browser and test.

## 2. Ship to production
Once staging looks good, merge it into `main`:
```bash
git checkout main
git merge staging
git push origin main
```
Pushing to `main` auto-deploys to the **production URL**. The PWA on both phones picks it up on next open (hard-refresh if needed).

## 3. Back to work
```bash
git checkout staging
```

## Notes
- `main` should always equal the last known-good staging state. If `staging` is ahead of `main`, there are changes tested but not yet shipped.
- The local repo is linked to Vercel (`.vercel/`, gitignored). The Vercel CLI is installed if you ever want to deploy or inspect manually (`vercel ls`, `vercel deploy`).
- No manual deploy step is required for the normal flow — push is the deploy.
