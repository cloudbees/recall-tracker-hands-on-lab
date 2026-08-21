# Module 00 — Before we start

**15 minutes** · Session 1: Unify building blocks

Everything here is pre-work, and it takes about fifteen minutes. Getting it done
beforehand means session one starts with building rather than installing — and gives
your facilitator time to help if anything needs a hand.

The [README](../README.md) describes what you are building and why it is shaped the
way it is. This module is the four accounts and six repositories you need before
session one starts.

---

## What you need

### 1. CloudBees Unify

Your facilitator has created a sub-organization for you. Check the invitation email
and sign in.

**Confirm it works:** sign in and note the organization name in the top-left. It
should be yours, not a shared one. Everything you create today lives here, and it
is deleted after the workshop — so experiment freely.

### 2. GitHub

Any account. You will copy six template repositories into it.

**Confirm it works:** sign in and check you can create a new repository.

### 3. DockerHub

A free account, plus an **access token**.

Sign in, go to **Account Settings → Personal access tokens → Generate new token**,
give it a description and **Read, Write, Delete** permissions, then copy the token
somewhere safe. You cannot view it again after closing the dialog.

> **Worth pausing on — this one catches almost everyone.** An access token is not
> your password. If you paste your password instead, everything looks fine until
> your first build returns a `401`. Module 02 checks this before you build anything,
> so it will be caught either way — a minute here just saves the detour.

**Confirm it works:**

```bash
docker login -u YOUR_USERNAME
# paste the ACCESS TOKEN when prompted for a password
```

No Docker installed? No problem — nothing today requires it locally. Skip the check
and Module 02 will verify the token for you.

### 4. A terminal with `kubectl` (optional)

Everything in this workshop happens in a browser, so this is purely for the curious.
If you would like to look behind the curtain at the running pods, bring `kubectl` and
your facilitator will sort out access.

---

## Copy the six repositories

Each repository below is a **GitHub template**. You are not forking — you are
creating an independent copy with its own history, which you own completely.

For each one: open it, choose **Use this template → Create a new repository**, pick
your own account as the owner, and leave it public or private as you prefer.

| Repository | What it becomes |
|---|---|
| [`recall-db`](https://github.com/cloudbees/recall-db) | Database schema and migrations. Deploys first — everything else needs its tables. |
| [`recall-worker`](https://github.com/cloudbees/recall-worker) | Discovery worker. Pulls live recall data from the FDA. A backend service — nothing outside the cluster can reach it. |
| [`recall-ai-service`](https://github.com/cloudbees/recall-ai-service) | The Recall Advisor. A backend service, and it owns a feature flag. |
| [`recall-core-api`](https://github.com/cloudbees/recall-core-api) | The public API. |
| [`recall-web-ui`](https://github.com/cloudbees/recall-web-ui) | The public web interface. |
| [`app-recall-tracker`](https://github.com/cloudbees/app-recall-tracker) | The Application: no source code, just the workflows that release the other five together. |

> **Keep the names exactly as they appear above.** The Application finds the other
> five components by name, so `recall-db-yourname` will be skipped silently during a
> release — no error, just a component that never deploys. It is fixable later, but
> it means editing a few files, so it is worth a quick double-check here.

---

## Before you arrive

- [ ] Signed in to CloudBees Unify, in your own sub-organization
- [ ] Signed in to GitHub
- [ ] DockerHub **access token** generated and saved somewhere you can paste from
- [ ] All six repositories copied, names unchanged

That's everything. Bring the token, and we will do the rest together.

---

**Next:** [Module 01 — Orientation](01-orientation.md)
