# Product Recall Tracker — Hands-On Lab

Build and release a five-component application with CloudBees Unify, then change
its behaviour live with a feature flag.

## Get your own copies

Each repository below is a **GitHub template**. You are not forking — you are
creating an independent copy with no shared history.

For each: open it, choose **Use this template > Create a new repository**, pick
your own account as owner, and **keep the name exactly as listed**. The
Application workflow references the others by name.

| Repo | What it is |
|---|---|
| [`recall-db`](https://github.com/cloudbees/recall-db) | Database schema and migrations. Deploys first. |
| [`recall-worker`](https://github.com/cloudbees/recall-worker) | Discovery worker. Internal only. |
| [`recall-ai-service`](https://github.com/cloudbees/recall-ai-service) | Recall Advisor. Internal only. Owns a feature flag. |
| [`recall-core-api`](https://github.com/cloudbees/recall-core-api) | Public `/api/*`. |
| [`recall-web-ui`](https://github.com/cloudbees/recall-web-ui) | Public `/`. |
| [`app-recall-tracker`](https://github.com/cloudbees/app-recall-tracker) | The Unify Application and its release workflows. |

## Modules

Authored in Phase 5. Placeholder.

## Questions

Ask your facilitator. If the Unify UI does not match a screenshot exactly, the
product has moved on — the concept will still hold.
