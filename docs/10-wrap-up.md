# Module 10 — Wrap-up

**20 minutes** · Session 3: Feature management

---

## What you built

Not a tutorial project. A five-service application, built from source, released through
a gate, running at your own URL, with behaviour you can change while it runs.

| | |
|---|---|
| Components | 5, each a repository with its own build and deploy |
| Application | 1, orchestrating all five in dependency order |
| Environments | 3, with an approval gate before promotion |
| Container images | 5 or more, in your own registry, tagged by commit |
| Feature flags | 5, registered from code, controlled from the platform |
| Files you edited | 1 — and you know why that one had to be manual |

That last row is worth sitting with. Everything else was configuration in the UI or a
workflow already written. The single file you edited by hand was `deployer.yaml`, and it
had to be, because `uses:` is resolved before expressions are evaluated.

---

## The five things worth taking with you

**A Component is a repository with workflows. An Application is what releases several of
them together.** If you remember one sentence, this is the one.

**Artifacts, not source, move between environments.** What reached `PROD` was the exact
image tested in `DEV` — same digest. "Rebuild from the same commit" is not the same
promise, and the difference shows up on the day a dependency changes underneath you.

**Order is a property of the Application, not of the components.** Each component knows
how to deploy itself. Only the Application knows that the database goes first, and only
because someone wrote down why.

**_When_ something resolves matters as much as what it says.** `uses:` before expressions.
Configuration read at deploy time. A called workflow running in the caller's context. Most
of the confusing failures in this workshop were timing, wearing a costume.

**A flag is a decision moved out of the deploy and into the platform.** Existence and
default live in the code with the developer. Value lives in the platform with whoever
operates it. Neither can invent a flag the other has not agreed to.

---

## Where to go next

Things this application is already shaped for, that the workshop did not have time for:

| | |
|---|---|
| **Security scanning** | SAST, SCA, container and secret scanning on the components you just built, with findings attached to the same components |
| **DORA metrics** | Deployment frequency and lead time, from the releases you have been running |
| **Configuration as code** | Flag configuration in version control rather than clicked into a UI |
| **More environments** | Adding a fourth is a small edit to `release-wf.yaml`, because the deployer is called rather than copied |
| **Real approvers** | Name people in the `Approve` job, and set `disallowLaunchByUser` so the person who starts a release cannot approve it |

The parts of your own delivery worth comparing against what you just did: how long a
config change takes, whether you can name why each deployment dependency exists, and
whether turning a feature off requires a deploy.

---

## Teardown

Nothing for you to do. Your facilitator removes the sub-organization and the namespaces,
and the cluster goes with them.

Two things worth knowing:

- **The repositories are yours.** They are in your GitHub account, with their own
  history, and they do not disappear. So are the container images in your DockerHub
  account.
- **The URLs will stop working.** They point at namespaces in a cluster built for this
  workshop.

You will have about a week to continue experimenting in the workshop environment at which
point the URL's will be torn down.

---

## Questions
---

**Back to the start:** [Hands-on lab](../README.md)
