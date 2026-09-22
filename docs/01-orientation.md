# Module 01 — Orientation

**25 minutes** · Session 1: Unify building blocks

This module will cover Unify vocabulary and the application map — six words and one
diagram. Everything in the next nine modules will use the terminology described here,
so it is worth the read.

At the end, you will connect your repositories to Unify.

---

## Vocabulary

### Organization

Where everything lives, and the boundary for who can see it. Organizations nest: your
sub-organization sits under the one your facilitator created, and inherits environment
variables.

These can be viewed in _Configurations_ in the left nav bar.

### Component

**A functional repository with workflows to define actions.**

You copied six repositories in Module 00. Five of these will become Components.

A Component is not a service, container, or deployment. It is a reference to a
repository. What that repository *does* is defined the workflows inside it.

### Workflow

A YAML file in `.cloudbees/workflows/` describing jobs and steps. It uses a workflow
DSL similar to GitHub Actions.

Each of your five components has two: `build.yaml` and `deploy.yaml`.

### Application

**A logical repository that releases several Components together.**

Your sixth repository, `app-recall-tracker`, holds no application code at all — only
workflows that call the other five. When you run a release, the Application decides
what gets built, in what order, into which environment, and what has to be approved
before it goes further.

> A Component answers "how do I build and deploy this one thing." An Application
> answers "how do these five things go out together."

### Environment

A named deployment target — `DEV`, `STAGING`, `PROD` — with its own configuration
attached. The same workflow deploys to all three; what differs is the values it reads.

**You will create your own environments in the next module.** They are not handed to you, and
that is deliberate.

### Feature flag

A switch you change in the Unify UI that changes what the running application does,
with no build or deploy. Session 3 will explore feature flags and let you deploy
your own feature.

---

## What you are building

Five components, one Application, three environments.

| Component | Owns | Reachable from outside? |
|---|---|---|
| `recall-db` | Schema and migrations. Also seeds the demo accounts you will sign in as. | No |
| `recall-worker` | Discovery. Calls the live FDA and CPSC recall APIs. | No |
| `recall-ai-service` | The Recall Advisor, and the feature flag that gates it. | No |
| `recall-core-api` | The API surface everything else talks to. | No |
| `recall-web-ui` | Pages, and the browser-side flags. | **Yes** — this is your URL |

Only `recall-web-ui` has a route in from the internet. The other four are reachable
only from inside your namespace, which is why there is exactly one hostname.

### Order matters

```
  recall-db          migrations run, tables exist
      │
      ├── recall-worker
      ├── recall-ai-service        these three need the tables
      └── recall-core-api
                │
          recall-web-ui            needs the API
```

`recall-db` deploys first and everything else waits for it. If you deploy
`recall-core-api` into an empty database and the pod starts, it passes its
health check, but returns errors on every request — a failure that looks
like a bug in the API as opposed to the database.

Encoding that order is handled by the Application. You will write it in Module 05.

---

## Configuration scope

Worth noting before the next module, because it is the single most common thing to
get wrong.

| Scope | Applies to | Examples |
|---|---|---|
| **Organization** | Everything across the organization and any sub-organizations | Shared credentials, global properties |
| **Sub-Organization** | Everything specific to your sub-organization | Your DockerHub account, your cluster credentials |
| **Environment** | One environment only | `namespace`, `hostname` — different for `DEV` and `PROD` |

Module 02 is where you set all of this, and where a workflow called **Verify Setup**
checks it for you before anything is built.

---

## One thing to do: let Unify see your code

A Component is a repository with workflows in it — but Unify cannot read your
repositories until you say so.

In your sub-organization, add a **GitHub integration**. This can be found in Configurations
from the side bar, then the Integrations tab. Unify sends you to GitHub to
install its app on your account, and GitHub asks which repositories the app may see:

- Choose **Only select repositories**, and select all six.
- Not **All repositories** — that grants access to everything in your account,
  including private repositories with nothing to do with this workshop.

You do this **once**. Every later module picks from the repositories you granted here,
so nothing needs installing again in Modules 03 and 04.

---

## Before you move on

- [ ] All six repositories granted to the CloudBees GitHub app

You should also be able to answer these without looking. If one is fuzzy, ask now —
every later module assumes all four.

- [ ] What is the difference between a Component and an Application?
- [ ] Why does `recall-db` deploy before anything else?
- [ ] Which of your five components can be reached from the internet?
- [ ] Where do `namespace` and `hostname` need to be set?

---

**Next:** [Module 02 — Configuration](02-configuration.md)
