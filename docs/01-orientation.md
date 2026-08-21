# Module 01 — Orientation

**25 minutes** · Session 1: Unify building blocks

Nothing to build in this module. It is the vocabulary and the map — six words and one
diagram. Everything in the next nine modules is a variation on what is here, so it is
worth the twenty-five minutes.

---

## Six words

Unify's vocabulary is small. Most of the confusion people have with it comes from two
of these words sounding interchangeable when they are not.

### Organization

Where everything lives, and the boundary for who can see it. Organizations nest: your
sub-organization sits under the one your facilitator created, and it inherits
configuration from above.

You will not see most of what you inherit. Database credentials and API keys are set
once at the parent and flow down, which is why you have a working database without
anyone handing you a password.

### Component

**A repository with workflows in it.** That is the whole idea.

You copied six repositories in Module 00. Five of them become Components — Unify
watches the repository, notices the `.cloudbees/workflows/` directory, and offers to
run what it finds there.

A Component is not a service, a container, or a deployment. It is a repository. What
that repository *does* is entirely up to the workflows inside it.

### Workflow

A YAML file in `.cloudbees/workflows/` describing jobs and steps. If you have used
GitHub Actions the shape will look familiar; the differences are real but small enough
to learn as you go.

Each of your five components has two: `build.yaml` and `deploy.yaml`.

### Application

**The thing that releases several Components together.** This is the word that earns
the workshop.

Your sixth repository, `app-recall-tracker`, holds no application code at all — only
workflows that call the other five. When you run a release, the Application decides
what gets built, in what order, into which environment, and what has to be approved
before it goes further.

> A Component answers "how do I build and deploy this one thing." An Application
> answers "how do these five things go out together." If you remember only one thing
> from this module, make it that sentence.

### Environment

A named deployment target — `DEV`, `STAGING`, `PROD` — with its own configuration
attached. The same workflow deploys to all three; what differs is the values it reads.

**You create your own three in the next module.** They are not handed to you, and
that is deliberate: an environment inherited from the parent organization is shared,
so a value one person changed would land on everyone else mid-session.

### Feature flag

A switch you change in the Unify UI that changes what the running application does,
with no build and no deploy. Session 3 is entirely this. It is the part people
remember.

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
only from inside your namespace, which is why there is exactly one hostname to
remember rather than five.

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

`recall-db` deploys first and everything else waits for it. Deploy `recall-core-api`
into an empty database and the pod starts, passes its health check, and returns
errors on every request — a failure that looks like a bug in the API and is not.

Encoding that order is the Application's job. You will write it in Module 05.

### Why five and not one

This application could have been built as a single service, and the pipeline would
have been simpler. It would also have taught you nothing that matters.

Deployment ordering, coordinated releases, changing one service's behaviour without
touching its neighbours, working out which service actually broke — none of these
problems exist with one component. All of them are your Tuesday afternoon with
twenty.

---

## Configuration comes in two flavours

Worth knowing before the next module, because it is the single most common thing to
get wrong.

| Scope | Applies to | Examples |
|---|---|---|
| **Organization** | Everything in your sub-organization | Your DockerHub account, your cluster credential |
| **Environment** | One environment only | `namespace`, `hostname` — different for `DEV` and `PROD` |

Organization values reach your environments. Environment values do **not** fall back
to organization scope: if a workflow wants `namespace` and you set it at organization
scope, the workflow does not find it and fails before running a single step.

Module 02 is where you set all of this, and where a workflow called **Verify Setup**
checks it for you before anything is built.

---

## One thing that looks like a security hole

Your web UI serves its Feature Management SDK key from `/api/fm-config`, to anyone
who asks. You can open it in a browser and read the key.

That is correct, and deliberate. The browser-side SDK runs on the user's machine, so
it needs a key the user's machine can read — the same way a publishable Stripe key or
a Google Maps API key works. It permits reading flag configuration and nothing else.

Flag *changes* go through the Unify UI, authenticated as you.

We mention it now because somebody always finds it around Module 08 and reasonably
concludes the workshop app is broken.

---

## One thing to do: let Unify see your code

A Component is a repository with workflows in it — but Unify cannot read your
repositories until you say so. This is the step that makes the definition true, and it
is the only thing to click in this module.

In your sub-organization, add a **GitHub** integration. Unify sends you to GitHub to
install its app on your account, and GitHub asks which repositories the app may see:

- Choose **Only select repositories**, and select all six.
- Not **All repositories** — that grants access to everything in your account,
  including private repositories with nothing to do with this workshop. Selecting six
  is the same amount of clicking and models what you would actually do at work.

You do this **once**. Every later module picks from the repositories you granted here,
so nothing needs installing again in Modules 03 and 04.

> **Miss one and it goes missing later.** A repository you did not tick simply will not
> appear when you go to create its Component, with no explanation of why. The fix is on
> GitHub, in the app's configuration — not anywhere in Unify, which is where everyone
> looks first. Counting to six now is worth it.

---

## Before you move on

- [ ] All six repositories granted to the CloudBees GitHub app

You should also be able to answer these without looking. If one is fuzzy, ask now —
every later module assumes all four.

- [ ] What is the difference between a Component and an Application?
- [ ] Why does `recall-db` deploy before anything else?
- [ ] Which of your five components can be reached from the internet?
- [ ] Where do `namespace` and `hostname` need to be set, and what happens if you set
      them at organization scope instead?

---

**Next:** [Module 02 — Configuration](02-configuration.md)
