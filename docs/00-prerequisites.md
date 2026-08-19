# Module 00 — Before we start

**15 minutes** · Session 1: Unify building blocks

Everything here is pre-work, and it takes about fifteen minutes. Getting it done
beforehand means session one starts with building rather than installing — and gives
your facilitator time to help if anything needs a hand.

We are going to take a real application — five services, a database, an AI
assistant — and put it through CloudBees Unify end to end. Build it, release it
through three environments with an approval gate, then change how it behaves for
your users without deploying anything at all.

By the end of session three you will have your own running copy at your own URL,
and you will change what it does while it is running.

To get there, we need four accounts and six repositories.

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
| [`recall-worker`](https://github.com/cloudbees/recall-worker) | Discovery worker. Pulls live recall data from the FDA. Internal only. |
| [`recall-ai-service`](https://github.com/cloudbees/recall-ai-service) | The Recall Advisor. Internal only, and it owns a feature flag. |
| [`recall-core-api`](https://github.com/cloudbees/recall-core-api) | The public API. |
| [`recall-web-ui`](https://github.com/cloudbees/recall-web-ui) | The public web interface. |
| [`app-recall-tracker`](https://github.com/cloudbees/app-recall-tracker) | The Application: no source code, just the workflows that release the other five together. |

> **Keep the names exactly as they appear above.** The Application finds the other
> five components by name, so `recall-db-yourname` will be skipped silently during a
> release — no error, just a component that never deploys. It is fixable later, but
> it means editing a few files, so it is worth a quick double-check here.

---

## The five-minute version of what you are building

**Recall Tracker** helps a company track product recalls that affect it. A user
describes their business, the app searches live FDA and CPSC recall data, and builds
a compliance matrix of what they need to act on.

It is a real application rather than a demo shell — five services with genuine
boundaries between them:

```
                          ┌──────────────┐
    you ────── HTTPS ────►│  recall-web-ui  │  pages, and the browser-side flags
                          └───────┬──────┘
                                  │ /api
                          ┌───────▼──────┐
                          │ recall-core-api │  the public API surface
                          └───┬───────┬──┘
                    ┌─────────┘       └─────────┐
          ┌─────────▼────────┐       ┌──────────▼───────┐
          │ recall-ai-service │       │  recall-worker   │  internal only —
          │  Recall Advisor   │       │  FDA / CPSC      │  no route from outside
          └─────────┬────────┘       └──────────┬───────┘
                    └──────────┬────────────────┘
                        ┌──────▼──────┐
                        │  recall-db  │  migrations run before anything starts
                        └─────────────┘
```

That shape is the reason this workshop exists. A pipeline with a single service is
straightforward. The interesting problems — deployment ordering, coordinated
releases, changing one service's behaviour without touching its neighbours — only
show up once there are five.

---

## How the three sessions fit together

| Session | Focus | You finish with |
|---|---|---|
| 1 | The building blocks of Unify | All five components building real images |
| 2 | Release orchestration | One Application releasing all five, gated, across three environments |
| 3 | Feature management | Behaviour you change live, without deploying |

---

## Before you arrive

- [ ] Signed in to CloudBees Unify, in your own sub-organization
- [ ] Signed in to GitHub
- [ ] DockerHub **access token** generated and saved somewhere you can paste from
- [ ] All six repositories copied, names unchanged

That's everything. Bring the token, and we will do the rest together.

---

**Next:** [Module 01 — Orientation](01-orientation.md)
