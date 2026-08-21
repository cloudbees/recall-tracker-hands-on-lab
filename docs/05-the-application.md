# Module 05 — The Application

**40 minutes** · Session 2: Release orchestration

Five components that each know how to build and deploy themselves, and nothing that
knows they belong together. This module closes that gap.

You already created the Application in Module 02, to have somewhere to run Verify Setup
from. Now it gets its actual job.

---

## 1. Attach the five components

Open your Application and add all five: `recall-db`, `recall-worker`,
`recall-ai-service`, `recall-core-api`, `recall-web-ui`.

> **This is where exact names start to matter.** The Application's workflow looks up
> each component by name. A component you called something else is not an error — it is
> simply absent from the release, and the first you will know about it is a service
> that never deployed. If you renamed one earlier, fix it now.

---

## 2. Make one edit

Open `.cloudbees/workflows/deployer.yaml` in your `app-recall-tracker` repository and
find `YOUR-GITHUB-ORG`. Replace every occurrence with your own GitHub account or
organization.

**Use find-and-replace rather than editing line by line.** There are eleven
occurrences: ten that matter — a `uses:` and a `component-repo:` for each of the five
components — plus the comment at the top of the file telling you to do this. Replacing
all eleven is harmless.

```yaml
uses: YOUR-GITHUB-ORG/recall-db/.cloudbees/workflows/deploy.yaml
...
  component-repo: https://github.com/YOUR-GITHUB-ORG/recall-db.git
```

Commit to `main`.

> **Match GitHub's capitalisation exactly.** If your account is `AcmeDev`, write
> `AcmeDev` — not `acmedev`. GitHub treats those as the same account, so nothing you do
> in a browser will complain, but the release will fail with an error saying your
> workflow is "outside the calling workflow's scm organization" while quoting two
> strings that look identical apart from a capital letter.
>
> The trap is that `DOCKERHUB_USER` is lowercase by DockerHub convention. Your GitHub
> account and your DockerHub account may be spelled differently, and only one of them
> belongs here.

### Why this file cannot use a variable

Everything else in this workshop is configured in the Unify UI. This one file needs
editing, and the reason is worth understanding rather than resenting.

`uses:` cannot be templated. Unify resolves the workflow graph — which workflows call
which — *before* it evaluates any expression. At the moment it needs to know where
`recall-db/deploy.yaml` lives, `${{ vars.DOCKERHUB_USER }}` has not been evaluated yet
and does not exist. So the path has to be literal text.

This is the first of several places in this workshop where **when** something is
resolved matters more than what it says. It is also a useful thing to recognise in your
own pipelines: an expression in the wrong position fails in a way that looks like a
typo.

---

## 3. Read the deployer

`deployer.yaml` is the whole point of an Application, and it is short. Six jobs.

```
manifest ──► DB ──┬──► Worker
                  ├──► AI-Service
                  └──► Core-API ──► Web-UI
```

### The dependencies are real, and only two of them

`DB` runs first because the other three query tables that do not exist until the
migrations have run. `Web-UI` waits for `Core-API` because `Web-UI` owns the Ingress
that routes `/api` to `Core-API` — deploy it first and the site is up while every API
call returns 503.

Everything else runs in parallel. `Worker`, `AI-Service` and `Core-API` have no
relationship to each other, so making them queue would add minutes per environment and
guarantee nothing.

> An earlier version of this file chained all five in series. That was conservative
> rather than correct: three extra sequential deploys per environment, times three
> environments, for an ordering nobody needed. Being able to say *why* a dependency
> exists is what lets you delete the ones that do not.

### The manifest job

The first job deploys nothing. It prints what the release asked for:

```
=== Deploying to: DEV ===

Expected component keys:
  present  recall-db
  present  recall-worker
  ABSENT   recall-ai-service  <- this component will NOT be deployed
  present  recall-core-api
  present  recall-web-ui
```

It exists because a component missing from a manifest deploys nothing and reports
success. That is correct behaviour — you often want to release one component — but it is
indistinguishable from a mistake unless something says so out loud.

Read this job's output first on every release. It is the cheapest place to catch a
naming mismatch.

### `vars: inherit` and `secrets: inherit`

Each job passes your configuration down to the component's own `deploy.yaml`. Without
those two lines the called workflow starts with nothing — no `namespace`, no
`kubeconfig` — and fails before its first step, because an undefined variable is fatal.

### And now `component-repo` makes sense

In Module 03 you left `component-repo` as `self` and I promised an explanation.

A called workflow runs in the **caller's** repository context. When the deployer calls
`recall-db/deploy.yaml`, that workflow's checkout step would clone
`app-recall-tracker` — the caller — rather than `recall-db`, and then fail to find the
Helm chart it needs. Passing an explicit clone URL is what corrects it.

This is the same lesson as `uses:` in a different costume: what looks like the obvious
default is wrong once workflows start calling each other.

---

## 4. What you have not done

Notice there is no run button on any of this yet. The deployer is only callable —
`workflow_call`, the third trigger from Module 04 — so nothing you have built in this
module can be started directly.

That is deliberate. What starts it is a **release**, which is Module 06.

---

## Before you move on

- [ ] Five components attached to the Application
- [ ] `YOUR-GITHUB-ORG` gone from `deployer.yaml`, committed to `main`
- [ ] You can say why `DB` runs before `Worker`, and why `Worker` does not run before
      `AI-Service`
- [ ] You can say why `uses:` cannot take a variable

The Application knows what the components are and what order they go in. It does not yet
know which *versions*. That is what a release decides.

---

**Next:** [Module 06 — Release orchestration](06-release-orchestration.md)
