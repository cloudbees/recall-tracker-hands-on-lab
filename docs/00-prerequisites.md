# Module 00 — Before we start

**15 minutes** · Session 1: Unify building blocks

Everything here is pre-work, and it takes about fifteen minutes. Getting it done
beforehand allows session one to start with building rather than setting up accounts.

---

## What you need

### 1. CloudBees Unify

Your facilitator has created a sub-organization for you. Check the invitation email
and sign in.

**Confirm it works:** sign in and note the organization name in the top-left. It
should be the name of the workshop, but you can click the expand button to select
your own workspace. Everything you create today lives here, belongs to you, and 
is deleted after the workshop — so experiment freely.

<img width="380" height="71" alt="image" src="https://github.com/user-attachments/assets/a20d54e5-fe15-4fb1-a3c3-8930d42aec74" />

### 2. GitHub

Any account. You will copy six template repositories into it.

### 3. DockerHub

A free account, plus an **access token**.

[Sign in](https://app.docker.com), go to **Account Settings → Personal access tokens
→ Generate new token**, give it a description and **Read, Write, Delete** permissions,
then copy the token somewhere safe. You cannot view it again after closing the dialog.

---

## Copy the six repositories

Each repository below is a **GitHub template**. You are not forking — you are
creating an independent copy with its own history, which you own completely.

For each one: open it, choose **Use this template → Create a new repository**, pick
your own account as the owner, and leave it public or private as you prefer.

> **Keep the names exactly as they appear below.** The Application finds the other
> five components by name, so `recall-db-yourname` will be skipped silently during a
> release — no error, just a component that never deploys. It is fixable later, but
> it means editing a few files, so it is worth a quick double-check here.

| Repository | What it becomes |
|---|---|
| [`recall-db`](https://github.com/cloudbees/recall-db) | Database schema and migrations. Deploys first — everything else needs its tables. |
| [`recall-worker`](https://github.com/cloudbees/recall-worker) | Discovery worker. Pulls live recall data from the FDA. A backend service — nothing outside the cluster can reach it. |
| [`recall-ai-service`](https://github.com/cloudbees/recall-ai-service) | The Recall Advisor. A backend service, and it owns a feature flag. |
| [`recall-core-api`](https://github.com/cloudbees/recall-core-api) | The public API. |
| [`recall-web-ui`](https://github.com/cloudbees/recall-web-ui) | The public web interface. |
| [`app-recall-tracker`](https://github.com/cloudbees/app-recall-tracker) | The Application: no source code, just the workflows that release the other five together. |

---

## Before you arrive

- [ ] Signed in to CloudBees Unify and navigated to your own sub-organization
- [ ] Signed in to GitHub
- [ ] DockerHub **access token** generated and saved
- [ ] Six GitHub repositories copied, names unchanged

---

**Next:** [Module 01 — Orientation](01-orientation.md)
