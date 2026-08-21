# Product Recall Tracker — Hands-On Lab

Build and release a five-component application with CloudBees Unify, then change its
behaviour live with a feature flag.

## What you are building

**Recall Tracker** helps a company track product recalls that affect it. A user
describes their business, the app searches live FDA and CPSC recall data, and builds a
compliance matrix of what they need to act on.

It is a real application rather than a demo shell — five services with genuine
boundaries between them:

```
                ┌───────────────────┐
you ── HTTPS ──►│   recall-web-ui   │   pages, and the browser-side flags
                └───────────────────┘
                          │ /api
                ┌───────────────────┐
                │  recall-core-api  │   the API surface everything talks to
                └───────────────────┘
                          │
              ┌───────────┴───────────┐
    ┌───────────────────┐   ┌───────────────────┐
    │ recall-ai-service │   │   recall-worker   │   backend services:
    │   Recall Advisor  │   │     FDA / CPSC    │   no route from outside
    └───────────────────┘   └───────────────────┘
              └───────────┬───────────┘
                ┌───────────────────┐
                │     recall-db     │   migrations run first
                └───────────────────┘
```

Only `recall-web-ui` has a route in from the internet, so there is one hostname to
remember rather than five. `recall-db` has to be deployed before everything else, and
`recall-web-ui` after `recall-core-api` — an order the Application defines.

That shape is the reason this workshop exists. A pipeline with a single service is
straightforward. The interesting problems — deployment ordering, coordinated releases,
changing one service's behaviour without touching its neighbours, working out which
service actually broke — only show up once there are five.

## How it fits together in Unify

Each of the five repositories becomes a **Component**: a repository with workflows in
it. A sixth, `app-recall-tracker`, holds no application code — it is the **Application**,
and its workflows release the other five together, in order, through three
**Environments** with an approval gate after the `DEV` environment.

By the end you will have your own running copy with your own URL, and you will change what
it does while it is running, without deploying anything.

| Session | Focus | You finish with |
|---|---|---|
| 1 | The building blocks of Unify | All five components building real images |
| 2 | Release orchestration | One Application releasing all five, gated, across three environments |
| 3 | Feature management | Behaviour you change live, without deploying |

## Modules

| Module | Covers | Time |
|---|---|---|
| [00 — Before we start](docs/00-prerequisites.md) | Accounts and repository copies. Pre-work. | 15 min |
| [01 — Orientation](docs/01-orientation.md) | The vocabulary and the map. | 25 min |
| [02 — Configuration](docs/02-configuration.md) | Every value set, and Verify Setup passing. | 30 min |
| [03 — Your first component](docs/03-first-component.md) | `recall-db` built, pushed and deployed. | 45 min |
| [04 — The remaining components](docs/04-remaining-components.md) | The other four built. Five images, five artifacts. | 45 min |

## Questions

Raise your hand and ask the facilitator.
